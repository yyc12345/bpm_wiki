# 服务器手册

本手册目的在于介绍BPMServer的使用方法。BPMServer，也就是bpm的服务器本体。

我已经搭建了一个bpm下载服务器并面向全球提供下载，那么为什么还要这个手册？因为我的服务器在中国大陆，对于一些地区的访问速度可能并不是那么友好，我们可能需要在全球多部署几个服务器，以达到最好的下载速度，但是这些服务器不可能都由我一个人来部署，故有此手册，介绍如何搭建一个bpm服务器。

此外，如果您不喜欢我的服务器，您也可以利用此手册搭建一个私有bpm服务器以为您的Ballance的圈子做专门服务。这也是这个手册存在的原因之一。

服务器是免费开源的，您可以在bpm的Github项目的Release页面中找到下载链接。

BPMServer支持在任何可以运行.Net Core的平台上运行，您需要[点击这里](https://dotnet.microsoft.com/download)前往.Net Core下载网址，并配置好。

BPMServer是一个C/S应用，使用TCP流模式传输数据。因此你无需配置Web服务，只需要一个空闲端口，BPMServer就能跑起来。

在目前版本中，服务器本体BPMServer已和BPMSMaintenance合并，并在一起管理。（之前版本是分离的）

我们已删除BPMSDaemon，请使用 `bash`， `cmd` 或 `PowerShell` 相关命令来维持程序持续运行。（服务器偶尔会崩溃）

## 综述

服务器有两种工作模式：`Running`和`Maintain`，每个模式下只能执行对应模式的指令

`Running`标识服务器正在运行，可以接受连接请求，分发数据。`Maintain`标识服务器正在维护，在外界看来就是服务器断开了，此时可以执行添加包，删除包等操作

在任何情况下，服务器并不会主动给你输入命令的机会，输入命令的方式是在BPMServer中按下Tab键以打开命令输入栏，然后再输入命令执行

服务器开启后将自动进入`Running`模式，您可以使用`mode`指令切换模式。

## 启动服务器

下载可执行文件后，您可以直接运行`dotnet BPMServer.dll`，服务端程序将会自动创建2个必须目录：`package` 与 `dependency`，并初始化一个空的数据库`package.db`。并自动生成一个默认配置文件 `config.cfg`，用于决定监听端口：

```json
{
    "IPv4Port": "6161",
    "IPv6Port": "8181"
}
```

如果您不想使用这些端口，您可以在运行BPMServer之前提前配置好这个文件。或者是启动之后使用 `config`命令配置并重启BPMServer

## 关闭服务器

### Running 模式

执行`exit`命令将会立即拒绝任何新的连接请求并等待当前连接请求结束数据传输后退出程序。

当遇到紧急情况时，例如有人攻击了你的服务器，或者有人恶意大量获取你的数据导致服务器无法正常响应相关服务。可以执行`crash`命令来立即结束程序。此操作将会立即关闭程序，不等待任何资源的释放。

### Maintain 模式

执行`exit`命令将会保存当前对数据库的修改并退出程序

执行`crash`命令将会**丢弃**任何对数据库的修改，**保留**任何资源文件复制的修改，然后退出程序。**强烈不建议**在`Maintain`模式下使用此命令

## 服务器同步

同步？你在说啥？这破游戏流量那么少还需要分布式服务器？还需要同步？

好吧，如果流量变多的话我会搞个同步命令的。与互联网上指定计算机上的另一个 BPMServer 进行同步

## 命令注释

### 维护命令

* `addpkg NAME AKA TYPE VERSION DESCRIPTION`：添加包
* `addver NAME VERSION`：添加一个新版本到当前存在的包
* `delpkg NAME`：删除包
* `delver NAME VERSION`：删除当前存在的包的一个版本

#### 注意事项

* 添加包/版本时，需要将匹配的zip文件命名为`new_package.zip`，将匹配的json文件命名为`new_package.json`，并都置于BPMSMaintenance所在目录下，然后再执行命令
* 如果参数中需要空格，例如`description`中需要输入空格，可用`"`将参数框起来，否则会判定参数个数错误
* 如果参数为空，例如`aka`不存在，输入`""`即可
* `delpkg`不会检测存在的版本个数，会悉数删除
* `addpkg`中的`TYPE`参数应输入一个数字，请按照TYPE对照表的数据输入

#### TYPE对照表

|数值|类型|
|:---:|:---:|
|0|Mod|
|1|Map|
|2|Sky|
|3|Texture|
|4|SoundEffect|
|5|BGM|
|6|App|
|7|Miscellaneous|

### 其余命令

* `mode`
  - `mode maintain`：切换到`maintain`模式，将会先停止新的请求接受然后等待资源释放，最后打开数据库文件完成切换
  - `mode running`：切换到`running`模式，将会先保存数据库的修改，然后打开Tcp请求接受完成切换
* `exit`
  - `exit`：退出程序
* `crash`
  - `crash`：以丢失数据的状态强行退出程序
* `config`
  - `config`：列出所有设置项及其值
  - `config 设置名`：列出指定设置项的值
  - `config 设置名 新值`：修改指定设置项的值
* `client`
  - `client`：列出当前连接的客户端数值



