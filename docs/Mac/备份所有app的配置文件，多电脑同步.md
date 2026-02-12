# 备份所有 app 的配置文件，多电脑同步

## 大概原理

- **想要的效果**
  - 在本地电脑上，使用 `git` 管理所有 app 的配置文件，然后通过 `git` 的同步功能，将配置文件同步到其他电脑上。
  - 核心点: `git` 仓库里的文件，是真正的文件，而源文件的地方其实是一个符号链接。
- **实际上的**：
  - 当前是也通过 `git` 管理了，但是是从仓库复制一份文件到源文件的地方，不是通过符号链接！所以，源文件或者仓库一旦改了，两者之间就不一致。
  - 仓库 --> 本地文件/文件夹，通过命令 `apply`
  - 源文件/文件夹 --> 仓库，通过命令 `add`
  - 如果源文件/文件夹改了，仓库不会自动同步，需要再次执行 `add` 命令。

## 工具

`chezmoi` 是一个工具，可以用来管理所有 app 的配置文件。

## 安装

```shell
brew install chezmoi
```

> 😄 如果你不是首次使用，已经有备份了，看【新电脑加入】章节 😄

## 初次使用

初始化仓库

```shell
chezmoi init
```

> 初始化完成后，会看到一个 `~/.local/share/chezmoi` 文件夹，里面是存放配置文件的仓库

然后，进入这个仓库

```shell
# 它会启用一个子shell，直接帮你进入它内建的仓库目录
chezmoi cd
```

和自己的远程仓库进行关联，比如 github 的

```shell
git remote add git@github.com/xxxx
```

### 添加你的配置文件

```shell
# 比如添加 .zshrc 文件
# chezmoi add <file>
chezmoi add ~/.zshrc
```

> 如果你不想写全路径，就进入你的配置文件夹，然后使用 `chezmoi add .` 添加所有文件，或者使用 `chezmoi add xxx` 添加单个文件。

记住这个时刻 (git commit)，后续要同步的时候，直接从这个时刻开始同步。

```shell
git commit -m "add ~/.zshrc"
```

推送代码到远程仓库

```shell
git push origin -u main
```

### 应用配置文件

如果你在其他电脑上变更了，想要覆盖当前电脑，就执行这个命令

首先拉取，你可以手动去仓库 pull 代码，然后应用配置文件

```shell
chezmoi apply
```

或者，使用 chezmoi 的命令，它会自动拉取代码，然后应用配置文件

```shell
chezmoi update
# 或者，指定某个时刻??? 未测试
chezmoi update --since <commit-hash>

# 应用配置文件
chezmoi apply
```

## ⭐️ 新电脑加入 ⭐️

新电脑加入的时候，直接执行这个命令，会自动拉取远程仓库的配置文件，然后应用配置文件。

```shell
chezmoi init --apply https://github.com/$GITHUB_USERNAME/dotfiles.git

# 如果是 github 的仓库，并且你使用的仓库名叫做 dotfiles，可以更简单：
# 拉取并应用到本地电脑
chezmoi init --apply $GITHUB_USERNAME

# 仅仅拉取github上名为xxx的仓库进行初始化
chezmoi init $GITHUB_USERNAME
```

## 其他

### 查看配置文件的变更

```shell
chezmoi diff
```

### 命令太麻烦？ 改个名字

去 `~/.zshrc` 文件里，添加如下代码：

```shell
alias cz='chezmoi'
```

然后执行 `source ~/.zshrc` 使配置生效。

这样你就可以直接执行 `cz` 命令来代替 `chezmoi` 命令了。

## 附录

1. [chezmoi github](https://github.com/twpayne/chezmoi)
2. [chezmoi 快速入门](https://www.chezmoi.io/quick-start/)
