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

## 容器化

### 使用 Docker 运行 OpenCode

```shell
docker run -it --rm ghcr.io/anomalyco/opencode
```
