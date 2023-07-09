# Unity开发性能优化总结


<!--more-->



# Unity开发性能优化总结

## 1. 前言

工作几年，大部分的时间还是在业务开发，但是业务开发中也是要注意性能，如果不注意性能，积少成多可能也会导致游戏性能的下降。下面总结下自己理解到的性能注意项。

## 2. 编程和代码架构

### 2.1 lua使用的性能注意

- 交互原理：取值、入栈、c#到lua的类型转换，还有内存分配和GC。
- lua与C#交互，做好缓存，减少如gameobject.transform.position.x之类的操作，可以在C#封装成GameObejct.GetPosX()
- 在lua中引用c#的object，代价昂贵
- lua和c#之间传参、返回时，尽可能不要传递Vector3/Quaternion等unity值类型，数组等
- 参见：[Unity+Lua性能优化（Lua和C#交互篇）-腾讯游戏学堂 (tencent.com)](https://gwb.tencent.com/community/detail/125117)
- lua自身使用，减少全局表的查找，减少全局变量
- lua与C#交互原理：[wlgys8/UnityLuaInteractDemo: Unity中c#与lua的交互原理和用例 (github.com)](https://github.com/wlgys8/UnityLuaInteractDemo)

### 2.2 减少字符串的使用

- 字符串使用性能较差，主要是在传值或者操作时都会有临时变量的产生，引发GC。
- 代码中减少字符串的拼接，拆分等操作，在Update中尤其需要注意。
- 使用hash替代字符串传值，参考Unity组件Animator、Material、Shader实现，对于固定的string，用一个字典存下所有字符串，统一转成hash再调用。

### 2.3 缓存 GameObjects 和组件

- GameObject.Find、GameObject.GetComponent 和 Camera.main都会有较大的性能负担，尽量只在初始化时调用，缓存下来使用。

### 2.4 对象池

- 对象池包括类和GameObject，使用对象池可减少了实例化和销毁带来的性能消耗，减少托管内存分配的次数、防止产生垃圾回收。
- 注意对象池的缓存上限，没有上限的话对内存会有负担。

### 2.5 使用正确的数据结构

- 注意list，dict等的使用，查找数据的时候尽量减少遍历，可以使用字典查找，降低复杂度。

### 2.6 减少每帧的代码量

- Update每帧执行，不一定所有的代码调用都需要在里面调用。
- 对于需要定时执行的逻辑也不一定要每帧执行，如倒计时，可以每秒执行，定期执行。

### 2.7 其他

- 减少日志的打印，日志打印同样会有字符串的消耗，封装好的打印方法还会进行本地的写法，无用的日志就不要打印了。

## 3. UI优化手段

### 3.1 减少渲染的量级和少内存

- 避免过重UI，拆分成小UI，动态加载
- 需要响应Raycast事件时，不要使用空Image，可以自定义组件继承自MaskableGraphic，重写OnPopulateMesh把网格清空，这样可以响应Raycast而又不需要绘制Mesh

### 3.2 减少OverDraw

- 减少UI重叠
- 全屏UI隐藏后面的UI
- 关掉场景相机的渲染
- 慎用Mask组件（[浅谈Unity uGUI Mask组件实现原理 (rugbbyli.gitlab.io)](https://rugbbyli.gitlab.io/blog/post/2017-12-07-unity-stencil/),[Unity中的Mask组件增加DrawCall的原因_unity mask组件_流浪打工人的博客-CSDN博客](https://blog.csdn.net/qq_41841073/article/details/128336434)）
- 慎用Shadow与Outline组件，顶点过多（[慎用Outline ,UGUI Outline实现原理分析__Captain的博客-CSDN博客](https://blog.csdn.net/huutu/article/details/46389141)）
- 简单的粒子特效

### 3.3 图集

- 不同的图集/材质会产生DrawCall，且会引起加载问题

### 3.4 其他

- 避免频繁调用SetActive方法，会导致网格重建，会导致内存分配，触发GC，可以设置layer或把渲染移到相机外

### 3.5 参考资料

- [UGUI性能优化总结 | 无境 (drflower.top)](https://www.drflower.top/posts/aad79bf1/#Overdraw)
- [浅谈UGUI的渲染机制 - cancantrbl - 博客园 (cnblogs.com)](https://www.cnblogs.com/cancantrbl/p/16076748.html)


