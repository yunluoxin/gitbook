# Cocoapods 相关

## 普通工程

#### 已有的 project 工程，如何添加 pod 支持？

在 `项目.xcodeproject` 同级目录下，新建一个 `Podfile` 为名的文件：

```shell
vi Podfile
```

输入如下文本内容（`GPUImage` 替换成自己要导入的库）：

```text
platform :ios, '10.0'
target 'SwiftOpenGLES' do
	pod 'GPUImage'
end
```

退出 vim 文本编辑功能： 按下 `ESC` 键， 然后键入 `:wq` 即可。

>  注意：
>
> 如果你当前用的是 Xcode 最新的beta版本，可能会执行失败！
>
> 当前市面上最新发型版本为 Xcode 12+，我用 Xcode 13 beta 执行 pod install 一直失败，老怀疑自己文本内容写错了，最后阅读错误提示，猜测是这个原因，用了旧的版本直接成功了！



## 创建 Pod 库

#### 如何直接创建 Pod 库工程？

```shell
pod lib create Your-component-name
```

执行命令后，根据提示填入几个选项，即可创建并自动打开Pod工程



#### 检查 Pod 工程、podspec 文件是否合法

```shell
pod lib lint Your-podspec-file
```



#### 添加依赖库

有时候你的 Pod 库，还要依赖其他的库。比如你可能在自己的库中依旧使用 `AFNetworking`，这时候，可以通过以下方式：

- 直接把 `AFNetworking` 相关文件，直接添加复制到工程。这种方式**不推荐**，调用你的库的项目，很大可能也用了 `AFNetworking`，会导致编译失败！

- 把打包好的 `AFNetworking` 库拖入 Pod 库中， 然后在 `.podspec` 文件写：

  ```ruby
  s.vendored_frameworks = 'XXX/Classes/*.framework'    # 指定framework
  s.vendored_libraries = 'XXX/Classes/*.a'    				 # 指定 .a
  ```

  这种情况，和上面那种类似，也要注意主项目也引入同个库的问题！

- `.podspec` 文件中写上

  ```ruby
  s.dependency = 'AFNetworking' [, '~> Tag版本号']		# [ ] 中内容为可选的. 如果不写，默认读取最新的代码
  ```

  这样就相当于直接在该 Pod 库中，pod 导入其他库！



#### 添加资源文件

有两种方式<sup>[1]</sup>：

1. **resource_bundles**

   `resource_bundles` 允许定义当前 Pod 库的资源包的**名称和文件**。用 hash 的形式来声明，key 是 bundle 的名称，value 是需要包括的文件的通配 patterns。

   同时建议 bundle 的名称至少应该包括 Pod 库的名称，可以尽量减少同名冲突！

   Example:

   ```ruby
   spec.resource_bundles = {
       'MapBox' => ['MapView/Map/Resources/*.png'],
       'OtherResources' => ['MapView/Map/OtherResources/*.png']
   }
   ```

2. **resources**

   使用 `resources` 来指定资源，被指定的资源只会简单的被 copy 到目标工程中（主工程）。

   ```ruby
   spec.resource = 'Resources/HockeySDK.bundle'	# 这里的 bundle 会被视为一个普通的文件，被复制
   spec.resources = ['Images/*.png', 'Sounds/*']
   ```

CocoaPods 官方强烈推荐使用 `resource_bundles`，因为用 key-value 可以避免相同名称资源的名称冲突。

> 注意：
>
> 通过 `resource_bundles` 引入资源的方式，在调用端需要用特殊代码，具体可以看<sup>[1]</sup>。

更加具体的内容，请查看我的另外一篇文章：[探索Pod的资源引用](./探索Pod的资源引用.md)



#### 直接打包、导出库

打开终端，切换到 `Your-podspec-file.podspec` 文件的目录下，输入命令：

```shell
pod package lib名字.podspec
# 注意：别漏了 lib, 固定 lib 开头的！
# 后面是可以加参数的：
# --force：强制覆盖之前存在的文件
# --library：打包成.a文件
# --dynamic：打包成动态库
# --configuration：配置选项，默认为Release
```

就可以开始打包了，等待打包完成。



## 私有 spec 仓库

#### 如何创建私有仓库

1. 在 gitlab 或者 github 上，创建一个新工程，名字为 **MySpecs**（随便取，建议带有 `Specs`）。记得勾选 **Readme， 直接初始化**仓库

2. 复制刚刚创建的 git 仓库的地址

3. 打开终端，直接输入：

   ```shell
   # pod repo add [Private Repo Name] [GitHub HTTPS clone URL]
   pod repo add MySpecs git仓库的地址
   ```

4. 在终端执行 `cd ~/.cocoapods/repos`，然后`ls`，如果可以看到一个名为 **MySpecs** 的目录，就是成功了！



#### 如何获取仓库名字？

```shell
cd ~/.cocoapods/repos
ls
```

列出你现在所有的仓库列表，复制你要的仓库的仓库名即可！



#### 推送 podspec 文件到仓库

```shell
pod repo push Your-repo-name Your-podspec-file [--allow-warnings]
```



## 缓存问题

如果你是通过指定 Tag 导入某个库的，则如果远端把这个 Tag 指向了其他的 commit，本地死活会拉取不到最新的！为什么呢？

因为 cocopods 是有缓存的！我们下载后，本地就有了一份代码。这样有其他工程，需要导入同样版本的库时候，会直接使用而不用去网络拉取！这也导致我们，不管是 `pod install` 还是 `pod update 某个库` ，又或者是先移除库，再重新导入， 都无效！删除 Podfile.lock 以及 Manifest.lock 里对应库的 commit 那一行，也是无效的！

正确做法是：

- 删除缓存目录~/Library/Caches/Cocoapods里对应库的源码（整个文件夹）
- 删除 Podfile.lock 以及 Manifest.lock 里对应库的 commit 那一行
- 重新 `pod install --repo-update` 或者 `pod update 那个库`



参考文章：

[1]. [关于 Pod 库的资源引用 resource_bundles or resources](https://juejin.cn/post/6844903559931117581)
