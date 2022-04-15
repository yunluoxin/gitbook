# 如何彻底删除 M1 电脑上的 iPhone App

​	发现在 M1 电脑上安装 iPhone 或者 iPad App 后，不管用正常方式（直接在 `LaunchPad` 中删除），还是 `AppCleaner` 这类软件，都无法清理干净。 当你再次下载 app 之后，打开一开，还是上次的状态，就和没卸载重装过似的！

经过我搜索关键词 `m1 how to delete iphone app cache`，认真看了每个回复。我终于看到了一个网友提的[解决方案](https://forums.macrumors.com/threads/how-to-uninstall-ios-ipados-apps-on-m1-macs-and-where-do-they-store-their-files.2275923/?post=29453766#post-29453766)：

```shell
cd ~/Library/Containers
find . -iname "你要删的app名字"		# 名字可以带 * 进行模糊查询
# 或者
find . -name "你要删除的app名字"		# 不带i，不区分大小写。上面的那个区分
```

经过尝试，发现会出现很多的 `Operation not permitted`：

```shell
find: ./com.apple.VoiceMemos: Operation not permitted
find: ./com.apple.archiveutility: Operation not permitted
find: ./com.apple.Home: Operation not permitted
find: ./com.apple.Safari: Operation not permitted
find: ./com.apple.CloudDocs.MobileDocumentsFileProvider: Operation not permitted
./02712D28-2061-4D3F-ABB3-BB9EF821F56F/Data/Documents/XXXApp
find: ./com.apple.mail: Operation not permitted
find: ./com.apple.Notes: Operation not permitted
find: ./com.apple.news: Operation not permitted
./5C568457-A16E-47C1-8F9C-F5A166756CD4/Data/Library/Caches/com.apple.dyld/XXXApp_Example.closure
find: ./com.apple.corerecents.recentsd/Data/Library/Recents: Operation not permitted
find: ./com.apple.stocks: Operation not permitted
```

不用管那些错误的，没错误的那些行，就是你要找的。辨认好后，复制出他们的 `UUID`：

```shell
rm -rf 02712D28-2061-4D3F-ABB3-BB9EF821F56F
rm -rf 5C568457-A16E-47C1-8F9C-F5A166756CD4
```

删光他们！！！

