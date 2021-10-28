# 探索Pod的资源引用

## 引用的不同方式

```ruby
  s.resource = 'SwiftBasic/Assets/**/*'
  # 这种方式直接就是纯文件拷贝！文件会被拷贝到根目录下。 如果你放置的是一个 bundle，就是整个 bundle 被拷贝过去。
  # |- info.plist
 	# |- execuable二进制文件
 	# |- 人工创建的 bundle
  # |- ...

  s.resource_bundles = {
    'SwiftBasic' => ['SwiftBasic/Assets/**/*']
  }
	# 这种方式会自动帮你打包成一个以 key 为名的 bundle，比如这里的是 SwiftBasic.bundle，里面放置的是你的 Assets 目录下的文件。如果里面包含人工创建的 bundle 文件，也会复制过去，到这个 SwiftBasic.bundle 下！
  # |- info.plist
 	# |- execuable二进制文件
 	# |- SwiftBasic.bundle
 	# 			|- 人工创建的 bundle
  # |- ...
```

如果你直接使用：

```ruby
	s.resource = 'SwiftBasic/Assets/**/*.png'
  # 或者
	s.resource_bundles = {
    'SwiftBasic' => ['SwiftBasic/Assets/**/*.png']
  }
	# 主要是通配符上的区别！
	# 结果：
	# 	最终会略过包含 png 的文件夹，直接把各级的 png 文件抽出到 bundle 根目录（resource 是主 bundle，resource_bundles 是 SwiftBasic.bundle）下
```



## 疑问带来的新发现

看到网上引用的都是 `Bundle(for: class)` 方法来获取 Bundle，继而获取 `resourcePath`，为什么不直接用 `Bundle.main`？

我用一个在 framework 中的类，获取了下，发现确实和 `Bundle.main` 获取到的一致！那为什么多此一举？

```swift
  if let mainBundle = Bundle.main.resourcePath {
      print(mainBundle)
//   	/var/containers/Bundle/Application/0A62A247-F419-4406-B9B2-3B28C583C637/Test.app
  }

  /** Position 类在静态库中 */
  if let bundle = Bundle(for: Position.self).resourcePath {
      print(bundle)
//    /var/containers/Bundle/Application/0A62A247-F419-4406-B9B2-3B28C583C637/Test.app
  }
```

突然想到难道动态库不一样？我用另外一个动态库尝试了下，果然：

```swift
  /** ScreenUtil 类在动态库中 */
  if let bundle = Bundle(for: ScreenUtil.self).resourcePath {
      print(bundle)
//    /var/containers/Bundle/Application/0A62A247-F419-4406-B9B2-3B28C583C637/Test.app/Frameworks/SwiftBasic.framework
  }
```

动态库的时候，直接获取到的是动态库在 `Bundle.main` 即主 bundle 中的地址！



## 总结

上面两个主题是不冲突的哦！

**疑问带来的新发现** 决定了你引用的文件的存放**根目录**，

**引用的不同方式** 决定了 **根目录** 下文件的 **不同组织方式**。

为了不用区分根目录在哪里，直接使用 `Bundle(for: class)` 来获取根目录，然后拼接上你的文件名即可！如果你在根目录下还放置了另外一个自定义 Bundle，把拼接后的路径去创建 Bundle 对象即可！