## 基础使用

使用rebar3构建项目。

> **⚠ 仅支持OTP**

> Rebar3 仅处理遵循OTP组织结构的项目。上一个版本的rebar并不要求项目组织结构遵循OTP的标准，它可以编译目录下的任何Erlang源文件。

## 创建 App 或者 Release

Rebar3 包含用来创建 **applications**，**library applications**（没有`start/2`），**releases** 和 **plugins** 项目的模板。
使用`new`命令基于一种模板创建项目。`new` 接受 `lib`， `app`，`release` 和 `plugin` 作第一个参数，项目名字作为第二个参数。

```shell
$ rebar3 new app myapp
===> Writing myapp/src/myapp_app.erl
===> Writing myapp/src/myapp_sup.erl
===> Writing myapp/src/myapp.app.src
===> Writing myapp/rebar.config
===> Writing myapp/.gitignore
===> Writing myapp/LICENSE
===> Writing myapp/README.md
```

查看[命令](https://github.com/zyuyou/rebar3_docs/blob/master/documentation/Commands.md)获得更多关于`new`命令的可用选项参数，
查看[模板](https://github.com/zyuyou/rebar3_docs/blob/master/tutorials/Templates.md)学习如何使用自定义模板。

## 添加依赖
依赖定义在`rebar.config`文件中的`deps`配置下：
```erlang
{deps, [
        {cowboy, {git, "git://github.com/ninenines/cowboy.git", {tag, "1.0.1"}}}
        ]
}.
```

> **⚠ 忽略SCM的版本号**

> rebar3 支持旧格式的依赖定义以便于向后兼容, 例如 `{cowboy, ".*", {git, "git://github.com/ninenines/cowboy.git", {tag, "1.0.1"}}}`, 第二个参数，在这里是 ".*" 将被忽略。

现在你可以把依赖加到项目应用的`.app.src`文件中：

```erlang
{application, <APPNAME>,
 [{description, ""},
  {vsn, "<APPVSN>"},
  {registered, []},
  {modules, []},
  {applications, [
                 kernel
                 ,stdlib
                 ,cowboy
                 ]},
  {mod, {<APPNAME>_app, []}},
  {env, []}
 ]}.
```

查看[依赖](https://github.com/zyuyou/rebar3_docs/blob/master/documentation/Dependencies.md) 获取更多依赖处理的相关信息。

## 构建

获取依赖和编译所有应用只使用`compile`一个命令。

```shell
$ rebar3 compile
==> Verifying dependencies...
==> Fetching cowboy
==> Fetching ranch
==> Fetching cowlib
==> Compiling cowlib
==> Compiling ranch
==> Compiling cowboy
==> Compiling myapp
```

> **⚠ 依赖获取**

> 跟上一个版本的rebar不一样，rebar3使用`compile`命令获取所有本地没有的依赖并且编译。这是使用提供依赖的插件实现的，你可以查看[插件教程](https://github.com/zyuyou/rebar3_docs/blob/master/documentation/Plugins.md)。

## 输出格式
安装依赖，构建releases或者其他任何输出都将写入项目根目录下的`_build`目录。

```shell
_build/
└── default
  └── lib  
    ├── cowboy
    ├── cowlib
    └── ranch
```


