# FRP 内网穿透与 P2P/STCP 模式详细指南

本文档总结了我们对 FRP（Fast Reverse Proxy）相关功能、模式、配置及实践经验的讨论，方便快速参考和部署。

---

## 1. FRP 简介

FRP 是一款 **高性能反向代理应用**，用于将内网服务通过公网服务器映射到外网访问。它主要有两类组件：

* **frps**：服务器端，部署在公网服务器
* **frpc**：客户端，部署在内网机器或需要访问内网服务的机器

典型访问流程：

```
外网访问 → 公网服务器 frps → 内网 frpc → 内网服务
```

FRP 支持多种协议类型：TCP、UDP、HTTP、HTTPS 以及 P2P 模式（xtcp/stcp）。

---

## 2. frps 配置示例

FRP 服务器端配置主要位于 `frps.ini` 文件：

```ini
[common]
bind_port = 7000
#vhost_http_port = 80
#vhost_https_port = 443
token = 123456
```

说明：

* [common] 固定，不可改
* `bind_port`：frps 监听 frpc 连接的端口
* `token`：客户端和服务器认证令牌
* `vhost_http_port` / `vhost_https_port`：HTTP/HTTPS 映射端口，可选

可选 TLS 证书配置：

```ini
vhost_https_crt = /path/to/server.crt
vhost_https_key = /path/to/server.key
```

* 支持自签证书或 Let’s Encrypt 证书
* 用于 HTTPS 模式的加密

---

## 3. frpc 配置解析

### 3.1 [common] 全局配置

```ini
[common]
server_addr = 公网服务器IP或域名
server_port = 绑定端口
token = 认证密码
```

说明：

* [common] 固定，不可改
* `server_addr`：公网服务器 IP 或域名
* `server_port`：frps监听端口
* `token`：客户端和服务器认证令牌

### 3.2 HTTP/HTTPS 映射

示例：

```ini
[domain_nas_http]
type = http
local_ip = 192.168.1.100
local_port = 5000
custom_domains = nas.example.com

[domain_nas_https]
type = https
local_ip = 192.168.1.2
local_port = 443
custom_domains = esxi.example.com
```

说明：

* HTTP/HTTPS 模式 **必须使用真实域名**，因为 frps 根据域名匹配请求的 **Host（HTTP）或 SNI（HTTPS）**
* `custom_domains` 用于 frps 匹配请求的 **Host（HTTP）或 SNI（HTTPS）**
* 外网访问：

```
http://nas.example.com
https://esxi.example.com
```

### 3.3 TCP 映射

```ini
[domain_nas_drive]
type = tcp
local_ip = 192.168.1.100
local_port = 6690
remote_port = 26690
```

说明：

* TCP 不依赖域名
* 外网访问：`服务器IP:26690`
* 适合非 HTTP 服务（SSH、数据库、RDP）

### 3.4 [.ini] 中 [] 名称规则

* `[common]` 固定，不可改
* 其他代理名称 `[xxx]` 可以自定义，但**不能重复**，只允许英文、数字、下划线
* 每个 [] 对应一条穿透规则

---

## 4. HTTPS 模式与 SNI

* **SNI（Server Name Indication）**：TLS 扩展，客户端在握手时发送目标域名
* FRP 根据 SNI 选择内网服务并匹配证书
* 没有域名或客户端不发送 SNI，HTTPS 连接会失败

### 证书来源

1. **FRPS 提供证书**（自签或 CA 证书）
2. **Let’s Encrypt 自动签发**（推荐）
3. **内网服务自带 HTTPS**，配合 TCP 映射使用

---

## 5. 延迟与性能

* FRP 本身延迟几毫秒，不是瓶颈
* 延迟主要由 **服务器位置 + 网络质量 + 家庭上行带宽** 决定
* 同城服务器 + 家庭宽带：总延迟 10~60ms
* 跨国服务器：150~300ms+，明显增加延迟
* 优化：选择就近服务器、BGP/CN2 网络，必要时使用 P2P 直连

---

## 6. P2P 模式（xtcp）

### 6.1 原理

* 服务器仅协助打洞，不参与数据传输
* 客户端与内网机器直接建立点对点连接
* 利用 UDP/NAT 打洞实现穿透

### 6.2 配置示例

**内网 frpc**

```ini
[ssh_p2p]
type = xtcp
sk = 123456
local_ip = 127.0.0.1
local_port = 22
```

**外网访问 frpc (visitor)**

```ini
[ssh_p2p_visitor]
type = xtcp
server_name = ssh_p2p
sk = 123456
bind_addr = 127.0.0.1
bind_port = 6000
```

访问方式：

```bash
ssh user@127.0.0.1 -p 6000
```

### 6.3 特点

* 延迟低，数据直连内网，不占用服务器带宽
* 外网客户端必须运行 frpc，负责建立连接和打洞
* 不支持 HTTP/HTTPS vhost
* 打洞失败时不会自动降级到 TCP 中转，需要手动改配置

### 6.4 端口使用原则

* 内网 SSH 服务仍监听 22，frpc 多个代理段可以共用 22
* 外网访问端口必须不同，避免监听冲突

---

## 7. STCP 模式（Secure TCP）

* STCP 强调 **安全、可靠的内网服务映射**
* 数据全程加密，必须 `sk` 验证
* 依赖 frps 中转，无需 NAT 打洞成功
* 适合内网数据库、RDP、SSH 等私有服务

示例：

**内网 frpc**

```ini
[internal_db]
type = stcp
sk = mysecret
local_ip = 127.0.0.1
local_port = 3306
```

**外网 frpc visitor**

```ini
[db_visitor]
type = stcp
server_name = internal_db
sk = mysecret
bind_addr = 127.0.0.1
bind_port = 3306
```

访问方式：

```bash
mysql -h 127.0.0.1 -P 3306 -u user -p
```

### 7.1 STCP vs XTCP

| 模式   | 特点                  | 使用场景            |
| ---- | ------------------- | --------------- |
| xtcp | P2P直连，低延迟，依赖 NAT 打洞 | SSH、RDP、游戏、开发调试 |
| stcp | 安全中转，可靠，必须 sk       | 内网私有服务、数据库、公司服务 |

---

## 8. 总结

* **HTTP/HTTPS**：必须域名，FRP 根据 Host/SNI 转发，证书可自签或 Let’s Encrypt
* **TCP**：不依赖域名，可映射任意服务
* **P2P/xtcp**：低延迟直连，外网客户端需运行 frpc
* **STCP**：安全可靠中转，适合私有服务
* **端口管理**：内网可共用端口，外网访问端口需不同
* **frps.ini** 必须配置 bind_port、token，可选 TLS 证书、 vhost_http_port/vhost_https_port

---

文档完毕，可用于部署参考。
