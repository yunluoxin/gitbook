# 安装多版本Python

## pyenv

### 安装

1. 直接利用homebrew进行便捷的安装：

```shell
brew install pyenv
```

2. 配置环境变量。将以下内容添加到你的终端配置文件（例如 `~/.zshrc`、`~/.bash_profile` 或者 `~/.bashrc`）中：

```shell
eval "$(pyenv init -)"
```

保存并关闭终端配置文件，然后重新打开一个新的终端窗口，或者执行以下命令使配置生效：

```shell
source ~/.bash_profile  # 或者是你的配置文件名称
```


3. 使用 `pyenv` 安装你所需的 Python 版本。例如，安装 Python 3.8.12 和 Python 3.9.7：

```shell
pyenv install 3.8.12
pyenv install 3.9.7
```

4. 验证

- shell 中执行 ```python global 3.8.12```，然后执行 ```python --version``` 查看输出的是否是 python 3.8.12，打开新的shell窗口再次输出 python 版本，看是否对了;
- shell 中执行 ```python global 3.9.7```，然后执行 ```python --version``` 查看输出的是否是 python 3.9.7，打开新的shell窗口再次输出 python 版本，看是否对了

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

3. 查看当前安装的所有 Python 版本：

    ```shell
    pyenv versions
    ```

    会输出类似这样的：

    ```txt
      system
    * 3.8.12 (set by /Users/east/.pyenv/version)
    ```

    * 指向的就是当前正在用的版本。

4. 安装某个版本的 `python`

    在得到可安装 `python` 版本列表后，你可以从列表中选择想要的版本进行安装。

    ```shell
    # pyenv install <你要安装的python版本>
    pyenv install 3.10.6
    ```

5. 卸载某个版本的 `python`

    ```shell
    pyenv uninstall <你要卸载的python版本>
    ```

6. 切换整个电脑（全局）的 python 版本，可以使用

    ```shell
    # pyenv global <python版本号>
    pyenv global 3.8.12
    ```

7. 如果你只想在 **shell** 中临时切换下 python 版本，可以使用

    ```shell
    # pyenv shell <python版本号>
    pyenv shell 3.8.12
    ```

8. 如果你想在 某个目录下 固定使用某个版本，可以这样：
    1. 先切换到你要设置的目录
    2. 在当前目录下执行：
        ```shell
        # pyenv local <python版本号>
        pyenv local 3.8.12
        ```
    > 使用 `pyenv local` 设置的 Python 版本会优先于全局设置和其他项目的设置。这对于项目中需要特定 Python 版本的开发环境非常有用，因为每个项目可以使用独立的 Python 版本，而不会相互干扰。

9. 切换回Mac系统自带的Python

    ```shell
    python global system
    ```

    和上面的规则一样，如果用 `global` 就是全局，用 `local` 就是目录等等，重点是版本号用 `system` 代替，就代表切回系统的 Python。