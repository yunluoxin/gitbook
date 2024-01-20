# 怎么安装 python virtualenv

## 安装过程

### 使用 Mac 系统自带的 python3

1. 打开终端, 输入： `sudo pip3 install virtualenv`,  等待安装完毕

2. 开始创建独立执行环境，首先在你想要的位置, 创建一个目录: `mkdir python`

3. 进入此文件夹：`cd python`

4. `virtualenv --no-site-packages venv`, 参数--no-site-packages，已经安装到系统Python环境中的所有第三方包都不会复制过来.  venv既是目录，也是新的虚拟环境名字。

5. `virtualenv -p /your-python-path/python-3.6.2/bin/python3 venv`, 其中 **-p**  可以指定一个，需要复制的python版本的路径

6. OK。至此已经添加了一个系统的Python版本到virtualenv。

> 安装后的virtualenv 以及 python包存放在 `~/Library/Python/3.6.2/lib/python/site-packages` 下



### 使用 homebrew 安装的 python3

终端执行

```shell
pip3 install virtualenv
```

> 安装后的 virtualenv 以及其他的 python 包，都在 `/opt/homebrew/lib/python3.9/site-packages` 文件夹里（python3.9根据你的版本变化）。



### ⭐️⭐️⭐️ 查看包的安装路径 

刚刚知道的一种方式，可以直接通过 `pip` 的命令显示包信息，从而得到定位，这样也不用强记了😁

```shell
	pip3 show frida
```

输出以下内容：

```text
Name: virtualenv
Version: 20.23.0
Summary: Virtual Python Environment builder
Home-page:
Author:
Author-email:
License:
Location: /opt/homebrew/lib/python3.9/site-packages
Requires: filelock, platformdirs, distlib
Required-by:
```



## 使用

1.  `source venv/bin/activate` 命令提示符变了，有个(venv)前缀，表示进入了一个名为venv的Python环境
2.   `deactivate` 退出环境，进入到系统默认的python环境状态.



Ps.  如果执行命令时候，没用 `sudo`, 即 `pip3 install virtualenv`，则需要手动去配置环境变量！

```shell
vi ~/.zshrc
# 把下面几行填入到文件里👇🏻

# Python 用户级
PYTHON_USER="$HOME/Library/Python/3.9/bin"
export PATH="$PATH:$PYTHON_USER"
```

整个 Python 安装周期执行一次这个即可！之后所有安装的非 `sudo` 的都会在这里，也会被检索到命令！
