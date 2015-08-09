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

查看[命令](documentation/Commands.md)获得更多关于`new`命令的可用选项参数，
查看[模板](tutorials/Templates.md)学习如何使用自定义模板。

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

查看[依赖](documentation/Dependencies.md) 获取更多依赖处理的相关信息。

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

> 跟上一个版本的rebar不一样，rebar3使用`compile`命令获取所有本地没有的依赖并且编译。这是使用提供依赖的插件实现的，你可以查看[插件教程](documentation/Plugins.md)。

## 输出文件格式
安装依赖，构建releases或者其他任何形式输出文件都将写入项目根目录下的`_build`目录。

```shell
_build/
└── default
  └── lib  
    ├── cowboy
    ├── cowlib
    └── ranch
```

查看 [环境配置](documentation/Profiles.md) 获取更多 `_build` 目录相关信息。

## 测试
测试用例默认放在`test`目录下，`eunit`的测试方法也可以放在单独的模块中。

只有在测试时才需要的依赖包，可以声明在`test`环境配置中：
```erlang
{profiles, [
           {test, [{deps, [
                          {meck, ".*",
                           {git, "git://github.com/eproxus/meck.git", {tag, "0.8.2"}}}
                          ]}
                  ]}
           ]
}.

```

现在`rebar3 ct`命令将会获取并下载`meck`依赖到`_build/test/lib/`目录中，但是该依赖并不会加到锁文件`rebar.lock`

```
_build/
   └── test
     └── lib
       └── meck
```

## 目标系统与发布

版本发布使用工具 [relx](https://github.com/erlware/relx)

> **⚠ 使用 Relx 替代 Reltool**

> 跟上一个版本的rebar不一样，rebar3不再提供reltool工具的任何接口实现。但毕竟reltool是Erlang/OTP的内置库，所以你还是可以使用它。

创建一个release架构的项目会默认包含`relx`的配置在`rebar.config`中，执行下述命令：

```shell
$ rebar3 new release myrel
===> Writing myrel/apps/myrel/src/myrel_app.erl
===> Writing myrel/apps/myrel/src/myrel_sup.erl
===> Writing myrel/apps/myrel/src/myrel.app.src
===> Writing myrel/rebar.config
===> Writing myrel/config/sys.config
===> Writing myrel/config/vm.args
===> Writing myrel/.gitignore
===> Writing myrel/LICENSE
===> Writing myrel/README.md
```

`rebar.config`中有一些`relx`配置的例子：

```erlang
{relx, [{release, {myrel, "0.0.1"},
         [myrel]},

        {dev_mode, true},
        {include_erts, false},

        {extended_start_script, true}
       ]
}.


{profiles, [
           {prod, [{relx, [{dev_mode, false},
                           {include_erts, true}]}]}
           ]
}.

```

上述配置提供了一些默认的选项用于开发版（default环境）打包或者生产版本（prod环境）打包。
打包生产环境时我们通常会构建一个包含`erts`并且没有包含链接（`dev_mode`设置false）的软件包，

```shell
$ rebar3 release
===> Verifying default dependencies...
===> Compiling myrel
===> Starting relx build process ...
===> Resolving OTP Applications from directories:          
          _build/default/lib
          /usr/lib/erlang/lib
===> Resolved myrel-0.1.0
===> Dev mode enabled, release will be symlinked
===> release successfully created!
```

Relx dev_mode
> **⚠ Relx的开发模式**

> 由于默认的环境配置中的发布配置将`dev_mode`设置为`true`，所以生成`_build/rel/myrel/lib`发布目录下的应用是链接到`_build/lib`和`apps/myrel`目录下的。所以开发过程重新编译应用并不需要重新创建发布包，只需要编译完后重启节点或者重加载代码模块。

使用默认的`rebar.config`配置创建一个发布版本的压缩包，只需要环境参数为`prod`并且执行`tar`命令：

```shell
$ rebar3 as prod tar
===> Verifying default dependencies...
===> Compiling myrel
===> Starting relx build process ...
===> Resolving OTP Applications from directories:
          .../myrel/apps
          /usr/lib/erlang/lib
===> Resolved myrel-0.1.0
===> Including Erts from /usr/lib/erlang
===> release successfully created!
===> tarball myrel/_build/rel/myrel/myrel-0.1.0.tar.gz successfully created!
```

查看[发布章节](documentation/Releases)获取更多关于软件发布的信息。


