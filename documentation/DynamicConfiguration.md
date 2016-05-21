## 动态配置
覆盖像`rebar.config`和`*.app.src`那样的变量文件,你可以使用基于`file:script/2`的动态配置.

如果`<name>.script`的文件跟`<name>`的文件在同一个目录下(例如`rebar.config`和`rebar.config.script`), 后缀为`script`的文件将被解析执行,其结果被当作新的配置.

为了方便起见, 有两个绑定变量在解析执行文件过程中可以使用:

- `CONFIG` - 如果后缀为`script`文件存在无后缀的同名文件, 那么该值是无后缀文件`file:consult/1`的结果, 否则该值为 `[]`.

- `SCRIPT` - 后缀为`script`的文件名.

任何情况下, 后缀为`script`脚本文件返回的结果(文件中最后执行解析的结果)必须跟跟无`script`后缀的源文件保持相同内容格式. 例如, `rebar.config.script` 需要返回跟`rebar.config`一样的内容数据; 如果是`<app-name>.app.src.script`需要返回Erlang应用元数据格式内容.

## Simple Example
如果你正在构建相当复杂的系统,那么每次都从`github`获取依赖应用可能拖慢你的开发进度. 从本地获取依赖可能是一个要快好多的选择, 但是你不想修改`rebar.config`配置,也不想遭受因此带来的冲突困扰.

那么下面的`rebar.config.script`就非常有必要放在你的应用目录下:

```Erlang
case os:getenv("REBAR_DEPS") of
    false -> CONFIG; % env var not defined
    []    -> CONFIG; % env var set to empty string
    Dir ->
  lists:keystore(deps_dir, 1, CONFIG, {deps_dir, Dir})
end.
```

当你想'适当'地构建你的系统时,只要简单地调用`unset REBAR_DEPS`(其他等效命令),再清理构建即可.

注意`file:script/2`跟`file:consult/1`不同的地方在于只有最后一个表达式执行结果将被返回. 因此你要注意你返回的配置项元列表(list of config terms). 但在那之前, 你可以做任何IO操作(包括网络), 检查系统环境变量, 读写文件, 也可以调用`file:script/2`去解析其他文件. 几乎所有OTP库你都可以用. 跟`file:eval/2`一样, 每个表达式都以英文句号结束.