# Lua知识点总结


<!--more-->

## 1. 元表和元方法

**__index元方法**

```csharp
> other = { foo = 3 }
> t = setmetatable({}, { __index = other })
> t.foo
3
> t.bar
nil
mytable = setmetatable({key1 = "value1"}, {
  __index = function(mytable, key)
    if key == "key2" then
      return "metatablevalue"
    else
      return nil
    end
  end
})
```

**__nexindex方法**

```lua
--没有的减赋值到元表中
mymetatable = {}
mytable = setmetatable({key1 = "value1"}, { __newindex = mymetatable })

print(mytable.key1)

mytable.newkey = "新值2"
print(mytable.newkey,mymetatable.newkey)

mytable.key1 = "新值1"
print(mytable.key1,mymetatable.key1)
mytable = setmetatable({key1 = "value1"}, {
    __newindex = function(mytable, key, value)
        rawset(mytable, key, "\\""..value.."\\"")
				--如果这里写成mytable[key]=value，会递归了
    end
})

mytable.key1 = "new value"
mytable.key2 = 4

print(mytable.key1,mytable.key2)
```

**rawget和rawset**

raw方法就是忽略table对应的metatable，绕过metatable的行为约束，强制对原始表进行一次原始的操作，也就是一次不考虑元表的简单更新。

## 2. lua面向对象实现

使用元表，__index父类，并在new方法返回一个新的table

```lua
function class(classname, super)
    local superType = type(super)
    local cls
        -- inherited from Lua Object
     if super then
            cls = {}
            setmetatable(cls, {__index = super})
            cls.super = super
        else
            cls = {ctor = function() end}
        end

        cls.__cname = classname
        cls.__ctype = 2 -- lua
        cls.__index = cls

        function cls.new(...)
            local instance = setmetatable({}, cls)
            instance.class = cls
            instance:ctor(...)
            return instance
        end
    return cls
end
```

## 3. Table实现原理

- 数组和哈希的混合结构

## 4. 游戏常用方案

XLua ， ToLua ， ULua都只是提供了 C# 与 Lua 的互相调用机制，差别基本就是少量的语法差别。座位C#和lua的中间层，把C#生成Wrap代码，经过xlua调用Lua代码。

### Lua调用C#

- Lua-C dll- C# Wrap-C#
- 通过ObjectTranslator获取C#中的实例
- Wrap方式：非反射机制，需要为源文件生成相应的wrap文件，当启动Lua虚拟机时，Wrap文件将会被自动注册到Lua虚拟机中，之后，Lua文件将能够识别和调用C#源文件
- 总结：Lua调用Wrap文件， Wrap文件调用C#源文件

### 参考

[【ToLua】C#和Lua的交互细节](https://zhuanlan.zhihu.com/p/109198841)

[【Unity游戏开发】tolua之wrap文件的原理与使用 - 马三小伙儿 - 博客园](https://www.cnblogs.com/msxh/p/9813147.html)



## 5. LuaGC

### 原理

- 标记，清楚
- 三色标记

### 参考

[Lua GC机制分析与理解-上](https://www.zhihu.com/tardis/zm/art/133939450?source_id=1003)

## 6. 参考

[Lua知识点整理](https://www.drflower.top/posts/43f53d35/)

