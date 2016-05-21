## 配置
## 应用版本

目前版本系统支持`git`（别名：`semver`），`hg` 或者其他拥有对应`resource handler`插件的名称：

```erlang
{application, <app_name>,
 [{description, ""},
  {vsn, "git"},
  {registered, []},
  {modules, []},
  {applications, [kernel, stdlib]},
  {env, []}
 ]}.
```

当构建系统时，`vsn`的值将被替换可识别的唯一的版本号。

## Artifacts - 人工产品

Artifacts 是一些不同于Erlang模块，但在编译（包括编译hook）完成后必须存在的文件。这有助于`rebar3`确认那些非Erlang形式的依赖是否构建成功。
例如C的共享库，模板渲染或者`escript`的生成。

如果一个依赖构建成功（意味着其`.app`文件中列举的模块的`.beam`文件和所有artifacts都生成了），那么后续`rebar3`的命令不会去执行该依赖的编译和编译钩子。


```erlang
{artifacts, [file:filename_all()]}.
```
路径是在`_build/<profiles>`的相对路径，例如下面确保`escript`命令生成artifact - `rebar3`:

```erlang
{escript_name, rebar3}.

{provider_hooks, [{post, [{compile, escriptize}]}]}.

{artifacts, ["bin/rebar3"]}.
```

## 编译

编译参数可以使用`erl_opts`设置， Erlang的编译模块[文档](http://www.erlang.org/doc/man/compile.html)列举了可用编译参数。

```
{erl_opts, []}.
```

另外也可以根据操作系统平台设置指定参数。

```erlang
 {erl_opts, [{platform_define,
               "(linux|solaris|freebsd|darwin)",
               'HAVE_SENDFILE'},
              {platform_define, "(linux|freebsd)",
                'BACKLOG', 128},
              {platform_define, "R13",
                'old_inets'}]
}.
```

另外一个独立的参数用来声明哪些模块应该优先编译：

```
{erl_first_files, ["src/mymodule.erl", "src/mymodule.erl"]}.
```

其他一些常用参数：

```erlang
{validate_app_modules, true}. % Make sure modules in .app match those found in code
{app_vars_file, undefined | Path}. % file containing elements to put in all generated app files
```

支持其他Erlang相关的编译器的编译配置参数：

* [Leex compiler](http://erlang.org/doc/man/leex.html) with `{xrl_opts, [...]}`
* [SNMP MIB Compiler](http://www.erlang.org/doc/apps/snmp/snmp_mib_compiler.html) with `{mib_opts, [...]}`
* [Yecc compiler](http://erlang.org/doc/man/yecc.html) with `{yrl_opts, [...]}`

## Common Test

```
{ct_compile_opts, [...]}. % {erl_opts, ...} but for CT
{ct_first_files, [...]}. % {erl_first_files, ...} but for CT
{ct_opts, [...]}. % same as options for ct:run_test(...)
```

`ct_opts`的相关指引: [http://www.erlang.org/doc/man/ct.html#run_test-1](http://www.erlang.org/doc/man/ct.html#run_test-1)

## Cover

使用`{cover_enabled, true}`在[测试](RunningTests.md)中启动代码覆盖率. `cover` provider将在跑完测试用例后执行以显示测试报告。 
设置`{cover_opts, [verbose]}`强制在终端中打印代码覆盖率测试报告。 

## Directories

以下参数用于配置相关目录，下述值是默认值：

```erlang
%% directory for artifacts produced by rebar3
{base_dir, "_build"}.
%% directory in '<base_dir>/<profile>/' where deps go
{deps_dir, "lib"}.
%% where rebar3 operates from; defaults to the current working directory
{root_dir, "."}.
%% where checkout dependencies are to be located
{checkouts_dir, "_checkouts"}.
%% directory in '<base_dir>/<profile>/' where plugins go
{plugins_dir, "plugins"}.
%% directories where OTP applications for the project can be located
{project_app_dirs, ["apps/*", "lib/*", "."]}.
%% Directories where source files for an OTP application can be found
{src_dirs, ["src"]}.
%% Paths to miscellaneous Erlang files to compile for an app
%% without including them in its modules list
{extra_src_dirs, []}. 
```

而且，rebar3 保存一些配置数据在`~/.config/rebar3`和缓存一部分数据在`~/.cache/rebar3`，但是也可以通过设置`{global_rebar_dir, "./some/path"}`来覆盖那些数据。

## EDoc

[EDoc](http://www.erlang.org/doc/man/edoc.html#run-3)支持的参数都可以在`{edoc_opts, [...]}`中使用。

## Escript

```erlang
{escript_main_app, AppName}. % if more than 1 app, specify which is main
{escript_name, "FinalName"}. % name of final generated escript
{escript_incl_apps, [App]}. % apps (other than main and deps) to be included
{escript_incl_extra, ["path/*"]}. % other files to include in escript
{escript_shebang, "#!/usr/bin/env escript\n"}. % executable line
{escript_comment, "%%\n"}. % comment at top of escript file
{escript_emu_args, "%%! -escript main ~s -pa ~s/~s/ebin\n"}. % emulator args
```

## EUnit

```erlang
{eunit_compile_opts, [...]}. % {erl_opts, ...} but for eunit
{eunit_first_files, [...]}. % {erl_first_files, ...} but for CT
{eunit_opts, [...]}. % same as options for eunit:test(Tests, ...)
```

Eunit相关参考: [http://www.erlang.org/doc/man/eunit.html#test-2](http://www.erlang.org/doc/man/eunit.html#test-2)


## Overrides

Overrides提供三种方式： add, override on app and override on all.

```
{overrides, [{add, app_name(), [{atom(), any()]},
             {override, app_name(), [{atom(), any()]},
             {override, [{atom(), any()]} 
            ]}.
```

这些配置同样适用于依赖，同时依赖也可以拥有它们自己的overrides，应用顺序为：overrides on all, per app overrides, per app additions。


## Hooks

有两种类型的hook: shell hooks 和 provider hooks

### Shell Hooks 

Hooks provide a way to run arbitrary shell commands before or after hookable providers, optionally first matching on the type of system to choose which hook to run.

```erlang
-type hook() :: {atom(), string()}
                | {string(), atom(), string()}.

{pre_hooks, [hook()]}.
{post_hooks, [hook()]}.
```

使用rebar3的`pre_hook`构建[merl](https://github.com/richcarl/merl)的例子：

```erlang
{pre_hooks, [{"(linux|darwin|solaris)", compile, "make -C \"$REBAR_DEPS_DIR/merl\" all -W test"},
             {"(freebsd|netbsd|openbsd)", compile, "gmake -C \"$REBAR_DEPS_DIR/merl\" all"},
             {"win32", compile, "make -C \"%REBAR_DEPS_DIR%/merl\" all -W test"},
             {eunit, "erlc -I include/erlydtl_preparser.hrl -o test test/erlydtl_extension_testparser.yrl"},
             {"(linux|darwin|solaris)", eunit, "make -C \"$REBAR_DEPS_DIR/merl\" test"},
             {"(freebsd|netbsd|openbsd)", eunit, "gmake -C \"$REBAR_DEPS_DIR/merl\" test"},
             {"win32", eunit, "make -C \"%REBAR_DEPS_DIR%/merl\" test"}
            ]}.
```

### Provider Hooks

Providers are also able to be used as hooks. The following hook runs clean before compile runs. To execute commands in a namespace a tuple is used as second argument.

```
{provider_hooks, [{pre, [{compile, clean}]}
                  {post, [{compile, {erlydtl, compile}}]}]}
```

### Hookable Providers

只有特定内置Providers才支持Provider Hooks，作用范围取决于Provider是作用于单个应用还是整个项目。


Provider | before and after
-----   |  -----
clean | each application and dependency
common test | the entire run
compile | each application and dependency
eunit | the entire run
release | the entire run
tar | the entire run

## Dialyzer

```erlang
-type warning() :: no_return | no_unused | no_improper_lists | no_fun_app | no_match | no_opaque | no_fail_call | no_contracts | no_behaviours | no_undefined_callbacks | unmatched_returns | error_handling | race_conditions | overspecs | underspecs | specdiffs

{dialyzer, [
            {warnings, [warning()]},
            {get_warnings, boolean()},
            {plt_extra_apps, [atom()]},
            {plt_location, local | file:filename()},
            {plt_prefix, string()},
            {base_plt_apps, [atom(), ...]},
            {base_plt_location, global | file:filename()},
            {base_plt_prefix, string()}
          ]}.
```

## Relx

查看[发布](Releases.md)

## Shell

如果rebar.config配置中有`relx`项配置，使用`rebar3 shell`会打开一个REPL的shell环境并自动启动relx配置中的应用。但是shell启动的应用可以通过在rebar.config添加`{shell_apps, [App1, App2]}`指定，注意该配置会覆盖relx的配置。 

## XRef

```erlang
{xref_warnings,false}.
{xref_extra_paths,[]}.
{xref_checks,[undefined_function_calls,undefined_functions,locals_not_used,
              exports_not_used,deprecated_function_calls,
              deprecated_functions]}
{xref_queries,[{"(xc - uc) || (xu - x - b - (\"mod\":\".*foo\"/\"4\"))", []}]}
```

