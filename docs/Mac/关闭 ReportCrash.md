# 关闭 ReportCrash 进程

升级到了 Mac OS Monterey - OSX 12.0.1

突然发现 Report 进程一直占用高 CPU，久高不下，杀死强杀后都自动重启！！！

使用下面命令，立竿见影，立马生效，**不用重启**。 重启后，也不复发！

```shell
launchctl unload -w /System/Library/LaunchAgents/com.apple.ReportCrash.plist
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.ReportCrash.Root.plist
```



参考文章：

1. [关闭 ReportCrash 进程防止CPU占用率过高 [OSX] [MacBook]](https://blog.csdn.net/wxqee/article/details/105722190)

