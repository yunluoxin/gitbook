# 安装多版本Python

## pyenv

### 安装

1. 直接利用 `homebrew` 进行便捷的安装：

    ```shell
    brew install pyenv
    ```

2. 配置环境变量。 
    
    1. 执行下面代码就配置好啦：

    ```shell
    echo '\neval "$(pyenv init -)"' >> ~/.zshrc
    ```

    2. 打开一个新的终端窗口，或者 执行以下命令使配置生效：

    ```shell
    source ~/.zshrc
    ```

3. 使用 `pyenv` 安装你所需的 `Python` 版本。例如，安装 Python 3.8.12 和 Python 3.9.7：

    ```shell
    pyenv install 3.8.12
    pyenv install 3.9.7
    ```

4. 验证

    - shell 中执行 ```python global 3.8.12```，然后执行 ```python --version``` 查看输出的是否是 python 3.8.12，打开新的shell窗口再次输出 `python` 版本，看是否对了;
    - shell 中执行 ```python global 3.9.7```，然后执行 ```python --version``` 查看输出的是否是 python 3.9.7，打开新的shell窗口再次输出 `python` 版本，看是否对了

    如果都通过✅，恭喜你，`pyenv` 安装成功了！


### 使用

1. 显示 `pyenv` 这个工具本身的版本：

    ```shell
    pyenv --version
    ```

2. 显示当前 pyenv 指向的 python 的版本号：

    ```shell
    pyenv version
    ```

    > 注意1和2的区别。

3. 查看当前已经安装的所有 `python` 版本：

    ```shell
    pyenv versions
    ```

    会输出类似这样的：

    ```txt
      system
    * 3.8.12 (set by /Users/east/.pyenv/version)
    ```

    * 指向的就是当前正在用的版本。

4. 列出当前可安装的所有 `python` 版本

    ```shell
    pyenv install -l
    ```

5. 安装某个版本的 `python`

    在得到可安装 `python` 版本列表后，你可以从列表中选择想要的版本进行安装。

    ```shell
    # pyenv install <你要安装的python版本>
    pyenv install 3.10.6
    ```

6. 卸载某个版本的 `python`

    ```shell
    pyenv uninstall <你要卸载的python版本>
    ```

7. 切换整个电脑（全局）的 `python` 版本，可以使用

    ```shell
    # pyenv global <python版本号>
    pyenv global 3.8.12
    ```

8. 如果你只想在 **shell** 中临时切换下 `python` 版本，可以使用

    ```shell
    # pyenv shell <python版本号>
    pyenv shell 3.8.12
    ```

9. 如果你想在 某个目录下 固定使用某个版本，可以这样：
    1. 先切换到你要设置的目录
    2. 在当前目录下执行：
        ```shell
        # pyenv local <python版本号>
        pyenv local 3.8.12
        ```
    > 使用 `pyenv local` 设置的 Python 版本会优先于全局设置和其他项目的设置。这对于项目中需要特定 Python 版本的开发环境非常有用，因为每个项目可以使用独立的 Python 版本，而不会相互干扰。

10. 切换回Mac系统自带的Python

    ```shell
    python global system
    ```

    和上面的规则一样，如果用 `global` 就是全局，用 `local` 就是目录等等，重点是版本号用 `system` 代替，就代表切回系统的 Python。


## pyenv-virtualenv

> **Python 虚拟环境**：在同一个 `python` 版本中，使用不同版本的 `python` 包，比如A项目用 `requests v1.0`， B项目用 `requests v2.0`。

> 而这个 `pyenv-virtualenv` 其实就是 `virtualenv`，但被整合到了 `pyenv` 中了，更加方便的使用。

### 安装

它也通过 `homebrew` 直接进行安装：

```shell
brew install pyenv-virtualenv
```

### 使用

1. 创建虚拟环境

    ```shell
    # pyenv virtualenv <python版本号> <虚拟环境名>
    pyenv virtualenv 2.7.18 myenv
    ```
    > 其中 `python` 版本号是指，你要基于哪个 `python` 版本，创建虚拟环境。这个版本号必须是 ```pyenv versions``` 可以看到的，即已经安装的 `python` 版本。

2. 查看所有的虚拟环境

    ```shell
    pyenv virtualenvs

    # 输出如下的内容：
    #   2.7.18/envs/myenv (created from /Users/xxxx/.pyenv/versions/2.7.18)
    #   myenv (created from /Users/xxxx/.pyenv/versions/2.7.18)
    ```

    可以看出，确实有我们刚创建好的环境。但是这里不知道为什么同一个环境会显示两次。。。

3. 使用创建好的虚拟环境

    ```shell
    # pyenv activate <虚拟环境名>
    pyenv activate myenv
    ``` 

    这时候，终端上命令前会显示 `(myenv)`，代表成功进入了虚拟环境。

    这时候，如果你执行 ```python --version``` 或者 ```pip list```，你会发现已经和刚才不一样了。

4. 退出虚拟环境

    ```shell
    pyenv deactivate
    ``` 

5. 删除虚拟环境

    ```shell
    # pyenv virtualenv-delete <虚拟环境名>
    pyenv virtualenv-delete myenv
    ```


### 另类使用方式

这个方式我是不小心试出来的，不知道网上有没有。

竟然 ```pyenv versions``` 会把所有的 `python` 版本以及虚拟环境都现实出来，那么是不是可以直接当做普通 `python` 版本进行切换呢？ 答案是：可以！！！

```shell
pyenv global myenv
```

这种全局切换后，整个电脑，都在用虚拟环境了。如果要退出，则切换到其他的 `python` 版本即可！

同理， ```pyenv shell myenv``` 则在当前 `shell` 生效，退出 `shell` 则失效了，这个比手动 `activate` 简直更方便！ 

这种方式的**区别**是，切换到虚拟环境的时候，终端的命令前是不会有 `(myenv)` 这种标识，让你感受不到当前是虚拟环境。

> 也可以对 `local` 使用虚拟环境！


## 小测试

Q: 如果给一个文件夹 A 设置了制定的 `python` 版本 3.10，这时候又对当前 `shell` 设置了版本 3.11，请问在这个文件夹现在是以多少 `python` 版本执行？

A: 经过测试，是 3.11，也就是说 `shell` 的优先级是更高的。 进一步说，优先级是 `shell` > `local` > `global` !