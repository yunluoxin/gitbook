# Mac 下利用 Cloudflare Tunnel 实现安全 SSH 连接

## 一、教程说明

本教程适用于 macOS 系统，通过 Cloudflare Tunnel（cloudflared）实现 SSH 安全连接，无需暴露服务器公网 IP 和 22 端口，所有流量经 Cloudflare 加密转发，兼顾安全性和便捷性。

## 二、前置条件

1. 拥有 Cloudflare 账号，且已将域名（比如是 `yourdomain.com`）托管到 Cloudflare；

2. 服务端（目标 SSH 服务器）：Linux/macOS 系统，可联网且已安装 SSH 服务；
> 如果是 mac，可以直接去共享里面开启 ssh 服务

3. 客户端（Mac 本地）：已安装 Homebrew（方便安装 cloudflared）。

## 三、服务端配置（Linux/macOS 通用）

### 3.1 安装 cloudflared

```Bash

# Linux 系统
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb

# macOS 系统（若服务端是 Mac）
brew install cloudflared
```

### 3.2 认证 cloudflared（关联 Cloudflare 账号）

```Bash

cloudflared tunnel login
```

执行后会自动打开浏览器，登录 Cloudflare 账号并授权，授权成功后会生成认证文件（默认路径：`~/.cloudflared/cert.pem`）。

### 3.3 创建 Cloudflare Tunnel

```Bash

# 创建名为 my-ssh 的 Tunnel（名称可自定义）
cloudflared tunnel create my-ssh
```

执行后会输出 **Tunnel ID**（需记录，如 `3758bd65-196c-4b81-9ac1-960cb37afa0d`），并生成凭证文件（路径：`~/.cloudflared/<Tunnel ID>.json`）。

### 3.4 编写 Tunnel 配置文件

创建 `/etc/cloudflared/config.yml`（Linux）或 `~/.cloudflared/config.yml`（macOS 服务端），内容如下：

```YAML

tunnel: 3758bd65-196c-4b81-9ac1-960cb37afa0d  # 替换为你的 Tunnel ID 或者 tunnel 名
credentials-file: /root/.cloudflared/3758bd65-196c-4b81-9ac1-960cb37afa0d.json  # 替换为凭证文件实际路径

# 流量路由规则：域名转发到本地 SSH 服务
ingress:
  - hostname: ssh.yourdomain.com  # 替换为你的 Cloudflare 域名（需和后续 DNS 关联一致）
    service: ssh://127.0.0.1:22  # 本地 SSH 服务地址（默认 22 端口）
  - service: http_status:404  # 兜底规则：未匹配流量返回 404
```

### 3.5 关联 Tunnel 到域名

```Bash

# 将 Tunnel 关联到指定域名（需和 config.yml 中的 hostname 一致）
cloudflared tunnel route dns my-ssh ssh.yourdomain.com
```
> 这里，在自己的域名前添加 ssh, cloudflare 会自动创建子域名并转发

执行后会在 Cloudflare 域名解析中自动添加 CNAME 记录，验证是否关联成功：

```Bash

cloudflared tunnel route dns list my-ssh
```

> 也可以通过这个命令，查看当前的 tunnel 名和 dns 的映射！

### 3.6 启动并持久化 cloudflared 服务

```Bash

# Linux 系统（以 systemd 为例）
sudo cloudflared service install
sudo systemctl start cloudflared
sudo systemctl enable cloudflared  # 开机自启

# macOS 系统（服务端）
# 方式1：临时启动（测试用）
cloudflared tunnel run my-ssh
# 方式2：配置开机自启（略，可参考 Cloudflare 官方文档）
```

### 3.7 验证服务端 SSH 基础配置

确保服务端 SSH 服务正常运行，且已配置好 SSH 密钥（如 Ed25519）：

```Bash

# 检查 SSH 服务状态
systemctl status sshd  # Linux
sudo launchctl list | grep ssh  # macOS

# 确保 authorized_keys 权限正确
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

## 四、Mac 客户端配置

### 4.1 安装 cloudflared

```Bash

# 通过 Homebrew 安装（推荐）
brew install cloudflared

# 验证安装成功
cloudflared -v
```

### 4.2 配置 SSH Config 文件

编辑 Mac 本地的 `~/.ssh/config` 文件（无则创建）：

```YMAL

# Cloudflare Tunnel SSH 配置
Host my-ssh  # 自定义连接别名（如 my-ssh）
  HostName ssh.yourdomain.com # 替换为和服务端关联的域名
  User east  # 替换为服务端的实际登录用户名
  # 核心：通过 cloudflared 转发 SSH 流量
  ProxyCommand cloudflared access ssh --hostname %h
  # 可选：指定 SSH 私钥路径（如 Ed25519 密钥）
  IdentityFile ~/.ssh/id_ed25519
  # 防止长连接断开
  ServerAliveInterval 30
  # 首次连接跳过主机密钥确认（可选）
  StrictHostKeyChecking no
```

### 4.3 测试 SSH 连接

```Bash

# 使用自定义别名连接
ssh my-ssh
```

若配置正确，会自动通过 Cloudflare Tunnel 建立连接，无需输入复杂的 IP/端口/密钥参数。

## 五、常见问题排查

### 5.1 websocket: bad handshake（WebSocket 握手失败）

- **原因**：域名不匹配、Tunnel 未运行、config.yml 配置错误；

- **解决方案**：

    1. 确保「服务端 config.yml 的 hostname」「Tunnel 关联的域名」「客户端 SSH Config 的 HostName」三者完全一致；

    2. 检查服务端 cloudflared 服务是否运行：`sudo systemctl status cloudflared`；

    3. 验证 Tunnel 在线状态：`cloudflared tunnel list`（需显示 Connections 节点）。

### 5.2 kex_exchange_identification: Connection closed by remote host

- **原因**：ProxyCommand 格式错误、cloudflared 未安装、服务端 SSH 未运行；

- **解决方案**：

    1. 确保 ProxyCommand 格式正确：`cloudflared access ssh --hostname %h`（无多余的 `--user %u` 参数）；

    2. 验证客户端 cloudflared 安装：`cloudflared -v`；

    3. 服务端测试本地 SSH：`ssh localhost`。

### 5.3 权限错误

- **服务端**：确保 `~/.cloudflared` 目录权限为 700，凭证文件权限为 600；

- **客户端**：确保 `~/.ssh/config` 权限为 600，私钥文件权限为 600。

## 六、关键注意事项

1. 所有环节的域名必须完全一致（如均使用 `ssh.yourdomain.com`），否则流量路由失败；

2. cloudflared Tunnel 启动后，服务端无需开放 22 端口，仅需允许 cloudflared 出站访问 443 端口；

3. 若需增强安全性，可在 Cloudflare Zero Trust 中配置 Access 策略（如邮箱/OAuth 认证），限制仅指定用户可连接；

4. 定期更新 cloudflared：`brew upgrade cloudflared`（Mac 客户端）/ `sudo apt upgrade cloudflared`（Linux 服务端）。

## 七、总结

1. 核心流程：服务端创建 Tunnel → 配置域名路由 → 客户端配置 SSH Config 关联 Tunnel 域名；

2. 核心优势：无需公网 IP、无需开放 22 端口、流量加密转发、连接命令简化（`ssh 别名` 即可）；

3. 排错重点：域名一致性、cloudflared 服务状态、ProxyCommand 格式。
> （注：文档部分内容可能由 AI 生成）