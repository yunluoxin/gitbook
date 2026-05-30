# macOS CLI 进程实时监控架构（高性能 / 低 CPU / 接近实时）

## 1. 目标与约束

本方案用于在 macOS 上实现对 CLI 进程（如 Homebrew 安装工具、node/python/自定义命令等）的监控，要求：

- 高性能（低 CPU 占用）
- 低延迟（接近实时）
- 无需系统级权限（优先）
- 可扩展到任意 CLI 工具识别

---

## 2. macOS 进程监控的现实限制

macOS 并不存在统一的“进程启动监听 API”（在用户态）。因此必须组合多种机制：

| 能力 | 是否存在 | 说明 |
|------|----------|------|
| 全局进程事件监听 | ❌ | 用户态不可用 |
| GUI App 生命周期监听 | ✔ | NSWorkspace |
| 任意 PID 生命周期监听 | ✔（需已知 PID） | DispatchSourceProcess |
| 系统级实时 exec/exit | ✔ | Endpoint Security（需权限） |
| 全局进程枚举 | ✔ | libproc |

---

## 3. 总体架构设计

推荐采用“三层混合模型”：

```
        ┌──────────────────────────┐
        │   Layer 1: PID 探测层     │
        │   （libproc 轮询）        │
        └──────────┬───────────────┘
                   │ 新 PID
                   ▼
        ┌──────────────────────────┐
        │ Layer 2: 进程确认层       │
        │（路径 / argv / 规则匹配） │
        └──────────┬───────────────┘
                   │ 命中目标 CLI
                   ▼
        ┌──────────────────────────┐
        │ Layer 3: 生命周期监听层   │
        │ DispatchSourceProcess     │
        └──────────────────────────┘
```

---

## 4. Layer 1：低成本 PID 变化检测（核心）

### 作用

检测“系统中是否出现新进程”。

### 原理

使用 libproc 获取当前所有 PID 列表：

- proc_listallpids()

然后进行 diff：

```
previous_set
   ↓ diff ↓
current_set
   ↓
new_pids
```

### 特点

- O(n) 遍历 PID
- 不访问进程详细信息
- CPU 开销极低
- 是整个系统的“触发器”

---

## 5. Layer 2：进程识别层（过滤关键 CLI）

### 作用

判断新 PID 是否属于目标 CLI 工具。

### 可用信息来源

#### 1. 可执行路径（最可靠）

- proc_pidpath(pid)

示例特征：

- /opt/homebrew/bin/xxx
- /usr/local/bin/xxx

优点：稳定、不可伪造性较高

---

#### 2. argv[0]

- PROC_PIDLISTARGV

特点：

- 更接近用户执行命令
- 但可能被 shell 包裹

---

#### 3. 规则匹配策略

推荐组合：

- lastPathComponent == "claude"
- path contains "/homebrew/bin/"
- argv[0] matches CLI name

---

### 输出结果

命中目标后生成：

```
TrackedProcess {
    pid
    name
    path
    timestamp
}
```

---

## 6. Layer 3：生命周期监听（低延迟退出检测）

### 作用

在已确认 PID 后，监听其退出事件。

### 技术

- DispatchSourceProcess
- 基于 kqueue / EVFILT_PROC

### 监听事件

- exit（最重要）
- exec（可选）
- fork（可选）

---

### 特点

- 内核驱动
- 无轮询
- O(1) 通知
- 极低 CPU

---

## 7. 三层协同机制（关键设计）

### Step 1：轮询发现

每 0.5~1 秒：

- 获取 PID 列表
- diff 得到新 PID

---

### Step 2：筛选目标进程

对新 PID：

- 获取 path / argv
- 判断是否 CLI 目标

---

### Step 3：注册监听

对命中的 PID：

- 创建 DispatchSourceProcess
- 监听 exit

---

### Step 4：生命周期管理

当 exit 触发：

- 清理监听器
- 更新状态

---

## 8. 性能分析

### CPU 开销

| 操作 | 成本 |
|------|------|
| PID list | 极低 |
| diff set | 极低 |
| path lookup（新增 PID） | 中等（稀疏触发） |
| DispatchSourceProcess | 接近 0 |

---

### 为什么整体是低 CPU

因为：

- 大部分 PID 不重复查询 path
- 只有“新增进程”才访问详细信息
- 退出检测完全事件驱动

---

## 9. 可扩展增强方案

### 9.1 黑白名单机制

- CLI 白名单：claude, node, python
- 路径黑名单：/System/Library

---

### 9.2 缓存系统

- pid → path cache
- 避免重复 syscall

---

### 9.3 批量优化

- PID set diff 使用 HashSet
- 降低 O(n²)

---

## 10. 可选高级方案（系统级）

### Endpoint Security Framework

适用于：

- 安全软件
- 企业监控
- 实时 exec/fork/exit

特点：

- 真·实时
- 覆盖所有进程
- 需要系统扩展权限

---

## 11. 适用场景总结

| 场景 | 推荐方案 |
|------|----------|
| CLI 工具监控 | 本方案（三层模型） |
| GUI App 监控 | NSWorkspace |
| 子进程监控 | DispatchSourceProcess |
| 安全级监控 | Endpoint Security |

---

## 12. 核心结论

> macOS 上没有“万能进程监听 API”，高性能方案一定是“轮询 + 事件驱动混合架构”。

其中关键思想：

- libproc 负责“发现”
- 规则系统负责“识别”
- DispatchSourceProcess 负责“监听”

---

## 13. 架构优势总结

- CPU 极低
- 无系统权限需求
- 可扩展任意 CLI
- 接近实时响应
- 工业级稳定性

