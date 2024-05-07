# 怎么同时使用两份ssh key

假如你的电脑已经有一份默认的 `ssh key` 了，这时候，你要新增一份 `key`，来连接不同的 `github` 账户（`github` 不同账户使用同一份 `key` 会失败！）

默认的那份私钥是 `~/.ssh/id_rsa`，公钥是 `~/.ssh/id_rsa.pub`



开始操作：

1. 先生成一份新的 `ssh key`

    ```shell
    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    ```

会提示你输入文件的位置！这里要使用两份key，所以要取名不一样的，这里取名 **github_work**， 输入 `~/.ssh/github_work`，回车！

成功后，可以在 `~/.ssh` 文件夹下生成两个文件：`github_work` 和 `github_work.pub` ！




2. 创建或者编辑 **~/.ssh/config** 文件

    ```text
    # 私人的github
    Host self
      User git
      HostName github.com
      AddKeysToAgent yes
      UseKeychain yes
      IdentityFile ~/.ssh/id_rsa
	
    # 工作的github
    Host work
      User git
      HostName github.com
      AddKeysToAgent yes
      UseKeychain yes
      IdentityFile ~/.ssh/github_work
    ```

	这样就把账户和密钥关联啦！



3. 添加新密钥到 SSH 代理：

   ```shell
   # 这里记得换成你的新密钥位置
   ssh-add --apple-use-keychain ~/.ssh/github_work
   
   # 控制台输出类似下面的消息，就是成功了：
   Identity added: /Users/east/.ssh/github_work (your_email@example.com)
   ```



3. 把公钥复制到网站上，记住，是**公钥、公钥、公钥**！！！

   将 `github_work.pub` 文件里的内容文本，复制到 `github` 或者其他需要公钥的位置！

   

4. 这个时候，系统就能自动处理哪个仓库用哪个了么？

    不行的。拉取的时候， `git clone git@github.com:xxx/abc.git`, 系统无法知道用哪个账户进行拉取，只能用原来默认的 `id_rsa`（测试过发现的）。所以要想用其他的账户，必须显式指定！

    ```
    # 把 git@github.com 改成你设置的别名，即 config 文件里的 Host
    git clone work:xxx/abc.git
	```
这样就可以用你指定的账户拉取啦！



> 你也可以把 `id_rsa`，即默认的密钥添加到 `ssh-agent` 中，这样拉取时候可以用最初默认的，也可以用 `git clone home:xxx/abc.git` 这种格式的！（参考第2步中的私人github，然后第3步添加 `id_rsa` 文件）



## 其他

查看已经添加到 ssh-agent 的数据列表：

```shell
ssh-add -l
```

