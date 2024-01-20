# Crack 知识集锦

1. 如何查看一个ipa包是否加壳？

   解压 ipa 包，然后通过 `otool -l app可执行文件 | grep crypt` 查看里面的可执行文件，如果打印出来东西，并且 cryptid = 1，则说明 ipa 文件是加壳的！
   ```shell
    east:WrappedBundle/ $ otool -l Mach-O文件 | grep crypt  				[17:27:48]
         cryptoff 1122304
        cryptsize 4096
          cryptid 1
   ```
   
1. 查看 Mach-O 的构造

   ```shell
   # 会打印出所有的 load command 信息
   otool -l Mach-O文件	
   
   # 打印 Mach-O 依赖的动态库
   otool -L Mach-O文件	
   ```

​	  