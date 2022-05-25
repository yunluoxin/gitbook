# 把一个 Git 仓库推送到多个远程仓库的方法

## 场景

​	现在你在弄的一个项目，于公司内部私有的 `gitlab` 仓库开发，现在又要开源到 `github`，这时候你要怎么办？把代码拷贝一份，然后提交到 `github`? 第一次自然是没问题的，但是之后修改了代码，要如何同步？一行行复制，一个个文件替换？然后再次写 `commit`？这是一个麻烦又头疼的问题。

​	我们需要的是，改一次代码，只一次 `commit`，就能直接推送、发布到多个远程终端。



## 简单版（2分钟即可掌握）

1. 定义一个 `git remote`，这里取名为 `all`, 我们会把它指向多个远程仓库地址。

   ```shell
   git remote add all REMOTE-URL-1
   ```

2. 添加第 1 个你要添加的远程仓库 A，假设地址为 REMOTE-URL-1

   ```shell
   git remote set-url --add --push all REMOTE-URL-1
   ```

3. 同样的，添加第 2 个你要添加的远程仓库 B，假设地址为 REMOTE-URL-2

   ```shell
   git remote set-url --add --push all REMOTE-URL-2
   ```

4. 一次推送代码到多个远程仓库

   ```shell
   # BRANCH 是你要推送的分支的名字
   git push all BRANCH
   ```

   

## 详细版

### 前置

​	你需要对 `git remote` 以及其他基本的 git 命令有所了解

### 增加多个 `remote`

```shell
git remote add REMOTE-ID REMOTE-URL
```

​	其中 `REMOTE-ID` 为你给某个远程地址所取的名字，一般我们都使用 `origin`,

​			`REMOTE-URL`为远程仓库的地址。

你可以为每一个远程仓库添加为一个 `git remote`, 而不是上面简单版本的，一个 `remote` 包含多个 `url`。

一个例子：

```shell
git remote add origin git@xxx.private.com:east/test.git		# 添加公司内部的仓库
git remote add github git@xxx.github.com:east/test.git	# 添加github远程仓库
```

### 修改仓库的远程地址

假设你把 `github` 上的仓库，移动了位置。那么远程仓库地址会改变，这时候就要修改本地仓库跟踪的 `remote`：

```shell
git remote set-url origin git@xxx.github.com:east/test.git
```

### 配置本地和远程的跟踪

```shell
# 切换远程分支
git checkout BRANCH
# 配置当前本地分支，跟踪某个远程分支
git branch -u origin/BRANCH
```

​	你可以对当前的仓库的不同本地分支，分别跟踪不同 `remote` 的不同分支！

### 拉取代码

​	同时从对个 `remote` 拉取代码是不可能的，但是，却可以同时抓取远程的信息

```shell
git fetch --all
```



## 其它

查看当前仓库所有的 `remote`

```shell
git remote -v
```

移除` remote`

```shell
git remote remove upstream
```



## 参考

[1]. [Working with Git remotes and pushing to multiple Git repositories](https://jigarius.com/blog/multiple-git-remote-repositories)