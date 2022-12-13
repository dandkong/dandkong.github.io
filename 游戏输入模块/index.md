# 游戏输入模块


<!--more-->

## 概述

输入模块是游戏中的基础功能，主要有以下功能

- 为多平台提供统一接口支持，如移动设备和PC
- 触发开始按下，按下，抬起事件
- 区分点击UI和点击场景中的物体，执行不一样的逻辑
- 支持多点触控

## 多平台

- PC上鼠标操作：`Input.GetMouseButton(index)`
- 移动设备：`Input.GetTouch(index)`

## 判断点击在UI上

主要用到EventSystem中的RaycastAll接口，参考[https://blog.csdn.net/u013012420/article/details/106999229](https://blog.csdn.net/u013012420/article/details/106999229)

```csharp
public void RaycastAll(EventSystems.PointerEventData eventData, List<RaycastResult> raycastResults);
```

## 点击流程

![](https://raw.githubusercontent.com/dandkong/picgo/main/img/20221213211206.png)

