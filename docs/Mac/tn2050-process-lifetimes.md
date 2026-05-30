# TN2050: Observing Process Lifetimes Without Polling

> **Document:** Apple Technical Note TN2050  
> **Status:** Historical (archived, no longer updated)  
> **Last Updated:** 2008-09-10  
> **URL:** https://developer.apple.com/library/archive/technotes/tn2050/_index.html

---

## Overview

本技术说明描述了在 Mac OS X 上无需轮询即可跟踪进程生命周期的技术。所有讨论的方法都采用**通知机制**——进程状态变化时主动通知观察者，而非反复查询进程状态。

### 为什么避免轮询

- 消耗 CPU 时间
- 降低电池续航
- 增加工作集大小
- 增加响应延迟

---

## 核心原则

**使用 launchd 管理服务（推荐）**

在实现进程监控之前，首先考虑使用 `launchd` 来管理服务。将辅助进程重新设计为 launchd 托管的服务，由系统统一处理启动/终止细节。

参考: [launchd page](http://launchd.macosforge.org/)

---

## 观察你启动的进程

### 1. NSTask (Cocoa)

**同步方式:**

```objc
- (IBAction)testNSTaskSync:(id)sender {
    NSTask *syncTask;
    syncTask = [NSTask launchedTaskWithLaunchPath:@"/bin/sleep" 
                                        arguments:[NSArray arrayWithObject:@"1"]];
    [syncTask waitUntilExit];
}
```

**异步方式（推荐）:**

```objc
- (IBAction)testNSTaskAsync:(id)sender {
    task = [[NSTask alloc] init];
    [task setLaunchPath:@"/bin/sleep"];
    [task setArguments:[NSArray arrayWithObject:@"1"]];

    [[NSNotificationCenter defaultCenter] 
        addObserver:self 
        selector:@selector(taskExited:) 
        name:NSTaskDidTerminateNotification 
        object:task];
    [task launch];
}

- (void)taskExited:(NSNotification *)note {
    [[NSNotificationCenter defaultCenter] 
        removeObserver:self 
        name:NSTaskDidTerminateNotification 
        object:task];
    [task release];
    task = nil;
}
```

---

### 2. Application Died Events (Apple Events)

通过进程序列号 (PSN) API 启动时注册 `kAEApplicationDied` Apple Event:

```objc
- (IBAction)testApplicationDied:(id)sender {
    NSURL *url;
    static BOOL sHaveInstalledAppDiedHandler;

    if (!sHaveInstalledAppDiedHandler) {
        (void) AEInstallEventHandler(
            kCoreEventClass, 
            kAEApplicationDied, 
            (AEEventHandlerUPP) AppDiedHandler, 
            (SRefCon) self, 
            false);
        sHaveInstalledAppDiedHandler = YES;
    }

    url = [NSURL fileURLWithPath:@"/Applications/TextEdit.app"];
    (void) LSOpenCFURLRef((CFURLRef) url, NULL);
}

static OSErr AppDiedHandler(
    const AppleEvent *theAppleEvent, 
    AppleEvent *reply, 
    SRefCon handlerRefcon) {
    SInt32 errFromEvent;
    ProcessSerialNumber psn;
    DescType junkType;
    Size junkSize;

    (void) AEGetParamPtr(theAppleEvent, keyErrorNumber, typeSInt32, 
                         &junkType, &errFromEvent, sizeof(errFromEvent), &junkSize);
    (void) AEGetParamPtr(theAppleEvent, keyProcessSerialNumber, 
                         typeProcessSerialNumber, &junkType, &psn, sizeof(psn), &junkSize);

    NSLog(@"died %lu.%lu %d", 
          (unsigned long) psn.highLongOfPSN, 
          (unsigned long) psn.lowLongOfPSN, 
          (int) errFromEvent);
    return noErr;
}
```

**注意:** 仅对你启动的应用程序发送通知，基于进程序列号。

---

### 3. UNIX 方式 (fork/exec + wait)

**同步 waitpid:**

```objc
extern char **environ;

- (IBAction)testWaitPID:(id)sender {
    pid_t pid;
    char *args[3] = { "/bin/sleep", "1", NULL };
    pid_t waitResult;
    int status;

    pid = fork();
    switch (pid) {
        case 0: // child
            (void) execve(args[0], args, environ);
            _exit(EXIT_FAILURE);
            break;
        case -1: // error
            break;
        default: // parent
            break;
    }
    if (pid >= 0) {
        do {
            waitResult = waitpid(pid, &status, 0);
        } while ((waitResult == -1) && (errno == EINTR));
    }
}
```

**异步 via SIGCHLD:**

- 使用 `signal` 或 `sigaction` 安装信号处理器
- **仅使用异步信号安全函数**
- 两种重定向技术:
  1. **Sockets** — 创建 UNIX 域 socket 对，使用 CFSocket 添加到 runloop
  2. **Kqueues** — 使用 kqueue 监听信号，无需安装处理器 (10.5+)

---

### 4. Socket Pair (UNIX 替代方案)

创建 socket 对，子进程持有一端；子进程退出时，父进程检测到 EOF:

```objc
- (IBAction)testSocketPair:(id)sender {
    int fds[2], remoteSocket, localSocket;
    CFSocketContext context = { 0, self, NULL, NULL, NULL };
    CFRunLoopSourceRef rls;
    char *args[3] = { "/bin/sleep", "1", NULL };

    (void) socketpair(AF_UNIX, SOCK_STREAM, 0, fds);
    remoteSocket = fds[0];
    localSocket = fds[1];
    
    socket = CFSocketCreateWithNative(NULL, localSocket, kCFSocketDataCallBack,
                                      SocketClosedSocketCallBack, &context);
    CFSocketSetSocketFlags(socket, kCFSocketAutomaticallyReenableReadCallBack 
                           | kCFSocketCloseOnInvalidate);

    rls = CFSocketCreateRunLoopSource(NULL, socket, 0);
    CFRunLoopAddSource(CFRunLoopGetCurrent(), rls, kCFRunLoopDefaultMode);
    CFRelease(rls);

    childPID = fork();
    switch (childPID) {
        case 0:
            (void) execve(args[0], args, environ);
            _exit(EXIT_FAILURE);
            break;
        case -1:
            break;
        default:
            break;
    }
    (void) close(remoteSocket);
}

static void SocketClosedSocketCallBack(CFSocketRef s, CFSocketCallBackType type,
                                       CFDataRef address, const void *data, void *info) {
    int waitResult, status;
    do {
        waitResult = waitpid(((AppDelegate *) info)->childPID, &status, 0);
    } while ((waitResult == -1) && (errno == EINTR));
}
```

---

## 观察任意进程

### 1. NSWorkspace (Cocoa)

注册 NSWorkspace 通知中心的通知:

```objc
- (IBAction)testNSWorkspace:(id)sender {
    NSNotificationCenter *center;
    center = [[NSWorkspace sharedWorkspace] notificationCenter];

    [center addObserver:self 
                selector:@selector(appLaunched:) 
                    name:NSWorkspaceDidLaunchApplicationNotification 
                  object:nil];
    [center addObserver:self 
                selector:@selector(appTerminated:) 
                    name:NSWorkspaceDidTerminateApplicationNotification 
                  object:nil];
}

- (void)appLaunched:(NSNotification *)note {
    NSLog(@"launched %@\n", [[note userInfo] objectForKey:@"NSApplicationName"]);
}

- (void)appTerminated:(NSNotification *)note {
    NSLog(@"terminated %@\n", [[note userInfo] objectForKey:@"NSApplicationName"]);
}
```

---

### 2. Carbon Event Manager

注册 `kEventAppLaunched` 和 `kEventAppTerminated` 事件:

```objc
static OSStatus CarbonEventHandler(EventHandlerCallRef inHandlerCallRef, 
                                   EventRef inEvent, void *inUserData) {
    ProcessSerialNumber psn;
    (void) GetEventParameter(inEvent, kEventParamProcessID, 
                             typeProcessSerialNumber, NULL, sizeof(psn), NULL, &psn);
    switch (GetEventKind(inEvent)) {
        case kEventAppLaunched:
            NSLog(@"launched %u.%u", (unsigned int) psn.highLongOfPSN, 
                  (unsigned int) psn.lowLongOfPSN);
            break;
        case kEventAppTerminated:
            NSLog(@"terminated %u.%u", (unsigned int) psn.highLongOfPSN, 
                  (unsigned int) psn.lowLongOfPSN);
            break;
    }
    return noErr;
}
```

---

### 3. Kqueues (跨上下文)

适用于在 GUI 登录上下文之外监控，或在不同上下文中:

```objc
static pid_t gTargetPID = -1;

- (IBAction)testNoteExit:(id)sender {
    FILE *f;
    int kq;
    struct kevent changes;
    CFFileDescriptorContext context = { 0, self, NULL, NULL, NULL };
    CFRunLoopSourceRef rls;

    kq = kqueue();
    EV_SET(&changes, gTargetPID, EVFILT_PROC, EV_ADD | EV_RECEIPT, NOTE_EXIT, 0, NULL);
    (void) kevent(kq, &changes, 1, &changes, 1, NULL);

    noteExitKQueueRef = CFFileDescriptorCreate(NULL, kq, true, 
                                                NoteExitKQueueCallback, &context);
    rls = CFFileDescriptorCreateRunLoopSource(NULL, noteExitKQueueRef, 0);
    CFRunLoopAddSource(CFRunLoopGetCurrent(), rls, kCFRunLoopDefaultMode);
    CFRelease(rls);
    CFFileDescriptorEnableCallBacks(noteExitKQueueRef, kCFFileDescriptorReadCallBack);
}

static void NoteExitKQueueCallback(CFFileDescriptorRef f, CFOptionFlags callBackTypes, 
                                   void *info) {
    struct kevent event;
    (void) kevent(CFFileDescriptorGetNativeDescriptor(f), NULL, 0, &event, 1, NULL);
    NSLog(@"terminated %d", (int)(pid_t) event.ident);
}
```

**要求:** Mac OS X 10.5+

---

## 进程序列号 (PSN) API 的局限性

高级 API（Launch Services、Process Manager、NSWorkspace）有三个共同限制:

| 限制 | 说明 |
|------|------|
| **单一 GUI 登录会话** | 只能看到同一 GUI 会话中的进程 |
| **需要连接 Window Server** | 命令行工具不会触发通知 |
| **守护进程不可用** | 无法在 GUI 登录上下文之外使用 |

---

## 方案对比

### 维度对比表

| 方案 | 监控范围 | 平台/环境 | 复杂度 | 现代可用性 | 核心优势 | 核心局限 |
|------|---------|----------|--------|-----------|---------|---------|
| **launchd** | 任意进程 | macOS 全局 | 低 | ✅ 推荐 | 系统级管理、自动重启、开机启动 | 需重新设计为服务 |
| **NSTask** | 仅自己启动 | Cocoa/ObjC | 中 | ⚠️ 已废弃<br>(用 Process) | 简单易用、Cocoa 集成 | 只能监控 NSTask 启动的进程 |
| **Apple Events<br>(kAEApplicationDied)** | 仅自己启动 | Carbon/Cocoa | 高 | ❌ 历史方案 | 可获取退出码、PSN | 仅对启动的 GUI 应用有效 |
| **waitpid** | 仅自己启动 | POSIX/跨平台 | 中 | ✅ 推荐 | 跨平台、精确控制 | 同步/信号处理复杂 |
| **Socket Pair** | 仅自己启动 | POSIX | 高 | ✅ 可用 | 可集成 CFRunLoop、跨进程通信 | 实现较复杂 |
| **NSWorkspace** | 当前会话<br>所有应用 | Cocoa | 低 | ⚠️ 部分可用 | 简单通知、应用启动/终止 | 仅 GUI 会话、命令行工具不触发 |
| **Carbon Events** | 当前会话<br>所有应用 | Carbon | 高 | ❌ 已废弃 | 事件驱动 | Carbon 已废弃 |
| **kqueue<br>(EVFILT_PROC)** | 任意进程 | macOS 10.5+ | 中 | ✅ 推荐 | 跨上下文、精确过滤、无需信号处理 | 仅 macOS |

---

### 按使用场景推荐

#### 1. 守护进程/后台服务

| 推荐度 | 方案 | 理由 |
|-------|------|------|
| ⭐⭐⭐ | **launchd** | 系统级管理、自动重启、标准化 |
| ⭐⭐ | **kqueue + EVFILT_PROC** | 精确控制、支持 NOTE_EXIT 等事件 |

#### 2. 辅助进程 (你启动的子进程)

| 推荐度 | 方案 | 理由 |
|-------|------|------|
| ⭐⭐⭐ | **NSTask / Process** | 原生 API、简单易用 |
| ⭐⭐ | **waitpid** | 跨平台、精确控制 |
| ⭐ | **Socket Pair** | 可复用 socket、适合 IPC 场景 |

#### 3. 监控当前 GUI 会话中的任意应用

| 推荐度 | 方案 | 理由 |
|-------|------|------|
| ⭐⭐⭐ | **NSWorkspace** | 最简单、Cocoa 原生 |
| ⭐⭐ | **Carbon Events** | 底层控制 |

#### 4. 跨平台需求

| 推荐度 | 方案 | 理由 |
|-------|------|------|
| ⭐⭐⭐ | **waitpid** | POSIX 标准、macOS/Linux 通用 |
| ⭐ | **kqueue** | 限 BSD/macOS |

---

### 复杂度 vs 功能权衡

```
低复杂度 ────────────────────────────────────── 高复杂度
    │                                              │
    ├── NSWorkspace ◄── 简单通知、无需控制          │
    │                                              │
    ├── NSTask ─────── 简单子进程管理              │
    │                                              │
    ├── waitpid ────── 同步/异步皆可、跨平台       │
    │                                              │
    ├── Socket Pair ── 可复用连接、集成 RunLoop   │
    │                                              │
    └── kqueue ──────── 精确控制、跨上下文         │
```

---

### 关键决策树

```
你需要监控什么？
│
├─ 服务/守护进程
│  └─ 使用 launchd
│
├─ 自己启动的子进程
│  ├─ 需要跨平台 ──── waitpid
│  ├─ macOS/Cocoa ─── NSTask / Process
│  └─ 需要 IPC ────── Socket Pair
│
└─ 任意进程（任意会话）
   └─ 使用 kqueue + EVFILT_PROC
```

---

## 方法选择快速参考

| 场景 | 推荐方法 |
|------|---------|
| 你启动的简单辅助进程 | NSTask (异步通知) |
| 你启动的 macOS 应用 | Application Died Events |
| 守护进程/后台进程 | kqueue + EVFILT_PROC |
| 当前 GUI 会话中的任意应用 | NSWorkspace 或 Carbon Events |
| 服务化设计 | launchd (推荐) |

---

## 历史背景

本文档是 Apple Technical Note TN2050，于 2008 年 9 月 10 日最后更新。虽然现在已是归档状态，但其中描述的核心概念（通知优于轮询、服务化架构）至今仍具参考价值。

现代 macOS 开发中，进程监控方法可能有更新，但 `launchd` 和 `kqueue` 仍然是系统级进程管理的基石。