# 怎么安装python virtual env
## 安装过程
1. 打开终端, 输入： `sudo pip3 install virtualenv`,  等待安装完毕

2. 开始创建独立执行环境，首先在你想要的位置, 创建一个目录: `mkdir python`

3. 进入此文件夹：`cd python`

4. `virtualenv --no-site-packages venv`, 参数--no-site-packages，已经安装到系统Python环境中的所有第三方包都不会复制过来.  venv既是目录，也是新的虚拟环境名字。

5. `virtualenv -p /home/username/opt/python-3.6.2/bin/python3 venv`, ** -p **  可以指定一个，需要复制的python版本的路径

6. OK. 至此已经添加了一个系统的Python版本到virtualenv. 

## 使用
1.  `source venv/bin/activate` 命令提示符变了，有个(venv)前缀，表示进入了一个名为venv的Python环境
2.   `deactivate` 退出环境，进入到系统默认的python环境状态.
