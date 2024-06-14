# YooAsset学习笔记2：运行时


## 前言

上一篇文章简述了YooAsset资源管理框架的使用，接下来通过代码分析其内部原理。

## 整体框架

YooAsset框架主要分为运行时和编辑器部分，运行时实现了资源包加载，Bunlde加载，缓存，下载，从上到下的结构。编辑器部分实现了资源收集，资源构建。这篇文章主要分析运行时的机制，结构如下。

![](https://raw.githubusercontent.com/dandkong/picgo/main/img/202406142204257.jpg)

## 加载流程

运行时加载流程主要如下，加载开始以后，每一层的模块基本都是轮询实现。

![](https://raw.githubusercontent.com/dandkong/picgo/main/img/202406142204581.jpg)

简单的例子就是调用加载，从上到下，上层等待下层完成自己的步骤，如果完成，再轮询到下个阶段。

每一层根据加载模式和资源类型的不同派生到不同的对象负责轮询，下面的示例是远程加载AssetBundle的示意流程。

- 根据运行模式初始化包
- 外部调用加载
- 调用默认包的加载
- Provider轮询，等待Bundle加载完成，再从Bundle中加载具体资源
- Loader轮询，等待Bundle加载完成
- Downloader轮询，等待下载完成
- Requeseter轮询，等待请求完成

![](https://raw.githubusercontent.com/dandkong/picgo/main/img/202406142205930.jpg)

## 资源包（ResourcePackage）

资源包是一个比较上层的概念，在游戏初始化时要初始化包和其加载模式。具体的加载模式可以参照上一篇文章。

- 定义加载模式
- 维护版本
- 维护资源清单

## 资源管理器（ResourceManager）

资源管理器负责加载具体的资源，根据不同的加载模式和资源类型分成不同的Provider和Loader，并返回对应的Handle。

Provider功能：从加载好的Bundle中加载具体的资源

Loader功能：加载Bundle。

## 下载系统（DownloadSystem）

下载系统由上层的Loader持有，加载Bundle时，如果还没下载，调用下载系统进行下载。

下载器根据不同的加载模式和资源类型区分，分类普通文件和Web文件，区别在于Web文件不进行缓存。

下载功能由Requester承担，分为普通请求和断点续传。

## 缓存系统（CacheSystem）

缓存系统是在加载资源时，下载过就不需要再次下载了，直接在本地沙盒目录中读取。

缓存系统由下载系统调用，当下载完成以后，保存本地文件，并记录到缓存系统。

如果已有文件，则进行校验再进行记录。

另一个调用是在资源包初始化后，把已经下载的文件验证一遍，记录到缓存系统中。

## 后记

运行的框架就是如此，虽然功能很多，但结构清晰，使用了分层处理的思想。

下一篇将介绍编辑器的逻辑，敬请期待。

## 参考

https://github.com/tuyoogame/YooAsset

