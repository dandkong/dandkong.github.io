# YooAsset学习笔记3：运行时



## 前言

上一篇文章简述了YooAsset资源管理框架运行时的框架的使用，接下来通过代码分析其内部原理。

## 概览

YooAsset框架的编辑器部分包含了四个部分。

- 收集：通过过滤规则，资源包名，定位地址，划分打包资源。
- 构建：根据收集到的资源，同时收集其依赖资源，打包成Bundle包。
- Reporter：打包后的资源报告。
- Debugger：运行时的资源加载情况。

## 收集

![](https://raw.githubusercontent.com/dandkong/picgo/main/img/202406302149365.jpg)

收集器主要通过过滤规则，资源包名，定位地址来划分资源，具体的用法在使用文档有详细介绍，本系列的第一篇也有介绍。

## 构建

![](https://raw.githubusercontent.com/dandkong/picgo/main/img/202406302150989.jpg)

资源构建也是和运行时的设计相似，按步骤构建，流程如下图所示。

构建管线分为内置管线，可编程管线，原生管线，内置管线和可编程管线的构建步骤基本一致，调用的API有不同。原生管线就是直接复制文件，不构建Bundle。

## 总结

编辑器的框架就是如此，先收集再按步骤构建，再配套Reporter和Debugger进行排查分析。

本系列教程到此结束，感谢阅读。

## **参考**

https://github.com/tuyoogame/YooAsset

