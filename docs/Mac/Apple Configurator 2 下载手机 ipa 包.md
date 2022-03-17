# 通过 Apple Configurator 2 下载手机 ipa 包

1. 下载 Apple Configurator 2 软件，直接 Appstore 就可以下载
2. 在你的手机中先下载你要提取的软件，必须先下载！
3. 手机链接电脑，在Configurator 2 中搜索应用，下载安装
4. 当下载完成后会弹出提示，已经安装过此引用，此时什么都不要动！什么都不要动！什么都不要动！打开终端，输入：

```shell
cd ~/Library/Group\ Containers/K36BKF7T3D.group.com.apple.configurator/Library/Caches/Assets/TemporaryItems/MobileApps && open .
```

这个路径下就有你要的安装包了！

> 如果你用的是 M1 电脑，想要在 M1 上直接运行此 ipa，可以看 [M1上如何直接安装 ipa 包](./M1上如何直接安装 ipa 包.md)



## 参考

1. [如何在Mac上下载和运行iOS应用](https://reincubate.com/zh/support/how-to/download-run-ios-app-mac/)
2. [使用Apple Configurator 2下载IPA包](https://www.jianshu.com/p/664fb3bdef0d)