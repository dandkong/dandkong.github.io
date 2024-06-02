# 使用状态机初始化游戏


## 前言

最近在写游戏demo的时候，在写初始化流程的时候遇到一些问题，一开始通过回调函数来控制初始化流程，并不直观，也不好分离逻辑。进入游戏以后首先是准备好必要的资源，通过YooAssets进行加载，加载完成以后再进行游戏系统的初始化。一开始是在资源加载接口加了回调，加载完以后再进行游戏初始化，但不够优雅，如果需要更多的流程控制会写得比较乱。

最后用状态机实现游戏初始化的流程，首先就是要写一个有限状态机了，参考了一下开源的项目，这里总结一下。

## 状态机

- 管理状态：启动，状态转换，管理状态的生命周期
- 数据处理能力：增删查数据，状态中可以使用这些全局数据

## 状态

状态定义具体的逻辑，有`OnInit`,`OnEnter`,`OnUpdate`,`OnLeave`,`OnDestroy`生命周期。状态内调用状态机进行状态的切换。

## UML类图

![](https://raw.githubusercontent.com/dandkong/picgo/main/img/202406022118316.png)

## 参考

[GitHub - EllanJiang/GameFramework: This is literally a game framework, based on Unity game engine. It encapsulates commonly used game modules during development, and, to a large degree, standardises the process, enhances the development speed and ensures the product quality.](https://github.com/EllanJiang/GameFramework/tree/master)

[UniFramework/UniFramework at main · gmhevinci/UniFramework](https://github.com/gmhevinci/UniFramework/tree/main/UniFramework)

