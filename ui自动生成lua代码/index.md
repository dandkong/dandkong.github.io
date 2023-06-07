# UI自动生成Lua代码


<!--more-->

## 1 前言

最近回到了新项目，发现UI框架还是旧的那一套，每个UI控件，比如Text、Button等，都要手动打代码Find一遍，每次开发都会做很多无用功，所以打算优化一下效率。

设想是做一套工具，可以自动把UI代码生成出来。需求如下

- 把UI层和逻辑层的代码分开，第一次导出时才导出逻辑层，后续只导出覆盖UI层。
- 能对自动Button，Toggle等组件加上回调，由逻辑层复写
- 能控制是否导出子节点
- 能对BaseView和UIComponent分开导出
- 变量命名检测，重名自动改名
- 不影响旧代码
- 自动添加UIComponent的引用

思路

- 增加一个C#配置类（UIExportConfig），挂载在prefab上，表示需要导出成哪种代码类型（视图/组件）。
- 配置每种控件需要怎么导出，是否需要校验名字（因为有些不需要获取，比如界面中的固定图片和文本是不需要导出的）。
- 递归导出带UIExportConfig的GameObject。

## 2 UI结构

UI里面主要有两个概念，视图和组件，都由一个基类派生而来，在游戏中体现为一个单独的prefab。

![](https://raw.githubusercontent.com/dandkong/picgo/main/img/Untitled.png)

### 2.1 组件

组件UI逻辑的最小单位，比如背包中的背包页和仓库页就是两个组件，背包页和仓库页中的道具格子也是一个组件。

### 2.2 视图

视图也是组件，表现为游戏中的一个独立界面，比如一个背包界面，一个玩法界面，由自身以及嵌套的组件实现其不同功能，实现解耦和复用的目的。

## 3 实现

一个组件，对应一个prefab，会导出一个以“UI”为后缀的UI类，如下：

```lua
---BagViewUI.lua
--基础节点
self.txtTitle = self:FindCom("Text","textTitle")
self.img1 = self:FindCom("Image","textTitle/img1")
--背包页子组件
self.bagCom = BagCom.New(self:Find("BagCom")
--仓库页子组件
self.storgeCom = StorgeCom.New(self:Find("StorgeCom")
```

UI类会被逻辑类引用，如下：

```lua
---BagView.lua
self.ui = BagViewUI.New(self.gameObject)
self.ui.txtTitle.text = "背包"
self.bagCom:SetData(背包数据);
self.storgeCom :SetData(仓库数据);
```

### 3.1 导出图片，按钮，文本等组件

脚本通过深度遍历的方式遍历prefab的每个节点，对于合适的节点，生成一句查找代码，类似：

```lua
---BagViewUI.lua
--基础节点
self.txtTitle = self:FindCom("Text","textTitle")
self.img1 = self:FindCom("Image","textTitle/img1")
```

但不是所有节点都是需要导出的，网上有些做法是在节点加上组件表示要导出，我的做法是通过前缀命名来标识，每个需要导出的类型都有类型和前缀的双重验证，如果一个Text组件命名为“label”，就不会被导出。

| 组件   | 前缀 |
| ------ | ---- |
| Button | btn  |
| Image  | img  |
| Text   | txt  |

### 3.2 导出嵌套组件

如果一个组件中包含了另一个组件，会导出对应的引用比如，背包中有背包分页和仓库分页，背包分页和仓库分页又会导出他们对应的UI文件，层层嵌套解耦。

```lua
---BagViewUI.lua
--基础节点
self.txtTitle = self:FindCom("Text","textTitle")
self.img1 = self:FindCom("Image","textTitle/img1")
--背包页子组件
self.bagCom = BagCom.New(self:Find("BagCom")
--仓库页子组件
self.storgeCom = StorgeCom.New(self:Find("StorgeCom")
```

### 3.3 自动添加自定义内容

至此在写UI的时候应该不用手打Find了，但有一些重复的代码还可以进行针对性的优化，项目中主要做了，对按钮添加回调，在逻辑类中复写。这个都是可以拓展，对于不同的节点类型，生成不同的代码。

### 3.4 还能做得更好

- 可以给UI代码加入生命周期，在关闭界面以后做一些组件的清理，因为自己手动在逻辑类中清理经常会漏掉。
- 能对变体做一定的处理，如一种组件的不同样式，有没有办法可以不用复制两份一样的逻辑代码，应该应该可以用上继承的思想。

## 4 总结

- 这套方法已经用了一段时间了，确实能省下不少的重复工作，第一省去了手动Find的过程，第二是自动添加了组件的引用，让功能解耦，更好维护了。
- 项目中还配个了另一个工具，通过美术的PSD生成UI prefab，再对prefab进行一定调整，把需要用到的节点改下名，再导出UI类，就可以在逻辑类中使用了，让前端只需要关心逻辑实现，解放双手，后面研究一下里面的实现原理。

