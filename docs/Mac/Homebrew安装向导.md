# Homebrew安装向导 - ARM架构的Mac电脑

## 安装

### 安装 ARM 版 Homebrew

```shell
/bin/bash -c "$(curl -fsSL https://gitee.com/ineo6/homebrew-install/raw/master/install.sh)"
```

等待安装完成后，再执行下面两行代码：

```shell
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

> ARM版Homebrew会自动安装在 `/opt/homebrew` 路径下


### 安装 X86 版 Homebrew

```shell
arch -x86_64 /bin/bash -c "$(curl -fsSL https://gitee.com/ineo6/homebrew-install/raw/master/install.sh)"
```

> x86版Homebrew会自动安装在 `/usr/local` 路径下


### *同时安装两种版本的Homebrew

由于两种版本安装后的位置不一样，这给我们同时安装、使用两个版本提供了可能。如果你在用的软件，不支持arm版本的homebrew，可以考虑同时安装两个！

修改 `~/.zprofile` 文件，在最后面追加下面两行后保存文件：

```shell
alias abrew='arch -arm64 /opt/homebrew/bin/brew'
alias ibrew='arch -x86_64 /usr/local/bin/brew'
```

其中，abrew、ibrew 可以根据你的喜好自定义。

然后你可以执行 `source ~/.zprofile` 直接当前shell生效或者重新打开一个新的shell！


## 加速 - 设置国内镜像

```shell
# brew
git -C "$(brew --repo)" remote set-url origin https://mirrors.ustc.edu.cn/brew.git

# core
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git

# cask
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git

# bottles for zsh
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles/bottles' >> ~/.zprofile
source ~/.zprofile
```

可以通过 `brew config` 查看 homebrew 的各项配置。

注意：
> - 如果出错了，提示文件夹不存在，就别管了，有需要后面再回来设置。
> - 如果你安装了两个版本的homebrew，都需要设置，可以把 **$(brew --repo)** 中的 brew 改成对应的别名。分别执行一次。


## 其他命令

搜索可用的预编译二进制包（Bottles）：

```shell
brew search <软件包名称>
```

安装预编译二进制包：

如果有可用的预编译二进制包，则使用以下命令直接安装它：

```shell
brew install --force-bottle <软件包名称>
```


## *其他知识

### 它的cask和bottle有什么区别？

以下是来自 ChatGPT 的回答：

`brew cask` 和 `brew bottle` 是 Homebrew 的两个不同功能，用于安装不同类型的软件包。这里分别介绍它们的主要区别：

**Bottle**

- Bottle 是预编译好的二进制软件包，适用于命令行工具和库。

- 它们可以加快安装过程，因为不需要从源代码进行编译。

- 使用以下命令安装 Bottle 软件包：

    ```shell
    brew install --force-bottle <软件包名称>
    ```

**Cask**

- Cask 则用于安装 macOS 的图形界面应用程序（GUI 应用程序）。

- 它扩展了 Homebrew 的功能，使其能够直接处理原生的 macOS 应用程序、字体、插件等。

- 使用以下命令安装 Cask 软件包：

    ```shell
    brew install --cask <软件包名称>
    ```

总结一下，Bottle 是针对命令行工具和库的预编译二进制包，可以加速安装过程；而 Cask 是用于处理 macOS 图形界面应用程序等相关内容的扩展功能。在安装软件时，您可以根据实际需求选择使用哪种方式。


## 参考

1. [ARM64 版本 Mac 的 Homebrew 安装教程](https://zhuanlan.zhihu.com/p/341831809)