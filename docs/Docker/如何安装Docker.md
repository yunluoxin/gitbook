# 如何安装 Docker

1. 直接安装 Docker Desktop
- https://www.docker.com/products/docker-desktop/ 下载安装
- https://docs.docker.com/desktop/install/mac-install/ 下载安装

两个都需要 *VPN*。并且，**在企业环境中，可能必须付费而导致无法使用！**

2. 通过 homebrew 安装

```shell
brew install --cask --appdir=/Applications docker
```

未测试，不知道行不行。这个只能安装 Docker 引擎，没有 Docker Desktop 那样的界面。

> 没有 homebrew 的，需要自行先去安装。

---

## 安装 Docker Compose

- https://docs.docker.com/compose/install/ 下载安装
- 直接去开源的 github 上下载安装，地址是 [https://github.com/docker/compose](https://github.com/docker/compose)