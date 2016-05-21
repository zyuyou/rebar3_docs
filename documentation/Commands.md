## 命令
> rebar3 可运行的任务

每一个命令代表一个运行一组providers的任务。

## as
高优先级的任务，提供一个profile的配置名和一系列在该profile下运行的其他任务。

## compile
确保所有依赖应用有效率，如果本地还没有就会通过配置中的方式获取它们。`compile`将会编译这些依赖应用和项目应用的`.app.src`和`*.erl`文件。

## clean
从应用中移除已经编译生成的`beam`文件。

Option | Type | Description
-----   |  ----- | -----
--all/-a | none | 清除所有应用已经生成的beam文件，包括依赖应用的

## ct

Option | Type | Description
-----   |  ----- | -----
--dir | 多个目录用逗号分割(--dir "test1","test2") | 在指定目录编译并且运行所有测试套件
--suite | 多个目录用逗号分割(如上类似) | 编译并且运行指定测试套件，必须使用全路径,绝对路径和相对路径都可以，文件后缀可以不加。
--group | 多个目录用逗号分割(如上类似) | 测试组，查看 [Common Test Documentation](www.erlang.org/doc/apps/common_test/index.html)
--config | 多个目录用逗号分割(如上类似) | ct测试的配置文件，查看 [Common Test Documentation](www.erlang.org/doc/apps/common_test/index.html)
--logdir | 字符串 | 测试日志输出目录，查看[Common Test Documentation](www.erlang.org/doc/apps/common_test/index.html)。默认：`_build/test/logs`
-v,--verbose | Boolean | 启动详细输出模式。默认：`false`
-c,--cover | Boolean | 生成cover数据

## cover
当`Common Test`和`Eunit`测试套件时执行覆盖率分析。使用`rebar3 do ct, cover`, `rebar3 do eunit,cover`或者`rebar3 do eunit,ct,cover`。
当rebar.config配置启用了`{cover_enabled, true}`时，执行`ct`，`eunit`命令会自动生成覆盖率数据，也可以在测试命令中添加相应`cover`标志以执行覆盖率分析。

将生成一份HTML格式报告。

Option | Type | Description
-----   |  ----- | -----
--reset/-r | none | 重置所有覆盖率数据
--verbose/-v | none | 在控制台打印覆盖率分析
