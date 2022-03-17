# 推送到 Cocoapods 公有仓库

## 注册

首先需要用邮箱注册下，用户名和邮箱可写可不写：

```shell
pod trunk register 你的邮箱📮 '用户名' --description='描述。比如，可以写你电脑的名字'
```

提交后，会收到一封邮件，查收并点击里面的链接，即可激活！

## 查看自己的信息

会显示当前 session 的所有信息：

```shell
pod trunk me
```

## 换电脑登录

换电脑后需要重新登录的，和注册类似，也需要邮箱验证：

```shell
pod trunk register 你的邮箱📮
```

点击邮箱链接后，即可在此台电脑上建立 session

## 推送

把我们的 podspec 推送、提交到 github 上 Cocoapods 公有仓库中：

```shell
pod trunk push xxxx.podspec
```

