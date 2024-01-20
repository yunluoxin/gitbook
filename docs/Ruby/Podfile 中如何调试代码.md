# Podfile 进行 Debug 的方法

很久前学过，不知道是不是这个方法，但是给忘记了，这次在 [开启 Cocoapods 新选项，加快项目索引速度](https://kemchenj.github.io/2019-05-31/) 重新看到，给记下来！

1. 安装 debugger： 

   ```shell
   gem install pry
   ```

2. 在 Podfile 文件的开头，导入 `require 'pry'`

3. 在Podfile文件中，我们想要插入断点的地方，写上 `binding.pry` 即可！