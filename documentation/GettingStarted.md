## 开始学习
> **⚠ 向后兼容性**

> 我们尽量保证在配置文件上向后兼容上一个版本的**Rebar**，并且将在文档中列出那些重大的变化点。

## 安装二进制包

在[这里](https://s3.amazonaws.com/rebar3/rebar3)下载每日构建版。并且确认其是可执行的（`chmod +x`）后复制到 `$PATH`中的目录中。

或者创建一个目录 `~/bin/` 来放置 `Rabar3` 命令，并且添加`export PATH=~/bin/:$PATH` 到shell配置文件（`~/.bashrc`或`~/.zshrc`）中。

## 源码安装
```shell
$ git clone https://github.com/rebar/rebar3.git
$ cd rebar3
$ ./bootstrap
```

现在你已经得到脚本文件 `Rebar3`了，你可以将它复制到 `$PATH` 下的任意目录。

## 创建项目

```shell
$ rebar3 new release myrelease
===> Writing myrelease/apps/myrelease/src/myrelease_app.erl
===> Writing myrelease/apps/myrelease/src/myrelease_sup.erl
===> Writing myrelease/apps/myrelease/src/myrelease.app.src
===> Writing myrelease/rebar.config
===> Writing myrelease/config/sys.config
===> Writing myrelease/config/vm.args
===> Writing myrelease/.gitignore
===> Writing myrelease/LICENSE
===> Writing myrelease/README.md
```

查看 [基础使用](https://github.com/zyuyou/rebar3_docs/blob/master/documentation/BasicUsage.md) 学习更多`rebar3`的使用方法。