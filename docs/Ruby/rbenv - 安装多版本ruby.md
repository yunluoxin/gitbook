# rbenv - 安装多版本ruby

## rbenv

`rbenv` 是 `rvm` 类似的东西，目标都是在一台电脑上，同时安装和使用多个版本的 `ruby`，而互不干扰。

那之前已经写过 [rvm 相关的文章](./RVM%E7%9A%84%E5%AE%89%E8%A3%85.md)了，为什么还要这个呢？因为
- 它的安装比 `rvm` 简单太多了，一步到位
- 功能更加的强大
- 和 [pyenv](../Python/%E5%AE%89%E8%A3%85%E5%A4%9A%E7%89%88%E6%9C%ACPython.md) 简直是亲兄弟，会了这个工具，自然会使用另外一个工具，语法一模一样

### 安装

1. 直接利用 `homebrew` 进行便捷的安装：

    ```shell
    brew install rbenv
    ```

2. 配置环境变量。 
    
    1. 执行下面代码就配置好啦：

    ```shell
    echo '\neval "$(rbenv init - zsh)"' >> ~/.zshrc
    ```

    2. 打开一个新的终端窗口，或者 执行以下命令使配置生效：

    ```shell
    source ~/.zshrc
    ```

3. 使用 `rbenv` 安装你所需的 `ruby` 版本。例如，安装 ruby 2.7.8 和 ruby 3.2.2：

    ```shell
    rbenv install 2.7.8
    rbenv install 3.2.2
    ```

4. 验证

    - shell 中执行 ```ruby global 2.7.8```，然后执行 ```ruby --version``` 查看输出的是否是 ruby 2.7.8，打开新的shell窗口再次输出 `ruby` 版本，看是否对了;
    - shell 中执行 ```ruby global 3.2.2```，然后执行 ```ruby --version``` 查看输出的是否是 ruby 3.2.2，打开新的shell窗口再次输出 `ruby` 版本，看是否对了

    如果都通过✅，恭喜你，`rbenv` 安装成功了！


### 使用

1. 显示 `rbenv` 这个工具本身的版本：

    ```shell
    rbenv --version
    ```

2. 显示当前 rbenv 指向的 ruby 的版本号：

    ```shell
    rbenv version
    ```

    > 注意1和2的区别。

3. 查看当前已经安装的所有 `ruby` 版本：

    ```shell
    rbenv versions
    ```

    会输出类似这样的：

    ```txt
      system
      2.7.8
    * 3.2.2 (set by /Users/east/.rbenv/version)
    ```

    * 指向的就是当前正在用的版本。

4. 列出当前可安装的所有 `ruby` 版本

    ```shell
    # 注意：-L 是大写的
    rbenv install -L
    ```

    查看当前可安装的，最新的 `ruby` 版本：
    
    ```shell
    # -l 是小写的！
    rbenv install -l
    ```

5. 安装某个版本的 `ruby`

    在得到可安装 `ruby` 版本列表后，你可以从列表中选择想要的版本进行安装。

    ```shell
    # rbenv install <你要安装的ruby版本>
    rbenv install 3.2.2
    ```

6. 卸载某个版本的 `ruby`

    ```shell
    rbenv uninstall <你要卸载的ruby版本>
    ```

7. 切换整个电脑（全局）的 `ruby` 版本，可以使用

    ```shell
    # rbenv global <ruby版本号>
    rbenv global 3.2.2
    ```

8. 如果你只想在 **shell** 中临时切换下 `ruby` 版本，可以使用

    ```shell
    # rbenv shell <ruby版本号>
    rbenv shell 3.2.2
    ```

9. 如果你想在 某个目录下 固定使用某个版本，可以这样：
    1. 先切换到你要设置的目录
    2. 在当前目录下执行：

        ```shell
        # rbenv local <ruby版本号>
        rbenv local 3.2.2
        ```
    3. 这时候在项目根目录下，其实有一个 `.ruby-version` 的文件，里面存放了 `ruby` 的版本。 `rbenv` 就是通过这个识别的。

    > 使用 `rbenv local` 设置的 ruby 版本会优先于全局设置和其他项目的设置。这对于项目中需要特定 ruby 版本的开发环境非常有用，因为每个项目可以使用独立的 ruby 版本，而不会相互干扰。

10. 切换回Mac系统自带的 `ruby`

    ```shell
    ruby global system
    ```

    和上面的规则一样，如果用 `global` 就是全局，用 `local` 就是目录等等，重点是版本号用 `system` 代替，就代表切回系统的 ruby。


## 小测试

Q: 如果给一个文件夹 A 设置了制定的 `ruby` 版本 2.7.8，这时候又对当前 `shell` 设置了版本 3.2.2，请问在这个文件夹现在是以多少 `ruby` 版本执行？

A: 经过测试，是 3.2.2，也就是说 `shell` 的优先级是更高的。 进一步说，优先级是 `shell` > `local` > `global` !


## 附录

[1]. [rbenv github](https://github.com/rbenv/rbenv)