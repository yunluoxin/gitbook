# M1上如何直接安装 ipa 包



## 前言

Mac OS 11.3 系统及之后，苹果禁止了 ipa 包的直接安装运行，必须是通过 appstore 上分发的 app 才能进行使用了，这太坑爹了。很多手机软件不上架在 Mac 的 appstore 中，我们便无法使用了。这时候，只能回滚系统，但是麻烦太大，于是有了这个！



## 下载 ipa 包

ipa 包不能是随意的，必须是手机 appstore 上真有的。 （之前可能都可以，现在还是没法做到之前的！）

如何获取 ipa 包，见 [这个文章](./Apple Configurator 2 下载手机 ipa 包.md)



## 安装使用

下载之后，双击 ipa 包，即可安装到 Application 中，但是点开使用，还是会报错：`在您批准将其在系统上运行之前，iOS应用程序将无法运行！` 执行

```shell
sudo xattr -rd com.apple.quarantine /Application/你安装的app
```

其中 `/Application/你安装的app` 为你安装的app的路径！

然后， 你便可以使用啦！



## 参考

1. [如何在Mac上下载和运行iOS应用](https://reincubate.com/zh/support/how-to/download-run-ios-app-mac/)
2. [使用Apple Configurator 2下载IPA包](https://www.jianshu.com/p/664fb3bdef0d)