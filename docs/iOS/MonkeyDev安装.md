# MonkeyDev 安装

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



## 使用中的问题

1. 编译成功，安装和运行失败

   把 **`MONKEYDEV_DEFAULT_BUNDLEID`** 修改为 **`NO`**，即：使用自己的在工程中定义的 bundleID 为 app 包名， 而不是使用之前 app 默认的 `bundleID`。

