# Swift Package CLI 的使用

#Swift Package 制作脚本#

假设要制作的工具脚本叫做 `Tools`, 那么先创建 `Tools` 文件夹，然后进行这个文件夹， 执行 Swift Package 初始化命令：

```shell
mkdir Tools
cd Tools
swift package init --type executable

# 控制台输出如下：
Creating executable package: Tools
Creating Package.swift
Creating README.md
Creating .gitignore
Creating Sources/
Creating Sources/Tools/Tools.swift
Creating Tests/
Creating Tests/ToolsTests/
Creating Tests/ToolsTests/ToolsTests.swift
```



此时，得到的文件夹结构如下：

```shell
east:Tools/ $ tree .                                                  [10:22:35]
.
├── Package.swift
├── README.md
├── Sources
│   └── Tools
│       └── Tools.swift
└── Tests
    └── ToolsTests
        └── ToolsTests.swift

4 directories, 4 files
```

其中 ：

- `Package.swift` 是最重要的配置文件，类似于 Pod 的 `Podfile` 文件，用来配置各种依赖包。以及另外一个功能：定义 `target`（原本在 `project` 文件中定义的）
- `Sources` 文件夹下是放我们代码的地方
- `Tests` 单元测试文件



## 如何进行开发？

有两种方式：

1. 直接双击 `Package.swift`，系统会直接调用 *Xcode* 打开！

2. 使用 XcodeProj 文件进行开发。  在根目录 `Tools` 下，执行：

   ```shell
   swift package generate-xcodeproj
   ```
   
   即可生成一个 `Tools.xcodeproj` 文件，你就可以把它当做普通工程进行开发！

   > 实测发现 有点难用，可能是我不会用。我直接使用



## Package.swift

`Package.swift` 初始文件内容（省略了默认的英文注释）如下：

```swift
// swift-tools-version: 5.7
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "Tools",
    dependencies: [
      	// 如果你的工程依赖了第三方的包，都要在这里列出！ 这里是包总配置的地方
    ],
    targets: [ 
      // targets 下定义了所有的 target，这里默认生成了两个 target： 一个是我们想要的可执行文件，一个是单测试target
      // 一个 target 可以依赖另外一个 target. 当前就是 testTarget 依赖了 executableTarget.
        .executableTarget(
            name: "Tools",
            dependencies: []),
        .testTarget(
            name: "ToolsTests",
            dependencies: ["Tools"]),
    ]
)
```



假设我们写脚本时候，发现一些代码，其实可以抽出来到另一个库，后面还可以复用。那要怎么做呢 ？

大概两步骤：

1. 做一个 `framework` （这里取名为 `ToolsCore`）
2. 被当前的工具脚本引用 （`import ToolsCore`）

在 SPM 中要这么做：

1. 在 `Sources` 文件夹下（即与 `Tools` 同级），创建一个名为 `ToolsCore` 的文件夹 （库名必须和文件夹同名！！！）

2. 在 `Package.swift` 的 `targets` 下新增一个 `target`

   ```swift
       targets: [ 
           // 新建target:
   				.target(
           	name: "ToolsCore",	// 必须和文件夹同名
             dependencies: []),
           .executableTarget(
               name: "Tools",
               dependencies: []),
           .testTarget(
               name: "ToolsTests",
               dependencies: ["Tools"]),
       ]
   ```

3. 记得在 `Sources/ToolsCore` 文件夹下写的内容，如果要给 `Tools` 用，都要加上 *public*，它就是一个库，要用开发库的方式写代码！

4. 在 `Tools` 代码中要调用的地方， `import ToolsCore`



## 资源的引用

在 SPM 中，如果你要携带资源，不能都直接拖到工程中就解决了的。视情况而定<sup>[1]</sup>：
- 对于一些使用目的明确的文件类型，如下图，不需要做啥，拖到对应位置即可！

  <img src="Images/swift_package_clear_purpuse.jpg" alt="swift_package_clear_purpuse" style="zoom:50%;" />

- 其他类型的文件，如下图，都需要配置。类似 Pod 一样，进行一些资源配置：

  <img src="Images/swift_package_ambiguous_purpose.jpg" alt="swift_package_ambiguous_purpose" style="zoom:50%;" />

   1. 复制你的资源到你的库下。 哪个 `target` 需要的资源就复制到哪个 `target` 下！ 

      假设，当前 `ToolsCore` 需要一个文件夹`ABC`的资源，则把文件夹ABC 复制到 `Sources/ToolsCore/`下！ 

   2. 在 `Package.swift` 中进行配置

      在 `ToolsCore` 的 `target` 里，加入 `resources` 属性
  
     ```swift
         targets: [
             .target(
                 name: "ToolsCore",
                 dependencies: [],
                 exclude: [  		// 要排除的文件或者文件夹
                     "A"
                 ],
                 resources: [    // 资源属性是个数组，可以添加n个资源
                     .copy("ABC")	// 这个是相对于 当前你这个target所在的目录(Sources/ToolsCore) 为 根目录！ 
                 ]
             ),
             .executableTarget(
                 name: "Tools",
                 dependencies: []
             ),
             .testTarget(
                 name: "ToolsTests",
                 dependencies: ["Tools"]),
         ]
     ```
  
  ​		每一个资源，你可以分别使用 `.copy()`、`.process()`。如果不需要经过处理，想原样的搬运到库里，就用 `.copy()`。对于图片之类，可以考虑使用 `.process()`，苹果可能进行优化处理。
  
   3. 配置了加入资源后，系统会自动对 `Bundle` 类扩展一个 `module` 的 *static* 属性，在库里要使用资源，可以直接用 `Bundle.module.path(forXXX:)`！而不用自己去手动拼接啥的（在 *Pod* 里常这么干！！！）

> 路径的根目录问题，已经在上面注释了。



## 开发

### 程序的入口

脚本被执行时，标注为 `@main` 的 struct 或者 其他 中的 `static main` 方法会被调用！这里是整个程序的入口。

```swift
@main
public struct Tools {
    public private(set) var text = "Hello, World!"

    public static func main() {
        print(Tools().text)
    }
}
```

之后，就和正常的开发一样了。唯一的差别，可能就是这里没有 App 那种 `UI` 界面，也没用 `RunLoop`（主线程代码移走完，脚本就结束啦！）。



### 编译 - 生成最终的产物

在根目录下执行 

```swift
swift build
```

即可生成最终的二进制文件！ 如果有错误，这里也会提示。



生成的可执行文件在 根目录下 的 `.build/debug` 下， `.build/debug/Tools` 这个文件，就是你的工具啦。

> 注意： 这个产物仅是你的电脑架构的版本。
>
> 比如你用m1，就只生成 arm64 的：
>
> ```shell
> east:debug/ (main✗) $ file Tools                                      [13:47:05]
> Tools: Mach-O 64-bit executable arm64
> ```
>
> 
>
> 另外，`debug` 文件夹其实是个软链接，链接到的是当前运行的架构的专门的文件夹。 所以用 M1 生成的文件其实在 `.build/arm64-apple-macosx/debug` 下！
>
> ```shell
> east:.build/ (main✗) $ ls -al                                         [13:35:30]
> total 48
> drwxr-xr-x   9 east  staff    288 11  8 19:04 .
> drwxr-xr-x  14 east  staff    448 11  8 19:03 ..
> drwxr-xr-x   5 east  staff    160 11  8 19:04 arm64-apple-macosx
> drwxr-xr-x   2 east  staff     64 11  8 15:23 artifacts
> drwxr-xr-x   2 east  staff     64 11  8 15:23 checkouts
> lrwxr-xr-x   1 east  staff     24 11  8 19:04 debug -> arm64-apple-macosx/debug
> -rw-r--r--   1 east  staff  16624 11  8 17:44 debug.yaml
> drwxr-xr-x   2 east  staff     64 11  8 15:23 repositories
> -rw-r--r--   1 east  staff     97 11  8 19:33 workspace-state.json
> ```



#### *生成多架构的版本<sup>[3]</sup>

条件：需要 `Swift 5.3+`、`Xcode 12.0 beta+` 

```shell
swift build -c release --arch arm64 --arch x86_64
```

生成后的产物，可以在 `.build/apple/Products/Release` 下看到。此时的 `Tools` 文件，是一个 fat 二进制文件：

```shell
east:Release/ (main✗) $ lipo -info Tools                              [13:55:37]
Architectures in the fat file: Tools are: x86_64 arm64
```





### 运行 - 把你的脚本 Run 起来

#### 运行方式

有两种方式：

		1. 在根目录下直接执行 `swift run` 即可自动编译+运行！ （目前我不知道怎么加参数，以后懂了补上。）
		1. 找到可执行文件。 通过双击或者在终端中执行，可以添加运行参数。



#### 运行参数

脚本运行时候，一般都会附加一些参数，可以通过下方代码获取到：

```swift
let args = CommandLine.arguments
print(args)
```

*args* 是一个字符串数组。第一个元素是当前运行的可执行文件的地址，其他元素是运行时候传递进来的（通过空格分割后的）



如执行了 `./Tools xxx.plist -test`，则 args = [ "./Tools", "xxx.plist", "-test"]



如果你要弄成和很多脚本一样，可以有help，报错优雅提示，乱序输入参数什么的。可以用官方的 `swift-arguments-parser`<sup>[2]</sup> ! 具体使用教程见附录.



## 附录

[1]. [Swift Package Manager 添加资源文件](https://juejin.cn/post/6854573220784242702)

[2]. [苹果官方的 arguments-parser](https://github.com/apple/swift-argument-parser)

[3]. [Building Swift Packages as a Universal Binary](https://liamnichols.eu/2020/08/01/building-swift-packages-as-a-universal-binary.html)