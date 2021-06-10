# PyInstaller 的使用

1. 将程序打包成单一执行文件（适合比较简单的代码或者只有一个.py文件）

```Pyth
pyinstaller -F your-python-file.py
```

执行上面的命令后，你会发现：

	- 在路径下产生了同名的`your-python-file.spec`文件（这个是用来对打包过程、内容进行设置的)
	- build文件夹，log记录文件相关文件会放在这里
	- dist文件夹。 最终的可执行文件就在这里！ 如果是Mac， 是二进制可执行文件`your-python-file`，没有后缀,  而windows是一个`your-python-file.exe`文件

一般情况下，我们用上述命令就能搞定了！



2. 打包多个文件，exe文件以及依赖的东西会一起放到dist资料文件夹内（适合框架形式的程序）

```python
pyinstaller -D your-python-file.spec
```

其中`your-python-file.spec`就是上一步骤的产物



> 注意： 导入其他python库时候，最好用`from xx import yy`方式，这样不会把整个库都打包，导致打包出来的文件很大。

