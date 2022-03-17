# Python执行Shell的几种方式

学习自 [python执行shell脚本的几种方法](https://blog.csdn.net/qq_27825451/article/details/102909772)

## Python 系统库里自带的命令

### os.system

这个命令的返回值，为0表示命令执行成功。

**注意：这个方式无法得到命令执行的结果！**

```python
import os
os.system('ls -al')
```



### os.popen

这个命令可以得到执行的结果。`os.popen()` 返回的是 file read 的对象，对其进行读取 `read()` 的操作可以看到执行的输出。

**注意：os.popen() 返回的是一个文件对象哦！！！**

```python
import os
f = os.popen('ls -al')
content = f.read()
print(content)
```

没有返回指的shell命令，也可以使用popen()方法，不过 `read` 出来的是空的。



## subprocess

​	subprocess模块是python从2.4版本开始引入的模块，也是系统自带的，不需要再额外安装了。主要用来取代 一些旧的模块方法，如os.system、os.spawn*、os.popen*、commands.*等。subprocess通过子进程来执行外部指令，并通过input/output/error管道，获取子进程的执行的返回信息。

#### subprocess.call()

​	执行命令，并返回执行状态，其中shell参数为False时，命令以及命令的参数需要通过列表的方式传入，当shell为True时，可通过一个字符串直接传入命令以及命令所需要的参数

```python
import subprocess

# shell参数为false，则，命令以及参数以列表的形式给出
print(subprocess.call(["ls","-l"],shell=False)) 

# shell参数为true，则，命令以及参数以字符串的形式给出
print(subprocess.call("ls -l",shell=True))  		 
```



#### subprocess.check_call()

​	用法与subprocess.call()类似，区别是，当返回值不为0时，直接抛出异常，这里不再赘述了。

#### subprocess.check_output()

​	用法与上面两个方法类似，区别是，如果当返回值为0时，直接返回输出结果，如果返回值不为0，直接抛出异常。需要说明的是，该方法在 python3.x 中才有。



## 通过外部的模块

### sh

首先，通过 `pip install sh` 安装模块！

调用方式为: 

```python
sh.command_name("参数一", "参数二", "参数三")
```

如：

```python
import sh

sh.pwd()
sh.mkdir( new_folder )
sh.touch( new_file.txt )
sh.whoami()
sh.echo( This is great! )

sh.ls("-l")         # 等价于 ls -l
print(sh.ls("-l"))
 
sh.df("-h")         # 等价于 df -h
print(sh.df("-h"))
 
# 当有多个参数的情况，且参数可以赋值
sh.du("-h","-d 1")  # 等价于 du -h -d 1
print(sh.du("-h","-d 1"))
```
