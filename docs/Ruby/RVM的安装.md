# RVM 的安装

## 简介

​	**RVM** 是 Ruby Version Manager 的简称，即 Ruby 版本管理器。



## Why not system ruby ? 

​	为什么要用 RVM？苹果已经有内置的 ruby 了，有什么必要安装其他版本呢？

​	根据参考文章<sup>[1]</sup>的介绍，MacOS 内置的 ruby 是给自己系统用的，一般是比较低的版本（比如我写此文章时候是2.6）。而凡是我们执行 gem 命令，需要用到 sudo 的，比如 `sudo gem install ffi` 都代表着要对系统的 ruby 进行改变！

​	所以，最好的方式是自己安装另外一个 ruby，有几个优势：

- 安装时候不需要用到 sudo
- 不会改变系统 ruby
- 可以安装最新稳定版本的 ruby



## 安装过程

1. 由于需要安装 mpapis public key（别问我是啥，哈哈😄），所以需要 gpg。但是 MacOS 原生不支持 gpg，所以我们需要先安装 gpg！而安装 gpg，我们需要 Homebrew，所以没安装 homebrew 的，需要先去安装<sup>[4]</sup>哦，本文不涉及此！

   ```shell
   # 安装 gpg
   brew install gnupg
   # 安装 mpapis public key
   gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
   ```

   > 注意： gpg 的 keys 会发生变化！如果发现上面命令出现问题，请打开 [RVM 官网安装教程](https://rvm.io/rvm/install) 复制最新 keys，替换上面的命令的 **keys 部分**！ （只要替换 keys， 其他部分不要哦~  即只要替换 **409B6B....** 到最后）

2. 安装 rvm

   ```shell
   # 安装 rvm 最新的稳定版本
   \curl -sSL https://get.rvm.io | bash -s stable --ruby
   ```

3. 关闭、重开终端，使得最新的环境改变生效

4. 终端中执行 `rvm list`, 如果你看到了类似下面的 ruby 版本列表，即代表 rvm 已经安装成功啦！💐恭喜💐

   ```shell
   =* ruby-3.0.0 [ arm64 ]
   
   # => - current
   # =* - current && default
   #  * - default
   ```



## 使用

列出所有已安装的 ruby 版本

```shell
rvm list
```

通过 `rvm use` 命令，你可以选择一个生效的 Ruby 版本

```shell
rvm use 3.0.0
```

上面的命令只是在当前控制台生效，如果你需要之后新打开的所有控制台都生效，你需要使用 default：

```shell
rvm use 3.0.0 --default
```

通过 `ruby -v`，可以知道当前正在使用的 ruby 版本，来验证是否切换版本成功。

```shell
ruby -v
# ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [arm64-darwin21]
```



### 安装更多版本的 ruby

列出已知的 Ruby 版本

```shell
# 得到你可以安装的所有 ruby 版本列表
rvm list known 
```

安装一个 Ruby 版本

```shell
rvm install 2.7 --disable-binary
```



### 关于安装其他 Gem

在你指定了默认的 ruby 版本后，rvm 直接变更了 shell 环境，所以安装 gem 什么的，就和平时一样的！**不用加**什么 rvm 前缀!

```shell
gem install json
```

>  过程中不应该用到 sudo！



## 卸载某个版本 ruby

注意，这里的卸载竟然没和安装的 **install** 对应的 uninstall，用 **remove** !

```shell
rvm remove 3.0.0
```



## 安装插件太慢？

可以尝试更换源（未实验过，校验过网址是有效的）：

```shell
echo "ruby_url=https://cache.ruby-china.com/pub/ruby" > ~/.rvm/user/db
```



## 参考文章

1. [Stack overflow](https://stackoverflow.com/questions/69460048/unable-to-install-cocoapods-in-macos-monterey-version-12-0-beta-xcode-13-013a)
2. [How to Install Ruby on macOS with RVM](https://jeffreymorgan.io/articles/ruby-on-macos-with-rvm/)
3. [RVM 官网安装教程](https://rvm.io/rvm/install)
4. [Ruby China WIKI](https://ruby-china.org/wiki/rvm-guide)
5. [ARM64 版本 Mac 的 Homebrew 安装教程](https://zhuanlan.zhihu.com/p/341831809)