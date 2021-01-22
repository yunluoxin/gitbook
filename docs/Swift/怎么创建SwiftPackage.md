# 怎么创建SwiftPackage
1. 打开终端, 先创建一个文件夹： `mkdir WWDC`

2. 然后进入文件夹： `cd WWDC`

3. 执行 `swift package init [--type executable]` <br>
    其中, --type是可选参数, 可以有四种类型：
    - library：创建库
    - executable：创建可执行文件
    - empty：创建空项目
    - system module：创建系统模块项目
    默认创建的是 **library**
    支持，在WWDC文件夹下，就自动生成了所有package需要的文件
    
4. 为了能更方便的调试，我们需要创建一个demo工程，引用这个package！
    `swift package generate-xcodeproj`
    
1. 至此，OK啦。 在当前的文件夹下，找到**WWDC.xcodeproj**，打开即可！