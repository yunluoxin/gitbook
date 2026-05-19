# 探讨如何让 AI 长时间自动跑代码

## 本地运行

> 正常模式运行

### Claude Code

- 权限申请弹窗，跳的非常频繁！！！连网络搜索也一直跳。简直没法用。不敢用那个 dangerous 权限 怕删除磁盘文件。
- 中间设置过 permissions 的 allow 列表，但是没用。

  ```json
    "permissions": {
      "allow": [
        "Read(*)",
        "Write(*)",
        "Edit(*)",
        "Bash(*)",
        "Git(*)",
        "Npm(*)",
        "Pip(*)",
        "Network(*)"
      ]
    },
  ```

- 后面受不了了，直接开了 `bypass permissions on`，清净多了。（最后才知道，这个就是 dangerous 权限!!! 慎用）

  方法：直接修改 `~/.claude/settings.json` 文件，如下内容(和 env 同级)：

  ```json
  {
    //   "env": {},
    "permissions": {
      "defaultMode": "bypassPermissions" // 1. 先开启危险权限模式
    },
    "skipDangerousModePermissionPrompt": true // 2. 再关掉烦人的警告
  }
  ```

  | 设置项                                  | 控制层面 | 实际效果                             | 风险影响           | 启动提示      |
  | --------------------------------------- | -------- | ------------------------------------ | ------------------ | ------------- |
  | defaultMode: "bypassPermissions"        | 权限行为 | ✅ Claude 自动执行所有操作（无确认） | 高（授予全部权限） | 显示 红色警告 |
  | skipDangerousModePermissionPrompt: true | UI 提示  | ✅ 仅隐藏警告文字                    | 无（权限不变）     | 隐藏 红色警告 |

  > 结果：Claude 以完全无限制、静默无提示的模式运行。

  > 安全提醒：bypassPermissions 模式仅限在隔离环境（Docker / 虚拟机）中使用，严禁在本地开发机或生产服务器长期开启。

### OpenCode

OpenCode 本身提示就比较少，大多数时候都是自动运行。

直接修改 `~/.config/opencode/config.json` 文件，添加如下内容：

```json
{
  // ✅ 全局自动允许所有操作（无任何确认弹窗）
  "permission": "allow",
  // ✅ 隐藏「危险模式」红色顶部警告
  "skipDangerousModePermissionPrompt": true
}
```

或者直接设置 `yolo` 模式：

```json
{
  "$schema": "https://opencode.ai/config.json",
  // ✅ 等价 permission: "allow" + 隐藏警告
  "yolo": true
}
```

CLI 一键无提示（临时模式）：

```shell
opencode --yolo
# 或完整写法
opencode --dangerously-skip-permissions
```

另外，`opencode` 底部的 tips 条也可以关闭：

```json
{
  // 3. 隐藏底部输入提示（可选）
  "tui": {
    "tips": false
  }
}
```

---

## VS Code：扩展、版本与长时间运行（设置向）

下面把文档里涉及的 **Claude Code**、**OpenCode** 两套 CLI 在 VS Code 里的装法、**如何看/锁扩展版本**，以及**偏配置、少交互**地跑长任务时建议勾哪些项写在一起，方便对照。

### 通用前提

- **VS Code 版本**：Claude Code 官方扩展要求 **VS Code ≥ 1.98**（`帮助 → 关于` 或终端执行 `code --version`）。版本过低会出现扩展装不上、面板异常等问题。
- **`code` 在 PATH**：若从外部终端启动的 VS Code 读不到你在 shell 里配的 `ANTHROPIC_API_KEY` 等变量，可用 **`code .` 从已登录/已 export 的终端启动**，或在 VS Code 里用 **Remote / 集成终端** 保证环境一致。
- **长时间跑时**：系统睡眠、合盖、锁屏会挂起或打断子进程；除下面编辑器设置外，还需在 **系统电源策略** 里避免睡眠，或用 **Docker/远程机** 跑（见本文「容器化」）。

### Claude Code：Marketplace 扩展与 CLI 的关系

| 项目 | 说明 |
| --- | --- |
| **扩展 ID** | `anthropic.claude-code`（发布者 Anthropic） |
| **安装** | 扩展视图（`Cmd+Shift+X` / `Ctrl+Shift+X`）搜索「Claude Code」；或 Marketplace：[Claude Code for VS Code](https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code)；Cursor 用户可用 `cursor:extension/anthropic.claude-code` 链接触发安装。 |
| **与 CLI** | 扩展自带/联动 CLI；**图形面板与终端里的 `claude` 共用会话与 `~/.claude/settings.json`**。仅 CLI 才有的能力（文档写明的那部分）在集成终端里执行 `claude` 即可。 |
| **版本怎么写进文档** | Marketplace 上版本号**随发布变化**，不建议在笔记里写死某一串数字。需要「当前版本」时在扩展详情页看 **Version**；需要锁版本见下一小节。 |

#### 如何查看 / 固定扩展版本（避免自动升级打乱行为）

1. **图形界面**：扩展 → 已安装的 Claude Code → 齿轮 → **Install Another Version…**（安装其他版本），选某一历史版本。
2. **命令行**（示例格式，版本号请换成你在 Marketplace 上看到的）：

   ```bash
   code --install-extension anthropic.claude-code@x.y.z
   ```

固定版本后，可把 `x.y.z` 记在个人 runbook 里，升级前再评估 Release Notes。

#### VS Code `settings.json` 里与「少打断、长会话」相关的项

这些在 **设置 → 搜索 `Claude Code`** 或 **用户 `settings.json`** 里配置，与上文 `~/.claude/settings.json` 的 **权限模式** 互补：前者管 **扩展在 IDE 内的默认行为**，后者管 **工具/Bash 等底层权限**。

| 配置键 | 作用（与长跑相关） |
| --- | --- |
| `claudeCode.initialPermissionMode` | 新会话默认模式：`default` / `plan` / `acceptEdits` / `bypassPermissions`。希望**少弹确认**可设 `acceptEdits`（自动接受编辑类操作）；`bypassPermissions` 与 dangerous 等价，**仅隔离环境**。 |
| `claudeCode.allowDangerouslySkipPermissions` | 为 `true` 时，模式选择器里才出现 **Auto / Bypass** 等选项（仍受官方对 Auto 模式的计划/账号等限制）。 |
| `claudeCode.useTerminal` | 为 `true` 时用 **终端式 Claude**（接近纯 CLI），长命令输出、滚动习惯与 `claude` TUI 更一致。 |
| `claudeCode.autosave` | 默认 `true`：Claude 读写前自动保存，减少「未保存 buffer」与工具读盘不一致。 |
| `claudeCode.disableLoginPrompt` | 使用 Bedrock/Vertex 等第三方鉴权时减少登录弹窗（按官方文档配好 env 后再开）。 |

**示例（按需裁剪；`bypassPermissions` 勿在主力机上长期用）：**

```json
{
  "claudeCode.initialPermissionMode": "acceptEdits",
  "claudeCode.allowDangerouslySkipPermissions": true,
  "claudeCode.useTerminal": false,
  "claudeCode.autosave": true
}
```

与本文「Claude Code」一节一致：**真正消除 Bash/网络等确认**，仍依赖 `~/.claude/settings.json` 的 `permissions` / `bypassPermissions`；VS Code 里两项是 **入口体验与编辑类自动接受**，不要混为一谈。

#### 长时间跑时建议一并调整的 VS Code 通用设置

| 配置键 | 建议 | 原因 |
| --- | --- | --- |
| `terminal.integrated.enablePersistentSessions` | `true`（若你的 VS Code 版本支持） | 重开窗口后尽量恢复终端会话，长任务不易丢。 |
| `terminal.integrated.confirmOnExit` / `confirmOnKill` | 按习惯设为 `always` 或至少对 **有子进程** 确认 | 防止误关终端杀掉跑了很久的 agent。 |
| `files.autoSave` | `afterDelay` + 合理 `delay` | 与 `claudeCode.autosave` 叠加，减少人为未保存。 |
| `window.confirmBeforeClose` | `always`（可选） | 避免误点关整个窗口中断任务。 |

**工作方式**：长任务用 **单独窗口或单独 Tab 会话**（命令面板搜 `Claude Code: Open in New Window` / New Tab），主窗口继续日常编辑，降低误操作概率。官方文档说明长时间命令时 **状态栏有进度**，复杂 tail 类可在集成终端里让模型给出命令自行观察。

---

### OpenCode：扩展与 CLI

OpenCode 的 **CLI 仍是真身**；VS Code 侧常见几种装法：

| 扩展 ID | 说明 |
| --- | --- |
| `sst-dev.opencode` | **官方**一类：在编辑器里 **分屏打开集成终端里的 opencode**，快捷键 `Cmd+Esc` / `Ctrl+Esc` 聚焦等（见 [OpenCode IDE 文档](https://open-code.ai/en/docs/ide)）。 |
| `sst-dev.opencode-v2` | **Beta**：侧栏增强聊天、多会话等，仍依赖本机已安装 `opencode` CLI。 |
| `TanishqKancharla.opencode-vscode` | **社区** OpenCode GUI，**非 SST 官方维护**；装前看 README 与 issue，与官方 CLI 版本是否匹配需自行验证。 |

**安装路径：**

1. **推荐**：集成终端里执行 `opencode`，不少环境下会 **自动触发/提示安装** 官方配套扩展（与文档 [IDE Integration](https://open-code.ai/en/docs/ide) 一致）。
2. **手动**：扩展市场搜索「OpenCode」或上述 ID；并确保 `which opencode`（Windows 用 `where opencode`）能命中 CLI，装完 CLI 后 **完全重启 VS Code** 以刷新扩展宿主 PATH。

**版本**：同样在扩展详情页看 **Version**；锁版本方式与 Claude 一节相同，例如 `code --install-extension sst-dev.opencode@x.y.z`。

**与「少确认、长跑」相关的配置**：仍以 **`~/.config/opencode/config.json`**（或项目根 `opencode.json`）为主，见本文 OpenCode 小节中的 `permission` / `yolo` / `skipDangerousModePermissionPrompt`；在 VS Code 里长跑时可在终端用：

```bash
opencode --yolo
```

或在配置里长期 `yolo: true`（风险同 bypass，**仅建议隔离环境**）。**TUI 的 `tips: false`** 等也在同一 JSON，减少底部干扰。

**编辑器环境变量**：在 TUI 里用 `/editor`、`/export` 等需要外部编辑器时，可设 `export EDITOR="code --wait"`（Cursor 则换成 `cursor --wait`），避免挂起或弹错编辑器。

---

### 长时间运行检查清单（偏设置）

1. **权限层**：Claude 用 `~/.claude/settings.json`；OpenCode 用 `~/.config/opencode/config.json`（或 CLI 参数 `--yolo`），与 IDE 扩展里的「默认权限模式」一起对齐预期。
2. **IDE 层**：`claudeCode.*` + 终端持久化/关窗确认 + `files.autoSave`。
3. **系统层**：合盖不睡眠、或 `caffeinate`（macOS）、电源计划（Windows）、防止 Wi‑Fi 节能断流。
4. **隔离层**：真正「无人值守 + 高权限」优先 **Docker / VM / CI**，与本文 Claude 小节的安全提醒一致。

## 容器化

### 使用 Docker 运行 OpenCode

```shell
docker run -it --rm ghcr.io/anomalyco/opencode
```
