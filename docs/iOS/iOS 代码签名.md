# Code Signing

使用 `security` 命令，可以查看系统中的这类 app 签名使用的证书

```shell
security find-identity -v -p codesigning
```



对 `.app` 内部的 `.framework` 文件、`.dylib` 文件、PlugIns 目录里的 .appex 文件、Watch 目录里的 `.app` 文件分别进行签名 

```shell
codesign -f -s 证书ID  xxx.dylib
```

> 发现有个别 `.framework` 库文件里面，又有动态的 `framework` 文件，这时候两个都要签。建议先签里面的，再签外面的！



对 `.app` 文件进行签名

```shell
# -f 代表对已经存在的签名进行替换，否则如果已经有了，不会替换！
codesign -f -s 证书ID  xxx.app
```



查看签名

```shell
codesign -vv -d xxx.app
```



检查一下当前的签名是否完美，即未被破坏

```shell
codesign --verify  xxx.app
```



从 `Provisioning Profile` 文件中直接提取 `entitlements`

```shell
# 从embedded.mobileprovision文件中提取出entitlements.plist权限文件
security cms -D -i embedded.mobileprovision > temp.plist
/usr/libexec/PlistBuddy -x -c 'Print :Entitlements' temp.plist > entitlements.plist
```



`.app` 文件主要包含三部分：

- `Mach-O` 格式的二进制可执行文件，这个是一个App最重要的文件，我们编写的`Objective-C`、`Swift`代码都被编译在里面
- 资源文件，包括：`.bundle`文件，`.framework`文件，`.dylib`文件，`.nib`文件，图片文件，音视频文件，字体文件等所有项目用到的文件
- `CodeResources`，签名信息
- `embedded.mobileprovision` 文件，或者 `entitlements` 文件
  - 对于没有上传App Store的`.app`文件，里面会包含`embedded.mobileprovision`文件，没有`entitlements`文件
  - App Store下载的`.app`文件，里面会包含`.entitlements`文件，没有`embedded.mobileprovision`文件



## 附录

[1]. [iOS代码签名:Code Signing](https://blog.csdn.net/skylin19840101/article/details/79966006)

[2]. [iOS 的 Code Signing 体系](https://juejin.cn/post/6844903902605737997)