# STL总结


<!--more-->

## 分类

### Vector动态数组

**原理：**连续的内存，start,end,max，max代表容量，end代表当前最大

**扩容流程：**申请空间，复制，清理

**访问方式**

```lua
//方式一：单个访问，假设num数组中已经有了5个元素
cout << num[4] << "\\n";  //输出第五个数据
//一二维可变数组和普通数组的访问方法一样

//方式二:遍历
for(int i = 0; i < num.size(); i++)
	cout << num[i] << " ";//下标范围在[0,num.size()),前开后闭

//方式三：智能指针
for(auto i : num)
	cout << i << " ";
```

### **list**

**概述：**较快增删查

**原理：**双向链表

### **stack栈**

**概述：**先进后出

**原理：**底层一般用list或deque实现，封闭头部即可，不用vector的原因应该是容量大小有限制，扩容耗时

### **queue队列**

**概述：**先进先出

**原理：**底层一般用list或deque实现，封闭头部即可，不用vector的原因应该是容量大小有限制，扩容耗时

### **deque双端队列**

**概述：**首位都能操作

**原理：**deque 容器的分段存储结构，提高了在序列两端添加或删除元素的效率，但也使该容器迭代器的底层实现变得更复杂。

### **priority_queue优先队列**

**概述：**优先队列是在正常队列的基础上加了优先级，保证每次的队首元素都是优先级最大的。

**原理：**priority_queue的底层数据结构一般为vector为底层容器，堆heap为处理规则来管理底层容器

### **set和multiset集合**

**概述：**set和multiset会根据特定的排序准则，自动将元素进行排序。不同的是后者允许元素重复而前者不允许

**原理：**底层红黑树，平衡二叉树可以保证在最坏情况下仍然具有较高的搜索效率，时间复杂度为O(logn)。

### **map和multimap**

**原理：**底层红黑树

### **hash_set和hash_multiset**

**原理：**哈希

### **hashmap/unordered_map和hash_multimap**

**原理：**哈希

## 注意

### **什么时候需要用hash_map，什么时候需要用map?**

- 总体来说，hash_map 查找速度会比map快，而且查找速度基本和数据量大小无关，属于常数级别;而map的查找速度是log(n)级别。并不一定常数就比log(n) 小，hash还有hash函数的耗时，明白了吧，如果你考虑效率，特别是在元素达到一定数量级时，考虑考虑hash_map。但若你对内存使用特别严格，希望程序尽可能少消耗内存，那么一定要小心，hash_map可能会让你陷入尴尬，特别是当你的hash_map对象特别多时，你就更无法控制了，而且hash_map的构造速度较慢。
- 现在知道如何选择了吗？权衡三个因素: 查找速度, 数据量, 内存使用。

## 参考

[C++ STL deque容器底层实现原理（深度剖析）](http://c.biancheng.net/view/6908.html)

[C++ STL中哈希表 hash_map从头到尾详细介绍_c++ 哈希表_susandebug的博客-CSDN博客](https://blog.csdn.net/u010025211/article/details/46653519)

[C++  STL 几个容器的底层实现 收藏一下_stl c++底层实现_孤鸿踏雪的博客-CSDN博客](https://blog.csdn.net/single_wolf_wolf/article/details/52854015)

[红黑树（图解+秒懂+史上最全） - 疯狂创客圈 - 博客园](https://www.cnblogs.com/crazymakercircle/p/16320430.html)

