# 游戏对象框架


## 概述

游戏对象管理一般是通过总分结构，管理器为RoleMgr，对象为Role，Role的功能由各种功能组件实现，从而实现继承和解耦。

<br/>

## RoleMgr

管理类管理游戏中所有的Role，负责Role的创建、查找、销毁，一般配合UID生成器，给每一个Role分配一个唯一ID作为唯一的标识。

<br/>

## Role

Role是游戏中的对象，可以是角色，特效，建筑等等游戏中的一切实体，由BaseRole派生。

<br/>

## BaseRole

基类，定义Role的最基本信息，如GameObject，Transform信息，显示隐藏状态等等。

<br/>

## 派生

对BaseRole进行派生就是具体的游戏对象，如添加了控制器以后就是角色，再添加主动移动的控制器以后就是主角，通过绑定不同的组件实现Role的不同方向功能。

<br/>

## Role的功能

Role的功能都由挂载的组件实现，常用的组件有

- 模型组件
- 被动移动组件
- 主动移动组件
- 显示隐藏组件（多条件）
- 范围触发组件（当玩家走进时会触发回调）
- 头顶文本组件
- 还有各种项目中需要实现的功能

<br/>

## Role的结构

![](https://raw.githubusercontent.com/dandkong/picgo/main/img/20221127085411.png)

