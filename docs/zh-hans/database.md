# 数据库指南

!!! warning "未完成的篇目"
    本篇目尚未完成，请谨慎阅读

本篇目介绍bpm中使用的各类数据库以及其内部字段的含义。

bpm使用SQLite作为数据库驱动程序。bpm的数据库分为两部分，一个是`package.db`，此数据库会被服务器进行分发，其记录了各个包的相关数据，另一个则是只存在于客户端本地的`installed.db`，其记录了本地安装的包的依赖关系及存储的包的设置。

## package.db

`package.db`包含了两个表，`package`表记录了存在的包，而`version`表记录了所有的包的所有版本的详细信息。

## package表

```SQL
CREATE TABLE package (
    name   TEXT    NOT NULL
                   CONSTRAINT PK_package PRIMARY KEY,
    aka    TEXT    DEFAULT '',
    type   INTEGER NOT NULL,
    desc   TEXT    DEFAULT ''
);
```

## version表

```SQL
CREATE TABLE version (
    name               TEXT    NOT NULL
                               CONSTRAINT PK_version PRIMARY KEY,
    parent             TEXT    NOT NULL,
    additional_desc    TEXT    DEFAULT '',
    timestamp          INTEGER NOT NULL,
    suit_os            INTEGER NOT NULL,
    dependency         TEXT    DEFAULT '',
    reverse_conflict   INTEGER NOT NULL,
    conflict           TEXT    DEFAULT '',
    require_decompress TEXT    NOT NULL,
    internal_script    TEXT    DEFAULT '',
    hash               TEXT    NOT NULL
);
```

### timestamp

一个UNIX时间戳，不包含毫秒，长度为Int64，用于指示当前包的添加时间（在bpm内部被用于判断包的新旧）

### suit_os

表示适用的操作系统，通常用于App类型的包用于判断当前包是否可以被安在此操作系统上。此数值使用`[Flags]`标识的`enum`构成，详细构成请查询`ShareLib/Package.cs`。

### dependency

表示此包需要的依赖，即，没有此列表中的包，此包无法正常运行。可以填写为一个包名，例如：`999_sections_loader`，则表明此包可以以指定包的任何版本为依赖。也可以指定为`包@版本`格式，例如：`999_sections_loader@v_gamepiaynmo`，则表明此包只能以指定包的指定版本为依赖。

依赖可以有多个，多个依赖之间使用`,`分割即可

### reverse_conflict

一个Bool值，用于指示conflict的含义。如果为false，则表示conflict中存储的是与此包不兼容，有冲突的包。如果指定true，则表示反转conflict的含义，即此包只与conflict中所书写的包兼容。如果指定true，默认是兼容自己（此包的此版本）的，如果此时conflict中未填写任何包的话，则表明此包与所有包（除自己）都不兼容。

### conflict

原意为表示与此包冲突的包，同理，可以填写为包名或`包@版本`格式。其具体含义由`reverseConflict`指示。同样，其可以有多个，多个包之间使用`,`分割即可

### require_decompress

表示是否需要解压缩，即当前包是否是单文件包，单文件包是无需解压的。

对于压缩包，此字段填空字符串（不是`NULL`）。对于非压缩包，此字段指示该文件下载后应该被命名为何种名称，例如对于一个地图，你可以填入`MapNameInPackage.nmo`，这样它就会被自动命名为`MapNameInPackage.nmo`（这个名称是内置脚本`Map`所需要使用的名称）；对于那种只有单个脚本的包，可以填入`Setup.cs`，它就会被自动命名为`Setup.cs`，这样bpm就可以在后续过程中识别到这个脚本。

### internal_script

表示是否需要使用内置脚本。

无需使用内置脚本时，此字段填空字符串（不是`NULL`）。如若使用，需要填写`Map`，`Sky`，`Texture`，`SoundEffect`，`BGM`其中之一即可。

### hash

表示当前包文件的摘要，使用`SHA256`进行摘要并使用Base64进行格式化后再存储到此字段

## installed.db

```SQL
CREATE TABLE installed (
    name            TEXT    PRIMARY KEY
                            NOT NULL,
    reference_count INTEGER NOT NULL,
    reference       TEXT    DEFAULT ''
    data            TEXT    DEFAULT ''
);
```
