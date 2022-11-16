# 如果制作MacApp的dmg安装包

## 手动制作

### 新建一个空白的 dmg 磁盘镜像

磁盘工具	→	文件	→	新建映射	→	空白映像

名称改为你的app名字`ABC`，大小设置为大概你要存储的 `app` 大小 + 几 MB，保存为 `xxx.dmg`

eg: 	你的app为20MB，你可以设置为30MB，多余的10MB用来给App后期增长以及背景图片等资源。



### 拷贝app和其他资源到dmg中

双击打开刚生成的 `xxx.dmg`，你会发现` Finder`的左侧边栏或者左面生成一个新的磁盘，打开是空白的。把你的app、要设置的背景图片等资源，直接复制到里面。

经常我们打开别人的app，你会发现有一个`应用程序`图标，我们直接把app拖进去就能安装，这里我们也弄一个：

```shell
ln -s /Applications/Volumes/ABC/	# 其中ABC为你的app名字
```

这时候你就会发现，打开的dmg里面多了一个链接文件，指向`/Applications` 的！



### 设置背景图片

依旧是刚刚打开的`xxx.dmg`里，在`Finder`空白处右击	→	查看显示选项	→	背景，将刚刚添加的背景图片拖拽进来，这时候你会发现当前的`Finder`有了背景！

> 如果没有背景，看下`Finder`顶部的显示方式是否为`图标`，必须是设置为`图标`哦！

我们发现平时下的app里，一般就app和一个应用程序图标，没有第三个，但是却有背景，那是怎么回事？因为他们把背景图片隐藏了，我们也这么做：

```shell
chflags hidden /Volumes/ABC/bg.png	# 其中ABC为你的app名字，bg.png 为你背景图片名字，修改为你自己的
```

好了。这些镜像里，只能看到两个图标啦！

为了和你的背景图片配合，你还可以改变两个图标的位置，让整个画面更协调！也是在`查看显示选项`里，`间距`和`图标大小`什么的都可以尝试下。



### 压缩磁盘镜像

因为刚才建立的磁盘镜像，可能存在未使用完的空间！以及本身已使用的内容也能被压缩，所以这一步的压缩不可或缺，不然镜像会显得特别大！

路径为：

​	磁盘工具	→	映像	→	转换	→	设置为压缩

要不要加密的，自由选择！一般拿去分发的软件，都不设置密码哈！

经过这一步的压缩，10MB的dmg，最后可能仅为1MB+！





## 用脚本制作镜像

### 创建一个固定占位空间的镜像

```shell
hdiutil create -size 100m  -type UDIF -fs 'APFS' -volname "MyDMG" myDMG.dmg
# volname 是镜像文件被挂载后，显示的名字。 也是出现在 /Volumes/XX 的名字
```



### 从指定的文件或者文件夹创建镜像

```shell
hdiutil create -srcfolder "文件或者文件夹的路径"  -fs 'APFS' -volname "MyDMG" myDMG.dmg
# 注意： 
#		1. 用了 -scrfolder 之后，无法用到 -size。 也就是它默认就是创建和你传入的文件一样大的+一些空闲的文件。
# 	2. 传入文件夹路径时候，是把你的文件夹下的所有文件加入，不包含文件夹本身。 比如你用 -scrfolder "ABC", 则会把你 ABC 文件夹下的所有文件都添加到镜像中，但是不包含 ABC 文件夹。 所以如果你需要带文件夹传入的话，自己再包裹一层。
```



### 压缩镜像文件

```shell
hdiutil convert "$temp" -format UDZO -imagekey zlib-level=9 -o "$target"  # 压缩镜像
# $temp   为你要需要压缩的文件地址
# $target 为新生成的目标文件地址
```



### 挂载镜像文件

```shell
hdiutil attach "$name"	            # 挂载镜像
# $name 是需要挂载的dmg镜像文件的地址

# 也可以用这个判断是否挂载成功
name=$(hdiutil info | grep "/Volumes/XX" | cut -f1)	# /dev/diskf1s 之类的
if [[ -z "$name" ]]; then
    echo "找不到已经挂载的镜像，挂载失败!"
    exit -1
f
```



### 卸载镜像文件

```shell
hdiutil detach "$name"		# 卸载镜像
# $name 一般为 /Volumes/XX
```



### 复制外部文件到挂载后的镜像里

前提： 把镜像文件挂载！不管是用双击还是用脚本挂载哦。挂载后会在 `Finder` 和 `桌面` 生成一个虚拟盘符，类似U盘一样的东西。。

当我用 `cp` 命令复制到镜像里的时候一直失败，直接拖拽到里面，却成功了？后面发现，复制时候用的目的地址不能用 `/dev/diskxx` 这种，要用 `/Volumes/XX` 这种。然后成功了！

```shell
cp -r source_file_or_dir /Volumes/XX
```

> 注意： 这里的镜像文件，我是用第一个那种，有空闲空间的固定大小镜像文件。 压缩过后的，挂载后，我不知道能不能存进去了。