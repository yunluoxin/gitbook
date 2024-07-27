# MonkeyDev 安装和使用

## 环境

macOS:  14.2.1

Xcode: 15.2



## 安装步骤

最好提前创建一个新文件夹，比如 `Funny`，然后在里面搞事情。

```shell
mkdir Funny && cd Funny
```

1. 安装 `theos` 

   ```shell
   # 先把仓库下载下来
   git clone  --recursive https://github.com/theos/theos
   # 再拷贝到 /opt/theos 目录
   sudo cp -R theos /opt/

2. 安装 `ldid`

   ```shell
   brew install ldid
   ```

3. 安装 `MonkeyDev`

   ```shell
   # 也先把仓库下载下来
   git clone git@github.com:AloneMonkey/MonkeyDev.git
   # 然后用 sudo 执行仓库下的 md-install 进行安装
   sudo sh MonkeyDev/bin/md-install
   ```

   这时候会提示失败：

   `File /Applications/Xcode.app/Contents/PlugIns/IDEiOSSupportCore.ideplugin/Contents/Resources/Embedded-Device.xcspec not found`

   下载网络上别人搞好的一些替换模板：

   ```shell
   # 这里创建了一个新文件夹，是因为大佬这个仓库也叫 MonkeyDev，在和刚才文件夹同级有问题
   mkdir MonkeyDev_Patch && cd MonkeyDev_Patch
   git clone https://github.com/LongMeters/MonkeyDev
   
   # 把 MonkeyDev 文件夹下的所有文件，都复制到 /opt/MonkeyDev/templates/ 下，覆盖原有的文件
   sudo cp -R MonkeyDev/* /opt/MonkeyDev/templates/
   ```

4. 然后就可以在 Xcode 中，选择 MonkeyDevApp 模板创建工程了！

> 现在，你也可以把 Funny 文件夹删除了 ！！！



## 使用

选中当前应用 `Target` 后，在 `Build Settings` 中，可以修改的一些配置<sup>[1]</sup>：

| 配置字段 | 说明 |
| ------ | ------ |
| MONKEYDEV_APP_SUBSTRATE |    -    |
| MONKEYDEV_CLASS_DUMP	 | 头文件dump，会在testMonkeyDev下生成一个header文件，如果自己使用class-dump了就不需要用他的dump了，设置为NO便可，想要使用就设置成YES |
| MONKEYDEV_DEFAULT_BUNDLEID |是否使用默认bundleId，针对一些应用对bundleId的检测。设置成YES就会向外暴露原始 ipa 的 bundleId 。 |
| MONKEYDEV_INSERT_DYLIB |是否注入动态库，这个monkeyDev默认给选了YES就不要动了。因为monkeyDev就是使用注入动态库的方式进行代码注入的 |
| MONKEYDEV_RESTORE_SYMBOL |是否还原符号表，如果使用fishhook之类需要修改符号表的工具就需要还原符号表。 |
| MONKEYDEV_TARGET_APP | 使用原始ipa的方式，Optional 为 ipa可选的 |
| PODS_ROOT | 包后存放的位置 |



## 使用中的问题

1. Unable to install "XXXX"

   设备上已经存在一个与当前bundleId相同的app。有下面两个解决方案：
    - 删除掉手机上的相同bundleID 的app 
    - 把 **`MONKEYDEV_DEFAULT_BUNDLEID`** 修改为 **`NO`**，即：使用自己的在工程中定义的 bundleID 为 app 包名， 而不是使用之前 app 默认的 `bundleID`。

2. 无法安装此App，因为无法验证其完整性。 Failed to verify code signature of xxx

   把 **`MONKEYDEV_DEFAULT_BUNDLEID`** 修改为 **`NO`**

   

## 附录

[1]. [monkeyDev使用及初次使用问题的解决方法](https://blog.csdn.net/dancheng1/article/details/120301439)
