# Git 大文件存储探索

## 前置

建立一个文件夹 `testgit`, 进入文件夹，`git init`初始化 git 仓库，初始化仓库总大小为 76KB.

```shell
east:testgit/ (master) $ du -sh . 
 76K	.
```

```shell
east:testgit/ (master) $ cd .git                                   
east:.git/ (master) $ du -sh *                                    
4.0K	HEAD
4.0K	config
4.0K	description
 60K	hooks
4.0K	info
  0B	objects
  0B	refs
```

可以看到初始化的 `objects` 是空的。其中 hooks 文件夹，都是一些 sample 文件，所以有占用了 60KB （不同版本git可能会有差异）



## 开始咯

### 普通模式

为了让效果更明显，我直接丢入一个大文件(二进制)：23.7MB，放入后文件夹详情如下：

```shell
east:testgit/ (master✗) $ du -sh *                             
 23M	ios.zip
east:testgit/ (master✗) $ du -h . 
  0B	./.git/objects/pack
  0B	./.git/objects/info
  0B	./.git/objects
4.0K	./.git/info
 60K	./.git/hooks
  0B	./.git/refs/heads
  0B	./.git/refs/tags
  0B	./.git/refs
 76K	./.git
 23M	.
```

`.git` 还没开始变化

1. 执行 `git add .`，文件夹发生了变化，`.git` 文件夹增长了差不多 `ios.zip`文件的大小 21MB，看来 zip 文件被提交到了**暂存区** 

```shell
 east:testgit/ (master✗) $ du -h .                                
 21M	./.git/objects/b4
  0B	./.git/objects/pack
  0B	./.git/objects/info
 21M	./.git/objects
4.0K	./.git/info
 60K	./.git/hooks
  0B	./.git/refs/heads
  0B	./.git/refs/tags
  0B	./.git/refs
 21M	./.git
 44M	.
 
 # 似乎 index 文件记录了暂存区情况？
 east:.git/ (master) $ cat index                                    
DIRCa�F^?�`�΀�����j��A�Ī{�����pQW��pios.zipE�خ�ؖ��
Hp1�ŀ��%
```

2. 执行 `git commit -m "init"`，提交修改。发现文件大小没有大变化，看来是从**暂存区**弄到**commit区域**了。但文件的物理位置似乎没变化，不知道是偶然还是必然，有待研究。`objects` 下多出来了一些文件。

```shell
east:testgit/ (master) $ du -h .                          
4.0K	./.git/objects/05
 21M	./.git/objects/b4
4.0K	./.git/objects/a2
  0B	./.git/objects/pack
  0B	./.git/objects/info
 21M	./.git/objects
4.0K	./.git/info
4.0K	./.git/logs/refs/heads
4.0K	./.git/logs/refs
8.0K	./.git/logs
 60K	./.git/hooks
4.0K	./.git/refs/heads
  0B	./.git/refs/tags
4.0K	./.git/refs
 21M	./.git
 44M	.
```

3. 删除 `ios.zip` 文件，然后 commit, 文件夹如下：

```shell
east:testgit/ (master) $ du -h .                             
4.0K	./.git/objects/66
4.0K	./.git/objects/05
 21M	./.git/objects/b4
4.0K	./.git/objects/a2
4.0K	./.git/objects/4b
  0B	./.git/objects/pack
  0B	./.git/objects/info
 21M	./.git/objects
4.0K	./.git/info
4.0K	./.git/logs/refs/heads
4.0K	./.git/logs/refs
8.0K	./.git/logs
 60K	./.git/hooks
4.0K	./.git/refs/heads
  0B	./.git/refs/tags
4.0K	./.git/refs
 21M	./.git
 21M	.
```

此时，从表面看起来 `testgit` 文件夹已经空了。但是，已经添加的 zip 文件，会永远存在 git 历史中啦。所以，体积不会低于 zip 文件大小的！

此时**暂存区**是空的，index 文件如下：

```shell
east:.git/ (master) $ cat index                              
DIRCTREE0 0
K�]�B�n��`�K�֒���Ir�?x}��2�Z�7��6�F��%
```

看来 index 文件多少和**暂存态**有关系!

### 使用 Git-LFS

删除整个 `testgit` 文件夹，并把前置重新做一遍，开始新的尝试❤️

1. 通过 `git lfs install`，让仓库支持 git lfs

```shell
east:testgit/ (master✗) $ git lfs install                      
Updated git hooks.
Git LFS initialized.

east:testgit/ (master) $ du -sh .                            
 92K	.
 
 east:.git/ (master) $ du -sh *                          
4.0K	HEAD
4.0K	config
4.0K	description
 76K	hooks
4.0K	info
  0B	lfs
  0B	objects
  0B	refs
```

仓库比之前大了16KB，可以看出主要是 `hooks` 文件夹大了16KB。因为 git lfs 是通过 **git hook**  实现的，所以默认会放几个钩子文件在这下面。没有带 `.sample` 后缀，一共4个文件，每个4KB。

```shell
east:hooks/ (master) $ du -sh * | grep -v "sample"             
4.0K	post-checkout
4.0K	post-commit
4.0K	post-merge
4.0K	pre-push
```

2. 发现根目录下并没有自动生成 `.gitattributes` 文件，我们开始 track 任意文件后，会自动生成！这里我们 track zip 文件，因为等会还是用 `ios.zip` 文件做测试哦：

```shell
east:testgit/ (master✗) $ git lfs track "*.zip"                   
Tracking "*.zip"
# 自动生成了 .gitattributes 文件，内容如下：

east:testgit/ (master✗) $ cat .gitattributes                      
*.zip filter=lfs diff=lfs merge=lfs -text
```

3. 加入 `ios.zip` 文件，发生文件 `.git` 依旧没有变化，和普通 git 时候一样

```shell
east:testgit/ (master✗) $ du -h .                                   
  0B	./.git/objects/pack
  0B	./.git/objects/info
  0B	./.git/objects
4.0K	./.git/info
 76K	./.git/hooks
  0B	./.git/refs/heads
  0B	./.git/refs/tags
  0B	./.git/refs
  0B	./.git/lfs/tmp
  0B	./.git/lfs
 92K	./.git
 23M	.
```

4. 执行 `git add .`，文件发生了变化，并且和普通 git 有很大变化：
```shell
east:testgit/ (master✗) $ git add .                            
east:testgit/ (master✗) $ du -h .                               
  0B	./.git/objects/pack
  0B	./.git/objects/info
4.0K	./.git/objects/3a
4.0K	./.git/objects/48
8.0K	./.git/objects
4.0K	./.git/info
 76K	./.git/hooks
  0B	./.git/refs/heads
  0B	./.git/refs/tags
  0B	./.git/refs
 23M	./.git/lfs/objects/ba/9e
 23M	./.git/lfs/objects/ba
 23M	./.git/lfs/objects
  0B	./.git/lfs/tmp
 23M	./.git/lfs
 23M	./.git
 45M	.
```

此时被暂存的文件实际上是放入 `.git/lfs` 文件夹下，而不是普通的 `.git/objects` 下，这是一个很大的区别！lfs 文件都是被放在 `.git/lfs` 下！
但你细看，会发现 `.git/objects` 也增长了 8KB，这个应该就是指针的大小，指针是当做普通文件，暂存到 `.git/objects` 下！

另外发现，这时候 git lfs track 列表中，已经有了这个 zip 文件：

```shell
east:testgit/ (master✗) $ git lfs ls-files                          
ba9ea02c86 * ios.zip
```



5. 加塞了一个实验。。我把**暂存区**的文件恢复到**工作区**，发生 lfs 文件夹大小并没有下降！说明文件残留了，但是此时 `git lfs ls-files` 也没有显示这个文件被 track ：

```shell
east:testgit/ (master✗) $ git reset .                               
east:testgit/ (master✗) $ du -h .                                   
  0B	./.git/objects/pack
  0B	./.git/objects/info
4.0K	./.git/objects/3a
4.0K	./.git/objects/e6
4.0K	./.git/objects/48
 12K	./.git/objects
4.0K	./.git/info
 76K	./.git/hooks
  0B	./.git/refs/heads
  0B	./.git/refs/tags
  0B	./.git/refs
 23M	./.git/lfs/objects/ba/9e
 23M	./.git/lfs/objects/ba
 23M	./.git/lfs/objects
  0B	./.git/lfs/tmp
 23M	./.git/lfs
 23M	./.git
 45M	.
 
 east:testgit/ (master✗) $ git lfs ls-files                                 
# 空的，没有任何输出！
```

然后，我建立了一个空的 `readme.md` 并**只 git add / commit 它**，然后推送到远程未 init 的仓库上，发现远程只有 96KB 大小！所以这个 lfs 文件缓存算是永久的留在本地了。坑！

```shell
east:testgit/ (master✗) $ touch readme.md                            
east:testgit/ (master✗) $ git add readme.md                    
east:testgit/ (master✗) $ git commit -m "init"
[master (root-commit) 3ff3b0e] init
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 readme.md
east:testgit/ (master✗) $ git remote add origin git@xx.com:east/testgit.git
east:testgit/ (master✗) $ git push -u origin master              
```

这时候我想，是不是刚好实验下 `git lfs prune` 的删除历史文件功能，到底行不行呀？试下：

```shell
east:testgit/ (master✗) $ git lfs prune                            
prune: 1 local object(s), 0 retained, done.
prune: Deleting objects: 100% (1/1), done.
```

哈哈，竟然成功给删除了！验证下：

```shell
east:testgit/ (master✗) $ du -h .                                    
# 越来越长了，仅列出部分...
4.0K	./.git/lfs/cache/locks/refs/heads/master
4.0K	./.git/lfs/cache/locks/refs/heads
4.0K	./.git/lfs/cache/locks/refs
4.0K	./.git/lfs/cache/locks
4.0K	./.git/lfs/cache
  0B	./.git/lfs/objects/ba/9e
  0B	./.git/lfs/objects/ba
  0B	./.git/lfs/objects
  0B	./.git/lfs/tmp
4.0K	./.git/lfs
144K	./.git
 23M	.
```

确实 `.git/lfs` 文件夹下的那个残留的 21MB 的 zip 文件消失了！

6. 回到正轨，提交 `ios.zip`，发现文件夹差不多和**只暂存它**的时候差不多了，还是在 `.git/lfs` 下：

```shell
east:testgit/ (master✗) $ git add .                                  
east:testgit/ (master✗) $ git commit -m "add ios.zip"                
[master f484e07] add ios.zip
 2 files changed, 4 insertions(+)
 create mode 100644 .gitattributes
 create mode 100644 ios.zip
east:testgit/ (master) $ du -h .   
# 越来越长了，仅列出部分...
4.0K	./.git/lfs/cache/locks/refs/heads/master
4.0K	./.git/lfs/cache/locks/refs/heads
4.0K	./.git/lfs/cache/locks/refs
4.0K	./.git/lfs/cache/locks
4.0K	./.git/lfs/cache
 23M	./.git/lfs/objects/ba/9e
 23M	./.git/lfs/objects/ba
 23M	./.git/lfs/objects
  0B	./.git/lfs/tmp
 23M	./.git/lfs
 23M	./.git
 45M	.
```

把它推送到远程，可以看到它是先上传了 lfs 文件，然后才提交其他的普通提交：

```shell
east:testgit/ (master) $ git push                                    
Locking support detected on remote "origin". Consider enabling it with:
  $ git config lfs.https://xx.com/east/testgit.git/info/lfs.locksverify true
Uploading LFS objects: 100% (1/1), 24 MB | 1.7 MB/s, done.
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 463 bytes | 463.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
To xx.com:east/testgit.git
   3ff3b0e..f484e07  master -> master
```

接下来，删除 `ios.zip` 文件，然后提交修改并推送到远程，我们会发现远程仓库的大小几乎没变化！ 因为 git 是可以回滚到每个节点的，所以一旦被 track，就会永久存在了。哪怕是 lfs 也一样！

本地仓库的大小也没有发生变化！这次删除的其实只是一个文件指针！

```shell
east:testgit/ (master) $ du -h .                                  
# 越来越长了，仅列出部分...
4.0K	./.git/lfs/cache/locks/refs/heads/master
4.0K	./.git/lfs/cache/locks/refs/heads
4.0K	./.git/lfs/cache/locks/refs
4.0K	./.git/lfs/cache/locks
4.0K	./.git/lfs/cache
 23M	./.git/lfs/objects/ba/9e
 23M	./.git/lfs/objects/ba
 23M	./.git/lfs/objects
  0B	./.git/lfs/tmp
 23M	./.git/lfs
 23M	./.git
 23M	.
```

这时候如果执行 `git lfs prune`，你会发现本地仓库会降到只有几百KB（当前我的是172K）！ 因为当前工作区并没有引用到这个 lfs 文件，所以从本地被删除了！

```shell
east:testgit/ (master) $ git lfs prune                             
prune: 1 local object(s), 0 retained, done.
prune: Deleting objects: 100% (1/1), done.
east:testgit/ (master) $ du -h .                                 
# 越来越长了，仅列出部分...
4.0K	./.git/lfs/cache/locks/refs/heads/master
4.0K	./.git/lfs/cache/locks/refs/heads
4.0K	./.git/lfs/cache/locks/refs
4.0K	./.git/lfs/cache/locks
4.0K	./.git/lfs/cache
  0B	./.git/lfs/objects/ba/9e
  0B	./.git/lfs/objects/ba
  0B	./.git/lfs/objects
  0B	./.git/lfs/tmp
4.0K	./.git/lfs
168K	./.git
172K	.
```

> 这时候你的本地工作区是 `clean` 的，你无法暂存，无法提交！



7. 再开一个窗口，把远程代码拉到本地，你会发现也是就几百KB左右！甚至更少！！！

```shell
east:~/ $ du -sh testgit                                            
116K	testgit

# 为什么变更少了？ 进入看看
east:~/ $ cd testgit

east:testgit/ (master) $ du -h .                             
8.0K	./.git/objects/pack
  0B	./.git/objects/info
8.0K	./.git/objects
4.0K	./.git/info
4.0K	./.git/logs/refs/heads
4.0K	./.git/logs/refs/remotes/origin
4.0K	./.git/logs/refs/remotes
8.0K	./.git/logs/refs
 12K	./.git/logs
 60K	./.git/hooks
4.0K	./.git/refs/heads
  0B	./.git/refs/tags
4.0K	./.git/refs/remotes/origin
4.0K	./.git/refs/remotes
8.0K	./.git/refs
112K	./.git
116K	.
```

结果发现，甚至都没有创建 `.git/lfs` 文件夹！应该是做了优化了！

这时候如果你切换到某个commit（`ios.zip` 文件存在于工作区的），git lfs 会自动把 zip 文件下载下来，这时候，仓库又大了42MB（工作区一份，lfs 中一份）。

```shell
east:testgit/ (master) $ git log
commit 224a47bcd8e96338fd83c1f63c80bb8ebfe2701a (HEAD -> master, origin/master)
Author: East <east@xx.com>
Date:   Thu Dec 16 23:34:22 2021 +0800

    Deleted ios.zip

commit f484e0779b3a01cf01b2b6f923a6214f3e7928e8
Author: East <east@xx.com>
Date:   Thu Dec 16 23:26:23 2021 +0800

    add ios.zip

commit 3ff3b0eeaab17ac29aaf76fcd1fbca55adf380b6
Author: East <east@xx.com>
Date:   Thu Dec 16 23:10:55 2021 +0800

    init
    
east:testgit/ (master) $ git checkout f484e0779b3a01cf01b2b6f923a6214f
Note: switching to 'f484e0779b3a01cf01b2b6f923a6214f'.
# 中间省略打印的...
HEAD is now at f484e07 add ios.zip

east:testgit/ (f484e07) $ du -h .   
# 越来越长了，仅列出部分...
  0B	./.git/lfs/incomplete
 23M	./.git/lfs/objects/ba/9e
 23M	./.git/lfs/objects/ba
 23M	./.git/lfs/objects
 16K	./.git/lfs/logs
  0B	./.git/lfs/tmp
 23M	./.git/lfs
 23M	./.git
 46M	.
```

