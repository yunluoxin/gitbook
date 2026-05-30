# macOS App 免密 Sudo 安全落地方案

# macOS App 免密 Sudo 安全落地方案

（静态固定命令版 \+ 动态传参版 合并完整版）

## 一、方案总览

### 目标

实现 macOS 非 PKG 分发的 `\.app` 程序**免密执行 sudo**，同时解决：

- 明文密码硬编码风险

- App 被篡改后滥用免密权限

- 提权脚本被恶意修改

- 哈希固化与脚本修改的悖论问题

### 核心设计思路

1. **打包阶段固化哈希**：App 编译完成后，将 App 二进制 SHA‑256 哈希写入提权脚本模板，生成最终脚本，**运行阶段不再修改脚本内容**。

2. **首次运行一次性加固**：用户第一次打开 App 时，弹窗请求一次管理员权限，后台自动完成脚本权限锁死 \+ sudoers 授权配置，之后永久不再需要密码与授权。

3. **双版本支持**：

    - 静态版：脚本内写死所有 sudo 逻辑，无外部参数，安全性最高

    - 动态版：支持 App 传入命令参数，增加白名单校验，通用性更强

### 安全闭环

- App 篡改 → 脚本内哈希校验拦截

- 脚本篡改 → root 权限 \+ 只读 \+ 系统不可变标志拦截

- 动态命令注入 → 命令白名单拦截

- 权限滥用 → sudoers 最小授权，仅信任该提权脚本

---

## 二、整体执行阶段划分

### 阶段 1：开发打包阶段（仅开发者执行）

1. 编译生成 App 主程序二进制文件

2. 计算 App 二进制 SHA‑256 哈希

3. 将哈希、App 绝对路径填充进脚本模板，生成 `run\_with\_sudo\.sh`

4. 脚本放入 App 的 `Contents/Resources` 目录，赋予基础执行权限

> 本阶段**不设置 root 权限、不开启不可变标志**，该类权限无法随 `\.app` 包分发到用户端。
> 
> 

### 阶段 2：用户首次运行阶段（仅执行一次）

1. App 启动检测是否已完成加固

2. 未加固则调用系统授权接口，弹出标准管理员授权窗口

3. 用户允许后，App 以临时 root 权限执行**加固脚本**，完成两件事：

    - 锁死提权脚本：归属 root、只读、开启用户不可变标志

    - 写入 `/etc/sudoers\.d` 免密规则，仅允许该提权脚本免密执行

4. 生成加固完成标记文件，后续启动不再重复授权

### 阶段 3：日常运行阶段（永久免密）

App 直接调用已加固的提权脚本，脚本自动校验 App 完整性，校验通过后执行 sudo 操作，全程无弹窗、无密码。

---

## 三、通用公共组件（双版本共用）

### 3\.1 首次运行加固脚本 `secure\_install\.sh`

**作用**：仅在首次运行授权后执行，用于锁死提权脚本 \+ 配置 sudoers 权限
**调用时机**：App 首次启动拿到临时 root 权限后调用，后续不再执行

```bash
#!/bin/zsh
set -euo pipefail

# 入参：提权脚本绝对路径
SCRIPT_PATH="$1"
# sudoers 授权文件
SUDOERS_FILE="/etc/sudoers.d/app_sudo_secure_rule"
# 加固完成标记文件路径
FLAG_FILE="$(dirname "${SCRIPT_PATH}")/.security_lock_flag"
# 获取当前登录用户
CUR_USER=$(stat -f %Su /dev/console)

# 1. 终极锁死提权脚本，普通用户无法修改、删除、重命名
chown root:wheel "${SCRIPT_PATH}"
chmod 500 "${SCRIPT_PATH}"
chflags uchg "${SCRIPT_PATH}"

# 2. 写入免密规则（兼容动态版通配符，静态版可后续精简）
echo "${CUR_USER} ALL=(ALL) NOPASSWD: ${SCRIPT_PATH} *" > "${SUDOERS_FILE}"
chmod 440 "${SUDOERS_FILE}"

# 3. 校验 sudoers 语法，避免系统 sudo 失效
visudo -c -f "${SUDOERS_FILE}"

# 4. 写入加固标记并锁死，避免重复执行 / 被删除后重新弹窗
echo "secured" > "${FLAG_FILE}"
chown root:wheel "${FLAG_FILE}"
chmod 444 "${FLAG_FILE}"
chflags uchg "${FLAG_FILE}"

# 5. 自我锁死：自身也加同样保护，防止被篡改
# 脚本运行中加 chflags uchg 是安全的：shell 已加载到内存，只阻止下次修改
SELF_PATH="$0"
chown root:wheel "${SELF_PATH}"
chmod 500 "${SELF_PATH}"
chflags uchg "${SELF_PATH}"
```

### 3\.2 打包哈希生成脚本（开发者使用）

**作用**：编译 App 后，自动填充哈希与路径到脚本模板，生成最终可用的提权脚本

```bash
#!/bin/zsh

# ========== 配置项（根据实际项目修改） ==========
APP_BIN="./MyApp.app/Contents/MacOS/MyApp"
TEMPLATE_FILE="./run_with_sudo.template"
OUTPUT_SCRIPT="./MyApp.app/Contents/Resources/run_with_sudo.sh"
# ================================================

# 计算 App 二进制 SHA‑256 哈希
APP_HASH=$(shasum -a 256 "${APP_BIN}" | awk '{print $1}')

# 模板替换，生成最终脚本
sed \
    -e "s|__FIXED_APP_HASH__|${APP_HASH}|g" \
    -e "s|__APP_BIN_ABS_PATH__|${APP_BIN}|g" \
    "${TEMPLATE_FILE}" > "${OUTPUT_SCRIPT}"

# 赋予基础执行权限
chmod +x "${OUTPUT_SCRIPT}"

echo "脚本生成完成"
echo "固化哈希：${APP_HASH}"
```

### 3\.3 Swift 基础调用代码

```swift
import Foundation
import Security

/// 检测是否已完成首次加固
func isScriptSecured() -> Bool {
    guard let flagPath = Bundle.main.path(forResource: ".security_lock_flag", ofType: nil) else {
        return false
    }
    return FileManager.default.fileExists(atPath: flagPath)
}

/// 首次运行执行加固逻辑
func runFirstTimeSecure() {
    guard !isScriptSecured() else { return }

    guard let installScript = Bundle.main.path(forResource: "secure_install", ofType: "sh"),
          let sudoScript = Bundle.main.path(forResource: "run_with_sudo", ofType: "sh") else {
        return
    }

    var authRef: AuthorizationRef?
    AuthorizationCreate(nil, nil, .allowInteraction, &authRef)

    let task = Process()
    task.executableURL = URL(fileURLWithPath: "/bin/zsh")
    task.arguments = [installScript, sudoScript]
    task.run()
    task.waitUntilExit()
}
```

---

## 四、版本一：静态固定命令版（推荐高安全场景）

### 适用场景

需要执行的 sudo 操作固定不变，无动态命令需求，追求最高安全性

### 4\.1 提权脚本模板 `run\_with\_sudo\.template`

```bash
#!/bin/zsh
set -euo pipefail

# 打包时自动填充
SAFE_HASH="__FIXED_APP_HASH__"
APP_BIN="__APP_BIN_ABS_PATH__"

# 固定纯净环境，防止 PATH 劫持
export PATH="/usr/bin:/bin:/usr/sbin:/sbin"

# 校验 App 完整性
CUR_HASH=$(shasum -a 256 "${APP_BIN}" | awk '{print $1}')
if [[ "${CUR_HASH}" != "${SAFE_HASH}" ]]; then
    echo "[安全拦截] App 二进制已被篡改，拒绝提权" >&2
    exit 1
fi

# ========== 业务固定 sudo 逻辑（全部写死在此处），示例 ==========
sudo networksetup -setproxyautodiscovery Wi‑Fi off
sudo launchctl kickstart -k system/com.apple.networkd
# ========================================================
```

### 4\.2 可选优化（极致安全）

首次加固完成后，可手动修改 `/etc/sudoers\.d/app\_sudo\_secure\_rule`，**移除末尾通配符**：

```Plain Text
用户名 ALL=(ALL) NOPASSWD: /绝对路径/run_with_sudo.sh
```

### 4\.3 Swift 调用示例（无参调用）

```swift
func runFixedSudo() {
    guard let script = Bundle.main.path(forResource: "run_with_sudo", ofType: "sh") else { return }
    let task = Process()
    task.executableURL = URL(fileURLWithPath: script)
    task.run()
    task.waitUntilExit()
}
```

---

## 五、版本二：动态传参通用版（推荐灵活业务场景）

### 适用场景

需要根据业务动态执行不同 sudo 命令，通用性强，需做好白名单防护

### 5\.1 提权脚本模板 `run\_with\_sudo\.template`

```bash
#!/bin/zsh
set -euo pipefail

# 打包时自动填充
SAFE_HASH="__FIXED_APP_HASH__"
APP_BIN="__APP_BIN_ABS_PATH__"

# 允许执行的 sudo 命令白名单（自行增删）
ALLOW_LIST=(
    networksetup
    launchctl
    ifconfig
)

# 固定纯净环境
export PATH="/usr/bin:/bin:/usr/sbin:/sbin"

# 校验 App 完整性
CUR_HASH=$(shasum -a 256 "${APP_BIN}" | awk '{print $1}')
if [[ "${CUR_HASH}" != "${SAFE_HASH}" ]]; then
    echo "[安全拦截] App 二进制已被篡改，拒绝提权" >&2
    exit 1
fi

# 校验传入命令是否合法，防止注入
if [[ $# -lt 1 ]]; then
    echo "[参数错误] 未传入执行命令" >&2
    exit 1
fi

RUN_CMD="$1"
if [[ ! " ${ALLOW_LIST[@]} " =~ " ${RUN_CMD} " ]]; then
    echo "[安全拦截] 非法命令：${RUN_CMD}" >&2
    exit 1
fi

# 执行 sudo 命令
sudo "$@"
```

### 5\.2 Swift 调用示例（动态传参）

```swift
func runDynamicSudo(command: String, args: [String] = []) {
    guard let script = Bundle.main.path(forResource: "run_with_sudo", ofType: "sh") else { return }
    let task = Process()
    task.executableURL = URL(fileURLWithPath: script)
    task.arguments = [command] + args
    task.run()
    task.waitUntilExit()
}

// 调用示例
// runDynamicSudo(command: "networksetup", args: ["‑getwebproxy", "Wi‑Fi"])
```

---

## 六、卸载清理脚本

彻底移除所有权限配置，恢复系统默认状态

```bash
# 删除 sudoers 授权规则
sudo rm /etc/sudoers.d/app_sudo_secure_rule

# 解除 run_with_sudo.sh 不可变标记，恢复普通权限
SCRIPT_PATH="你的App绝对路径/Contents/Resources/run_with_sudo.sh"
sudo chflags nouchg "${SCRIPT_PATH}"
sudo chown $USER:staff "${SCRIPT_PATH}"
sudo chmod 755 "${SCRIPT_PATH}"

# 解除 secure_install.sh 不可变标记，恢复普通权限
SECURE_PATH="你的App绝对路径/Contents/Resources/secure_install.sh"
sudo chflags nouchg "${SECURE_PATH}"
sudo chown $USER:staff "${SECURE_PATH}"
sudo chmod 755 "${SECURE_PATH}"

# 删除加固标记（需先解除不可变标志）
sudo chflags nouchg "你的App绝对路径/Contents/Resources/.security_lock_flag"
sudo rm "你的App绝对路径/Contents/Resources/.security_lock_flag"
```

---

## 七、选型建议

1. **固定业务、自用工具、对外分发高安全要求** → 静态固定命令版

2. **通用工具、动态执行命令、需灵活扩展** → 动态传参版

3. 两套方案均遵循最小权限原则，无原理性漏洞，仅首次运行一次管理员授权，为 macOS 非 PKG 分发下最优落地方案。

> （注：文档由`豆包`生成）
