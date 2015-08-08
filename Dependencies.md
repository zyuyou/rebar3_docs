## 依赖
> **⚠ 解决冲突**

> 跟上一个版本的 **Rebar** 不同，**Rebar3** 按照一套严格的规则来决定当获取或更新依赖时应该选择哪些依赖，哪些依赖不用改变。该套规则描述如下：

Rebar3 仅把依赖版本当作参考。鉴于目前Erlang社区的开源环境，试图强加 [semantic versioning](http://semver.org/) 或者其他类似的方案通常是没有意义的：

* 人们通常没有更新所有的版本号(git tags vs. branch names vs. OTP application versions) ，结果导致它们之间互相冲突；
* 有些人根本不会更新版本号；
* 并非所有人都认可相同的版本号方案；
* 人们使用 semantic versioning 制定版本号犯错；
* 许多应用程序都停留在小于 **1.0.0** 的版本号，因此一直被认定为是不稳定的；
* 写这篇文章的时候主要使用**Source dependencies**：每一次为了搞清楚版本冲突，都要下载所有关联的依赖以确认它们是否有冲突；

在**Rebar3**得到应用之前，任何其他格式的依赖性都将需要改变整个开源Erlang项目的开发方式。

相反，**Rebar3**是按照层级顺序获取和下载依赖包的。这意味着Rebar3会选择最接近顶层依赖树的依赖而不管其版本的。

这意味着在项目根目录下的**rebar.config**定义的依赖不会被传递依赖所覆盖，并且传递依赖也不会被后续的传递依赖所覆盖。

这也意味着，如果你有指定喜欢版本的依赖，你可以讲它添加到**rebar.config**文件中，并选择保留哪个版本。

每次获取和更新依赖后，最终的依赖列表将被记录在**rebar.lock**。

> **⚠ 冲突即错误**

> 如果你想**Rebar3**检测到依赖冲突后马上终止编译，而不是跳过该文件继续处理其他，你有可以添加`{deps_error_on_conflict, true}`到**rebar.config**文件中去。

## 声明依赖

依赖关系可以在项目根目录的**rebar.config**文件中声明，并且使用命令`rebar3 deps`进行检查。

一般来说，**Rebar3** 支持两种类型的依赖：
* Source dependencies （源依赖）
* Package dependencies（包依赖）

这两种类型的依赖的处理方式略有不同（在下载它们之前，Package比Source提供更详细的信息 ），但是它们具有相同的作用。

所有的依赖都是基于当前项目的。这对于避免因其与全局库的版本冲突引发的问题非常有用。这也有助于Erlang的发行机制以构造独立的软件系统。

依赖可以使用以下任何格式：

```erlang
{deps,[
  %% Packages
  rebar,
  {rebar,"1.0.0"},
  %% Source Dependencies
  {rebar, {git, "git://github.com/rebar/rebar3.git"}},
  {rebar, {git, "https://github.com/rebar/rebar3.git"}},
  {rebar, {hg, "https://othersite.com/rebar/rebar3"}},
  {rebar, {git, "git://github.com/rebar/rebar3.git", {ref, "aef728"}}},
  {rebar, {git, "git://github.com/rebar/rebar3.git", {branch, "master"}}},
  {rebar, {git, "git://github.com/rebar/rebar3.git", {tag, "3.0.0"}}},
  %% Legacy support -- added parts are ignored
  {rebar, "3.*", {git,"git://github.com/rebar/rebar3.git"}},
  {rebar, {git, "git://github.com/rebar/rebar3.git"}, [raw]},
  {rebar, "3.*", {git, "git://github.com/rebar/rebar3.git"}, [raw]}
]}.
```

如上述例子所示，在当前版本中，只支持**packages**，**git sources** 和 **mercurial sources**。若要使用自定义的依赖源，可以为**Rebar3**添加一个实现了[Resource Behaviour](www.rebar3.org/v3.0/docs/custom-dep-resources)的插件。

**Rebar3**将会获取所有需要的依赖。但是，为了让 **Erlang/OTP** 能够启动和关闭相应的应用，你应该将相应的依赖添加到`app`文件中：
```erlang
{application, <APPNAME>,
 [{description, ""},
  {vsn, "<APPVSN>"},
  {registered, []},
  {modules, []},
  {applications, [kernel
                 ,stdlib
                 ,cowboy
                 ]},
  {mod, {<APPNAME>_app, []}},
  {env, []}
 ]}.
 ```

如果需要支持更多的依赖声明格式，可以使用[`rebar_resource` 行为](https://github.com/rebar/rebar3/blob/master/src/rebar_resource.erl) 来扩展**Rebar3**，并创建一个[pull request](https://github.com/rebar/rebar3/blob/master/CONTRIBUTING.md)给维护者们。

## Source Dependencies (源依赖)

源依赖将按照层次顺序被下载 -- 从依赖树的顶层逐层往下。当一个依赖被发现并下载下来，其他拥有相同名字的版本将被忽略，并且给出警告显示。

对于一个常规的依赖关系树，如：

```
  A
 / \
B   C 
```

依赖`A`，`B`和`C`将被获取。

然后，对于更复杂的树，如：

```
   A
 /   \
B    C1
|
C2
```

依赖 `A`, `B` 和 `C1` 将被获取， 而 **Rebar3**并不会获取依赖``C2``, 而是显示警告： `Skipping C2 (from $SOURCE) as an app of the same name has already been fetched`。

这样的提示信息应该可以让开发者知道哪个依赖被跳过了吧。

那么如果两个有相同名字的依赖是在相同层级的情况呢？

```
   A
 /   \
B     C
|     |
D1    D2
```

这种情况下，`D1`将取代`D2`,因为`B`按字典顺序排在`C`前面。这是一条比较随意的规则，但是起码可以避免重复的依赖。

如果开发者不希望这种结果，可以将`D2`添加到顶层配置中以确保它能被尽早获取：

```
   A      D2
 /   \
B     C
|     |
D1    D2
```

这将获取`A`，`B`，`C` 和 `D2`。

对于包依赖，**Rebar3** 也是采用相同的规则，并且也会检查循环依赖和错误。然而，源依赖总是优先于包依赖。

在`_checkouts`目录中依赖将保持不变。

## Package Manager Dependencies（包管理依赖）

**Rebar3** 使用 [hex.pm](https://hex.pm) 管理依赖包，你可以使用下述命令获取包列表：

```shell
$ rebar3 update
===> Updating package index...
$ rebar3 pkgs
```

包依赖可以使用下述简单的格式声明：

```erlang
{deps, [
        {cowboy, "1.0.0"}
       ]
}.
```

包管理器将基于依赖关系图和拓扑排序获取最小的一组依赖包。并在如果有冲突的情况下像源依赖那样确定最终的依赖包和显示警告。

主要的区别在于包依赖是预先处理好的数据集合，所以要比源依赖快很多。

## 检出依赖 

如果要处理只留在本地的依赖，你可以使用 `_checkouts` 目录，将需要的依赖建立连接到该目录或者直接拷贝过来。任何位于`_checkouts`的应用的优先级都高于已经声明在`rebar.config`文件中或者已经获取并下载到`_build`目录的。

```
_checkouts
└── depA
    └── src
```

## 升级源依赖

每当一个依赖被获取下载并且加锁后，**Rebar3**会提取一个源文件的引用，并且给其指定一个版本。在以后的构建中该依赖都要严格遵循该版本。

**Rebar3** 可以升级旧版本的依赖包。 这里可以采用两种方式: 
* forwarding a branch's reference to its latest version (e.g. an old `master` up to the new `master`'s `HEAD`)
* crushing an existing dependency with a new version from the `rebar.config` file.

**Rebar3** 只允许升级顶层的依赖包; 因为升级传递依赖包没有多大意义 -- 如果需要升级此类依赖包，应该将其移至顶层配置中。

如下依赖树：

```
A  B
|  |
C  D
```

开发者可以升级单一依赖 (`rebar3 upgrade A` and `rebar3 upgrade B`) 或者同时升级 (`rebar3 upgrade A,B` 或者使用 `rebar3 upgrade` 升级所有).

只升级 `A` 意味着 `A` 和 `C` 将被升级。`B`和`D`的升级将被忽略。

升级源依赖是危险, 不过也有比较有趣的情况。 如下依赖树：

```
    A       B       C1
   / \     / \     / \
  D   E   F   G   H   I2
  |   |
  J   K
  |
  I1
```

在获取上述依赖树之后, `I2`将优先于`I1`被获取。 然而，将 `C1` 升级到 `C2`之后，`C2` 不再需要`I2`, **Rebar3** 将自动获取`A`依赖树中的`I1`（尽管`A`没有升级）以纠正整个依赖树：

```
    A       B     C2
   / \     / \    |
  D   E   F   G   H
  |   |
  J   K
  |
  I1
```

项目不再需要`I2`，但是需要`I1`

## 依赖锁管理

可以使用`rebar3 deps`检查依赖锁或者依赖的状态：

```
→ rebar3 deps
cowboy* (package)
recon* (git source)
erlware_commons (locked git source)
getopt* (locked git source)
providers (locked hg source)
relx (locked git source)
```

依赖包会与锁文件的记录进行对比并显示它们的状态。当前的依赖如果与锁文件的记录不符的话会使用星号（`*`）标出。

如果一个已经不再需要的依赖存在于锁文件记录中，可以使用`rebar3 unlock <app> (rebar3 unlock <app1>,<app2>,...,<app3> for many apps)`解锁它们.

执行 `rebar3 unlock` 将会清空整个锁文件。
