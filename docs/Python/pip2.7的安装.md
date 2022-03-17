# Mac下Python pip2的安装
首先，我之前以为pip对于python2.7和3.8是有不同版本的，但是[https://www.runoob.com/w3cnote/python-pip-install-usage.html]介绍，用什么版本安装pip，pip就会被关联到哪个版本，所以到底是不是这个安装脚本里做了判断，安装了不同版本，还是python两个版本其实可以安装一样的！未知！

## 到底需不需要安装？
发现装了Xcode之后，会让系统自动拥有python3.8. 而Mac系统内置的是python2.7. 
我的系统当前命令中只有**pip3**, 没有pip不知道怎么回事。 如果你已经有了pip就不需要了！

## 过程
1. 尝试了用`sudo easy_install pip`安装失败了，过程中出现脚本运行出错，应该是脚本中用了python 3的特有语法！
如果安装失败了，可以执行下面命令， 把垃圾清理干净：
```
rm /usr/loca/bin/pip
rm /usr/loca/bin/pip2
rm /usr/loca/bin/pip2.7

// 删除pip安装目录site-packages下部分文件和目录, 路径是：
/Library/Python/2.7/site-packages
自己分辨删除
```

2. 
```
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py   # 下载安装脚本
sudo python get-pip.py    # 运行安装脚本
```
这里提下，如果是要安装pip3可以用 
```
sudo python3 get-pip.py    # 运行安装脚本
```
结果，还是报错了。原因也是2.7中没有format string!  网上搜索了下，用更老版本的安装脚本可以：
https://bootstrap.pypa.io/2.7/get-pip.py

完整命令是
```
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py -o get-pip.py   # 下载安装脚本
sudo python get-pip.py    # 运行安装脚本
```

3. 这次用了旧的get-pip.py脚本，搞定了！ 安装好的pip在刚才提到的site-package下，多了个pip目录！

4.  pip3这里没有尝试，应该直接用最新的get-pip.py是可以的！

>  pip安装的package最终发现在 `~/Library/Python/2.7/lib/python/site-packages`下
