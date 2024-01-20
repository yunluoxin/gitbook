# LLDB 使用

`ASLR (Adress Space Layout Randomration)` : 地址空间布局随机化

*代码的地址（函数的内存地址） = ASLR偏移 + 代码在文件中的偏移量*

### 获取 ASLR 

```lldb
# 获取所有的动态库的偏移和文件地址等
image list

image list -f -o
#todo: 和直接的 image list 差别是什么？
```



## 帮助

lldb 的命令非常多，不可能都列举。当忘记命令时候，可以通过 `apropos` 命令帮你：

```lldb
apropos breakpoint		# 可以显示出 breakpoint 所有可用的命令
apropos watchpoint 
```



## 断点

### 查看断点

- `breakpoint list`

- `br list`

- `b`



### 设置断点可以用

- `breakpoint set xxxx`
- `br s xxxx`
- `b xxxx` 



### 通过符号断点（OC用的，Swift不知道）
- `b -[SKPaymentQueue addPayment:]` 

  > 记住 **:** 号不能省略
  >
  > 😁 断点这个后，可以通过 `po [((SKPayment *)$x2) productIdentifier]` 得到当前正在付款的SKU



### 通过地址断点

```lldb
br s -a 代码的地址
# 或者
br s -a ASLR偏移+文件偏移量
# 或者
br s --address 代码的地址
# 。。。。更多方式
```



### 启用断点

```lldb
br enable 1
# 其中1是br list 显示中可以看到的，断点序号
```



### 禁用断点

```lldb
br disable 1
```



### 删除断点

```lldb
br delete 1		# 删除断点1
br delete 		# 删除所有
```



### ⏸ 暂停（中断）程序

程序运行中，若想继续设置断点等，可在 lldb 命令中直接使用 `Control + C`，即可中断程序执行。或者执行这个命令：

```lldb
process interrupt
```



## 调试

继续运行

```lldb
c
# 或者
continue
```

步进

```lldb
s
# 或者
step
```

下一步

```lldb
n
# 或者
next
```

跳出子程序

```lldb
finish
```

直接返回（适用于忽略后面的代码，提前返回。动态调试时候，可以绕过检测函数）

```lldb
thread return
```



## 寄存器

LLDB中操作寄存器的命令是 `register`

### 读取寄存器

```lldb
register read 			# 输出当前所有的寄存器信息
register read $x0		# 读取x0寄存器
```

将 x1 寄存器的 C字符串 打印出来：

```lldb
x/s $x1
```

如果寄存器里的值是对象，还可以用 `po (print object)` 命令

```lldb
po $x0
po [$x0 config]	# 假设 x0 对象有 config 属性
```

`p (print)` 命令直接输出原生类型的信息，可以指定输出的格式

```lldb
p/x $x2 	# 输出16进制的
p/d $x2 	# 输出整数
```



### 写寄存器

```lldb
register write $x0 1	# 对寄存器x0写入值1
```



## 内存操作

LLDB中操作内存的命令是 `memory`，主要分为读取、保存与修改。

### 读取内存

使用 `read` 参数读取指定内存，支持寄存器地址和内存地址读取，`-c` 参数指定要读取的字节

```lldb
memory read $x0									# 读取寄存器
memory read -c 100 0xabcdef			# 按个数读取
memory read 0xabcdef 0xfedcba		# 按区域读取
```

如果被读取的数据超过1024字节，就会出现错误，加入--force参数即可解决：

```lldb
memory read -c 1025 --force 0xabcdef
```



### 写入内存

```lldb
memory write 0xabcdef 0x24	# 对0xabcdef的内存写入值
```

> #todo: 看书里写的，这里好像只会写入一个字节？



### 保存文件

```lldb
memory read -c 300 0xabcdef --outfile /path/to/txt
```

如果只需要保存原始字节，加入--binary参数即可：

```lldb
memory read -c 300 0xabcdef --binary --outfile /path/to/txt
```

> #todo: 什么是原始字节？



## 内存监视

`watchpoint` 用来监视某个内存的变化，可以理解为 **内存断点**。比如在调试游戏数据时，想知道生命值什么时候被修改就可以使用它。它包括访问（read）、写入（write）及读写（read_write）三种类型

### 内存访问

```lldb
watchpoint set expression -w read -- 0xabcdef
```

### 内存写入

```lldb
watchpoint set expression -w write -- 0xabcdef
```


### 内存读写

```lldb
watchpoint set expression -w read_write -- 0xabcdef
```

### 条件监视

```lldb
watchpoint modify -c '*(char *)0xabcdef == 20'
```

### 其他
像 list/delete/enable/disable 的，都和 breakpoint 一样， eg: `watchpoint list`



## 反汇编
`disassemble`（可以简写为 “`dis`”）命令可以反汇编指定地址，支持寄存器所对应的地址或直接地址：

```lldb
disassemble -a 0xabcdef
# 或则
disassmeble -a $pc
```





## 调用栈

使用 bt 命令可以打印出所有调用栈，后面加入编号可以指定打印几层

```lldb
bt
bt 1	# 打印第一层
```



## 附录

1. [官方LLDB文档](https://developer.apple.com/library/archive/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/document/lldb-terminal-workflow-tutorial.html)
