# 数据库指南

本篇目介绍bpm中使用的各类数据库以及其内部字段的含义。

bpm使用SQLite作为数据库驱动程序。bpm的数据库分为两部分，一个是`package.db`，此数据库会被服务器进行分发，其记录了各个包的相关数据，另一个则是只存在于客户端本地的`installed.db`，其记录了本地安装的包的依赖关系及存储的包的设置。

## package.db

`package.db`包含了两个表，`package`表记录了存在的包，而`version`表记录了所有的包的所有版本的详细信息。

### package表

```SQL
CREATE TABLE package (
    name   TEXT    NOT NULL
                   DEFAULT ''
                   CONSTRAINT PK_package PRIMARY KEY,
    aka    TEXT    NOT NULL
                   DEFAULT '',
    type   INTEGER NOT NULL
                   DEFAULT 7,
    [desc] TEXT    NOT NULL
                   DEFAULT ''
);
```

#### name

包名，主键（唯一性），只是包的名称，无需带上版本号，版本是由`version`表来描述的

#### aka

指示包的别名，此字段用于I18N，即此字段可以是多种语言组合而成的，例如自杀器插件，其包名是`ball-killer`，然后`aka`字段可以输入为`自杀器`，这样以来用户就可以很轻松地根据自己所使用的语言搜索到想要的包。多个aka之间使用`,`分割。bpm在执行搜索时会同时搜索`name`和`aka`字段

#### type

包的类型，分为以下几种类型：

* Mod：用于游戏的插件
* Map：游戏地图
* Sky：游戏天空盒子背景
* Texture：游戏材质（除天空部分）
* SoundEffect：游戏音效（除BGM部分）
* BGM：游戏背景音乐
* App：与游戏有关的一些附属程序
* Miscellaneous：不属于以上任意一个

在程序内部是一个`enum`，在数据库中转换为`INTEGER`进行存储。详细数据含义可参考`ShareLib/Package.cs`文件

#### desc

有关此包的描述，通常是这个包的功能的描述，此部分内容无需I18N，纯英文即可。可以为空字符串（不是`NULL`）

### version表

```SQL
CREATE TABLE version (
    name               TEXT    NOT NULL
                               DEFAULT ''
                               CONSTRAINT PK_version PRIMARY KEY,
    parent             TEXT    NOT NULL
                               DEFAULT '',
    additional_desc    TEXT    NOT NULL
                               DEFAULT '',
    timestamp          INTEGER NOT NULL
                               DEFAULT 0,
    suit_os            INTEGER NOT NULL
                               DEFAULT 0,
    dependency         TEXT    NOT NULL
                               DEFAULT '',
    reverse_conflict   INTEGER NOT NULL
                               DEFAULT 0,
    [conflict]         TEXT    NOT NULL
                               DEFAULT '',
    require_decompress TEXT    NOT NULL
                               DEFAULT '',
    internal_script    INTEGER NOT NULL
                               DEFAULT 0,
    hash               TEXT    NOT NULL
                               DEFAULT ''
);
```

#### name

包的 **全名**，即符合`包名@版本`格式的字符串。

#### parent

指示当前包是哪个包的一个版本，即`包名@版本`格式中的`包名`部分，此字段的值必须存在于`pakcage`表中的`name`字段

#### additional_desc

对于当前包的补充说明，即对于`package`表中`desc`字段未说明的内容或者是对于当前版本的特有内容的一个补充说明。可以为空字符串（不是`NULL`）

#### timestamp

一个UNIX时间戳，不包含毫秒，长度为Int64，用于指示当前包的添加时间（在bpm内部被用于判断包的新旧）

#### suit_os

表示适用的操作系统，通常用于App类型的包用于判断当前包是否可以被安在此操作系统上。此数值使用`[Flags]`标识的`enum`构成，详细构成请查询`ShareLib/Package.cs`。

#### dependency

表示此包需要的依赖，即，没有此列表中的包，此包无法正常运行。可以填写为一个包名，例如：`999_sections_loader`，则表明此包可以以指定包的任何版本为依赖。也可以指定为`包@版本`格式，例如：`999_sections_loader@v_gamepiaynmo`，则表明此包只能以指定包的指定版本为依赖。

依赖可以有多个，多个依赖之间使用`,`分割即可

#### reverse_conflict

一个Bool值，用于指示conflict的含义。如果为false，则表示conflict中存储的是与此包不兼容，有冲突的包。如果指定true，则表示反转conflict的含义，即此包只与conflict中所书写的包兼容。如果指定true，默认是兼容自己（此包的此版本）的，如果此时conflict中未填写任何包的话，则表明此包与所有包（除自己）都不兼容。

#### conflict

原意为表示与此包冲突的包，同理，可以填写为包名或`包@版本`格式。其具体含义由`reverseConflict`指示。同样，其可以有多个，多个包之间使用`,`分割即可

#### require_decompress

表示是否需要解压缩，即当前包是否是单文件包，单文件包是无需解压的。

对于压缩包，此字段填空字符串（不是`NULL`）。对于非压缩包，此字段指示该文件下载后应该被命名为何种名称，例如对于一个地图，你可以填入`MapNameInPackage.nmo`，这样它就会被自动命名为`MapNameInPackage.nmo`（这个名称是内置脚本`Map`所需要使用的名称）；对于那种只有单个脚本的包，可以填入`Setup.cs`，它就会被自动命名为`Setup.cs`，这样bpm就可以在后续过程中识别到这个脚本。

#### internal_script

表示是否需要使用内置脚本。此字段是一个`bool`类型字段，但是在数据库中被表现为`INTEGER`类型

无需使用内置脚本时，此字段填`false`（数据库中转换为`0`），否则填写`true`（数据库中转换为`1`）。

内置脚本的执行类型根据此包在`package`表中的`type`字段来决定，同时，仅支持`Map`，`Sky`，`Texture`，`SoundEffect`，`BGM`类型的内部脚本，其余类型的均会报找不到脚本文件的错误。

#### hash

表示当前包文件的摘要，使用`SHA256`进行摘要并使用Base64进行格式化后再存储到此字段

## installed.db

`package.db`只有一个表：`installed`

### installed表

```SQL
CREATE TABLE installed (
    name            TEXT    NOT NULL
                            DEFAULT ''
                            CONSTRAINT PK_installed PRIMARY KEY,
    reference_count INTEGER NOT NULL
                            DEFAULT 0,
    reference       TEXT    NOT NULL
                            DEFAULT '',
    data            TEXT    NOT NULL
                            DEFAULT '{}'
);
```

#### name

包的 **全名**，即符合`包名@版本`格式的字符串。此包必须是已经安装在此计算机上的包

#### reference_count

对于此包的引用计数，当此数值为`0`时，表示无任何包依赖此包，可安全删除

#### reference

引用列表，其是一个数组，各个元素之间由`,`分割，元素是符合`包名@版本`格式的字符串。表示的含义即为数组内的元素依赖此包。数组元素的格式与`reference_count`相等。

#### data

当前包存储的设置类数据。此数据是在执行包脚本的时候产生的。此字段是一个JSON字符串，在程序内部转换为字典提供给脚本使用。
