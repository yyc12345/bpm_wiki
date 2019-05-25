# bpm网络协议手册

本页面旨在介绍bpm在网络上传输包的相关协议。虽然bpm本体已将这些协议的细节抹除，只留下一个接口，但是，如果您想了解bpm的内部构造或者理解bpm的*辣鸡*代码并帮助我们提高它的效率，此文档会变得很有用。

如果您想开发一个新的bpm，并希望接入现有协议，此文档也可以给您的网络部分的设计进行一些指导。

bpm分为以下几个修改了协议的版本：

* Project-alice-in-wonderland（Protocol version 1）
* Project-sakura（Protocol version 2）

本文档描述的是`Project-sakura`版本的协议，协议版本2。

## 基本过程

传输的过程分为：

1. 客户端向服务器提出下载请求
2. 服务器根据结果向客户端返回数据，确认资源是否存在以及资源的分片个数
3. 客户端根据响应的结果，如果出现差错，则取消，如果存在，下一步
4. 客户端根据服务器的资源分片个数，依次下载各个分片
5. 客户端下载完所有分片后，通知服务器，此后服务器释放相关资源

## 下载请求

### 格式

|标识符|类型|
|:---:|:---:|
|SIGN|Int32 (4 byte)|
|TRANSPORT_VER|Int32 (4 byte)|
|Verification code|Int32 (4 byte)|
|Resources type|Int32 (4 byte)|
|Package name length|Int32 (4 byte)|
|Package name|Variable|

### 注释

* `SIGN`是标识头，以确认这是一个合法的bpm请求，此数值定义在`ShareLib.Transport.SIGN`，当前数值为`61`
* `TRANSPORT_VER`标识当前传输协议的版本号，当传输协议有不向下兼容的情况的时候，此数值变大，对于传输协议不相符的请求，服务器会拒绝请求。此数值定义在`ShareLib.Transport.TRANSPORT_VER`，当前数值为`1`
* `Verification code`用于验证后续服务器发来的包是否是对此请求的响应，此数值在发出时通过随机数产生
* `Resources type`指定此请求时请求何种文件，此枚举定义在`ShareLib.PackageType`，在进行Int的强制类型转换后发出。查看代码以了解各个枚举对应的数值。
* `Resources type`中，`PackageDatabase`指下载`package.db`；`PackageInfo`指下载包的依赖文件，也就是JSON文件；而`Package`即下载包的本体，ZIP文件。
* `Package name length`指示`Package name`的长度
* `Package name`标识要下载的包名，此包名需要带上版本号。文本使用UTF8编码。
* 当`Resources type`取`PackageDatabase`时，**不要**传输`Package name length`和`Package name`字段。因为`package.db`是唯一的，无需指定。

## 回应资源状态

### 格式

|标识符|类型|
|:---:|:---:|
|Version check|Bool (1 byte)|
|Verification check|Int32 (4 byte)|
|Package check|Bool (1 byte)|
|Segment count|Int32 (4 byte)|
|Signature|byte\[\] (256 bit)|

### 注释

* `Version check`指示传输协议版本号的检查情况，返回`true`标识检查通过，否则不通过，不通过之后，连接将会被断开。
* `Verification check`，即前文发送给服务器的`Verification code`，此步请客户端验证匹配。匹配不通过请客户端断开连接
* `Package check`检查您希望获取的包是否存在。如果不存在将返回`false`并断开连接
* `Segment count`指示你希望获取的文件有多少个分片，只有上述检查都通过了，这个值才能被获取
* `Signature`表示需要用于签名验证的数据，当且仅当传输`package.db`时此字段才存在
* 在`Project-sakura`后，bpm引入了签名验证机制，即服务器会对`package.db`进行签名，以规避中间人攻击。bpm不会对包数据签名，因为包数据的HASH计算结果被记录在`package.db`中，只要确信`package.db`来源，即可保证所有内容都是正确的
* 当你下载`package.db`时，在检查完`Segment count`后，如果没任何问题，此时服务端应当已经将你所请求的资源加入资源管理列表并等待传输，但若此时你在检查`Signature`时发现签名有问题，希望断开链接结束传输，则应当根据后文的 **下载结束释放资源** 章节的方法来关闭链接而不是直接关闭链接，否则服务器资源无法释放，同时，这个断开操作应当由客户端执行，因为服务端不会了解到你识别失败了。

## 下载分片

下载分片的基本原理是客户端按分片前后顺序依次向服务器发出下载某个分片的请求，然后服务器将数据传输回来，直到下载完毕。

顺序是客户端先发请求，然后等待接受数据，接收完毕后再发下一个分片请求，以此类推。

### 客户端分片请求格式

|标识符|类型|
|:---:|:---:|
|Index|Int32 (4 byte)|

### 服务器发送分片格式

|标识符|类型|
|:---:|:---:|
|Segment size|Int32 (4 byte)|
|Segment data|Variable|

### 注释

* `Index`是从`1`开始计数，而不是传统意义上的从`0`开始
* `Segment size`指示`Segment data`的长度
* `Segment data`是文件本体，在获得后无需过多操作，直接按顺序写入文件即可

## 下载结束释放资源

理论上，分片下载是可以乱序下载的，并可以加上重传机制。因此服务器不会计数以自动关闭资源，需要由客户端手动命令服务器关闭连接释放资源。

使用 **客户端分片请求格式** 并指定`Index`为`0`即可标识客户端下载结束，并使服务器释放资源。
