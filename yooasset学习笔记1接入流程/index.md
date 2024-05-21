# YooAsset学习笔记1：接入流程


<!--more-->

## 前言

YooAsset是一套Unity的资源管理方案，类似Unity提供的Addressable资源系统配置好了资源规则，后面只需要用一个“寻址路径”就能透明加载资源，无需关注位于什么bundle包，以及bunlde的依赖关系，也无需关注资源包是位于本地或者远端CDN。

## 接入流程

### 安装YooAsset

参照官网教程即可，这里使用package进行安装。

### 资源配置

资源配置是YooAsset系统的亮点所在，这里用一个简单的登录界面作为例子配置一下。

首先看看文件目录，这里需要加载LoginView，LoginView里用到了test图片，test图片又被收集在图集login中。

![https://raw.githubusercontent.com/dandkong/picgo/main/img/202405162150648.png](https://raw.githubusercontent.com/dandkong/picgo/main/img/202405162150648.png)

资源配置有几个重要的概念，分别是AddressRule，CollecterType，PackRule，FilterRule

### AddressRule

可寻址规则，这是加载资源时需要传入的字符串，通过这个地址找到资源所在的包，这里使用默认完整路径加载。

### CollecterType

> MainAssetCollector 收集参与打包的主资源对象，并写入到资源清单的资源列表里（可以通过代码加载）。
> StaticAssetCollector 收集参与打包的主资源对象，但不写入到资源清单的资源列表里（无法通过代码加载）。
> DependAssetCollector 收集参与打包的依赖资源对象，但不写入到资源清单的资源列表里（无法通过代码加载）（当依赖资源没有被任何主资源引用的时候，则会在打包的时候自动剔除）。

首先看UIPrefabs，需要直接用代码加载的，设置为MainAssetCollector

再看图集Atlas，login中的所有精灵图片都被收集在图集文件中，但是不需要单独被代码加载，设置为StaticAssetCollector。

### PackRule

> PackSeparately 以文件路径作为资源包名，每个资源文件单独打包。
> PackDirectory 以文件所在的文件夹路径作为资源包名，该文件夹下所有文件打进一个资源包。PackTopDirectory 以收集器下顶级文件夹为资源包名，该文件夹下所有文件打进一个资源包。PackCollector 以收集器路径作为资源包名，收集的所有文件打进一个资源包。
> PackGroup 以分组名称作为资源包名，收集的所有文件打进一个资源包。PackRawFile 目录下的资源文件会被处理为原生资源包。

首先看UIPrefabs，prefab的上层文件夹就是一个模块，所以希望同一个模块的prefab都打到一个资源包中，使用

PackTopDirectory。

### FilterRule

> CollectAll 收集目录下的所有资源文件
> CollectScene 只收集目录下的场景文件
> CollectPrefab 只收集目录下的预制体文件
> CollectSprite 只收集目录下的精灵类型的文件

UIPrefabs设置为CollectPrefab

![https://raw.githubusercontent.com/dandkong/picgo/main/img/202405162252139.png](https://raw.githubusercontent.com/dandkong/picgo/main/img/202405162252139.png)

## 资源构建

![https://raw.githubusercontent.com/dandkong/picgo/main/img/202405162251574.png](https://raw.githubusercontent.com/dandkong/picgo/main/img/202405162251574.png)

资源构建，分别有版本号，构建模式（强制打包，增量打包...），加密类型，压缩类型，Bundle包的命名风格，以及内建资源选项。

点击打包后，会生成几种关键的文件

![https://raw.githubusercontent.com/dandkong/picgo/main/img/202405162253431.png](https://raw.githubusercontent.com/dandkong/picgo/main/img/202405162253431.png)

bundle后缀的就是资源文件，这里包含了指定打包的prefab文件，和一个依赖引用的图片bundle。

PackageManifest开头的是资源清单文件，只是格式不同，这里打开json后缀的查看一下。

![https://raw.githubusercontent.com/dandkong/picgo/main/img/202405162253724.png](https://raw.githubusercontent.com/dandkong/picgo/main/img/202405162253724.png)

可以看到记录了一些打包的信息。

第一部分是打包的基本信息。

第二部分是资源列表，加载的时候就是从资源列表去找到对应的Bundle，因为只有Prefab设置为MainAssetCollector，所以资源列表只有下面的LoginView.prefab。

可以看到这个资源处于Bundle0中，也就是下面的assets_content_ui_prefabs_login.bundle，又依赖了share_assets_content_ui_images_login.bundle即图片文件。

## 初始化和加载

> 编辑器模拟模式
>
>
> 单机运行模式
>
> 联机运行模式
>
> WebGL运行模式

这里使用单机运行模式，如果使用联机运行模式需要CDN和指定更多的初始化参数，具体参考官方教程。

```csharp
void Start()
{
    // 初始化资源系统
    YooAssets.Initialize();

    StartCoroutine(InitializeYooAsset());
}

private IEnumerator InitializeYooAsset()
{
    // 创建默认的资源包
    var package = YooAssets.CreatePackage("DefaultPackage");
    // 设置该资源包为默认的资源包，可以使用YooAssets相关加载接口加载该资源包内容。
    YooAssets.SetDefaultPackage(package);
    var initParameters = new OfflinePlayModeParameters();
    yield return package.InitializeAsync(initParameters);

    var go = YooAssets.LoadAssetSync<GameObject>("Assets/Content/UI/Prefabs/Login/LoginView.prefab").AssetObject;
    Instantiate(go);
}
```

可以看到prefab被正确加载并实例化出来了。

![https://raw.githubusercontent.com/dandkong/picgo/main/img/202405162258623.png](https://raw.githubusercontent.com/dandkong/picgo/main/img/202405162258623.png)


## 总结

可以看到YooAsset提供了一个开箱即用的Unity资源管理框架，只需要少量的配置的和代码就能接入，后续的开发基本不用再维护了，只需调用接口加载即可。

这是YooAsset框架的第一篇文章，后面的文章会剖析其源码，也是对自己学习的一个记录。

待续。。。

## 参考

[Introduce | YooAsset](https://www.yooasset.com/docs/Introduce)

[【Unity游戏开发】SpriteAtlas与AssetBundle最佳食用方案 - 马三小伙儿 - 博客园](https://www.cnblogs.com/msxh/p/14194756.html)

