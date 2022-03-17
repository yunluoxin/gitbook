# Ruby 中执行 Shell

>  来自文章 [Ruby执行Shell的6种方法](https://droidyue.com/blog/2014/11/18/six-ways-to-run-shell-in-ruby/)

## exec

exec会将指定的命令替换掉当前进程中的操作，指定命令结束后，进程结束。所以也是无法知道指令执行的结果，是否成功失败！

```ruby
puts "abc"
exec('echo "hello world!"')
puts "def"

# 控制台输出:
abc
hello world!

# def 由于进程结束，无法输出了！
```



## system

system 和 exec 相似，但是 system 执行的命令不会是在当前进程，而是在一个新创建的进程。system 会返回布尔值来表明命令执行结果是成功还是失败。

```ruby
system 'ls -l'
puts $?		# system 会将进程退出的状态码放在 $? 里。 如果为0，则代表正常退出，反之异常。

# 控制台输出:
total 80
-rw-r--r--@ 1 east  staff   877 Jan 12  2021 File-.rb
-rw-r--r--@ 1 east  staff   307 Nov 10  2020 Re-.rb
-rw-r--r--@ 1 east  staff   523 Nov 10  2020 Time-.rb
pid 11377 exit 0
```

​	system会将进程的退出的状态码赋值给$?，如果程序正常退出，$?的值为0，否则为非0。通过检测退出的状态码我们可以在ruby脚本中抛出异常或者进行重试操作。

> 注：在Unix-like系统中进程的退出状态码以0和非0表示，0代表成功，非0代表失败。



## 反引号(`)

使用反引号是shell中常用的获取命令输出内容的方法，在ruby中也是可以，而且一点都需要做改变。使用反引号执行命令也会将命令在另一个进程中执行。

```ruby
s=`echo 'Hello world'`
puts s
s=`date`
puts s

# 控制台输出:
Hello world
Thu Mar 17 16:08:12 CST 2022
```

 注意，$?已经不再是上述的那样单纯的退出状态码了，它实际上是一个Process::Status对象。

`$?.to_i`  ===>   状态码

$?.to_s     ===>   包含了进程 id，退出状态码等信息的字符串

```ruby
s=`echo 'echoooooooo'`
puts $?.to_i
puts $?.to_s

# 控制台输出:
0
pid 13105 exit 0
```



### 问题

使用反引号的一个结果就是我们只能得到标准的输出（stdout）而不能得到标准的错误信息(stderr)。

即 如果脚本执行是出错的，我们无法获取到错误信息：

```ruby
s=`perl -e "warn 'dust in the wind'"`
puts '----'
puts s

# 控制台输出:
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LC_ALL = (unset),
	LC_CTYPE = "zh_CN.UTF-8",
	LANG = "zh_CN_#Hans.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
dust in the wind at -e line 1.
----
  
```

虽然控制台输出了，但是那是其他进程抛出的。变量 `s`并没有获取到值。 



## Open4#popen4

这个需要用 gem 安装模块：

```ruby
gem install open4
```

然后使用时候加入 `require "open4"`

```ruby
require "open4"
pid, stdin, stdout, stderr = Open4::popen4 "false"
puts pid, stdin, stdout, stderr
puts $?

# 控制台输出
14597
#<IO:0x00000001578a3e98>
#<IO:0x00000001578a3e48>
#<IO:0x00000001578a3d80>

```

可以看出， **stdin**, **stdout**, **stderr** 这3个，都是 `IO` 对象，而 **$?** 为空。 

通过 `Process::waitpid2` 加上对应的进程 ID 获得进程退出状态：

```ruby
ignored, status = Process::waitpid2 pid
puts ignored, status

# 控制台输出
18805
pid 18805 exit 1
# status 里就是状态码，进程 id 和退出状态整合，和上面的反引号的 `$?.to_s` 一样
```

具体的使用，直接看顶部的文章列表。大多数情况下，前面3种都够用了。