# Swift和OC混编的Pod库

假设接下来你要创建的库或者组件叫做`PodName`，开始我们的旅程：

通过  `pod lib create PodName` 创建的工程，设置的过程中默认为 Swift 工程！则pod会自动创建swift调用oc的桥接文件 `PodName-umbrella.h` 

1. 在Pod库中，相互调用

   Swift 调用 OC： 直接调用！已经有了桥接文件。

   OC 调用 Swift：在OC中 `#import <PodName/PodName-Swift.h>` ，则就可以使用了。记得swift中，相应方法和变量等标记 `@objc`。

   

2. 在项目中（这里我是Swift项目），调用Pod库（Swift库）

   可以通过在项目中，随便创建OC文件，则Xcode会提示，生成我们需要的 `BridgingHeader` 文件。

   Swift 调用库中 OC：直接调用！

   OC 调用库中 Swift：在OC中 `#import <PodName-Swift.h>` ，则就可以使用了。记得swift中，相应方法和变量等标记 `@objc`。

   OC 调用库中 OC：`#import <PodName/PodName-umbrella.h>` 就可以了

> 参考链接：https://www.jianshu.com/p/2337ad91094d

3. Pod库中，调用它依赖的pod库（OC或者C++)，比如叫 **XXX**

   ​	a. Swift  	

   ​		- 对于 C++ 库， 目前发现无法直接调用，要通过 oc 包一层，然后 swfit 直接调用 oc

   ​		- 对于 OC 的库，可以直接 `import XXX`

   ​	b. OC通过下面方式调用，优先使用

   ```c
   // 下面两种方式都可以
   //#import "SomeHeader.h"
   #import <XXX/SomeHeader.h>
   ```

4. 主项目直接使用

   和3一样！