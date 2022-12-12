# 人物移动模块


## 一、概述

游戏中角色移动分为了主动和被动，主动主要由玩家通过摇杆或者键盘进行操作，被动移动主要应用于自动寻路，通过寻路点行动。

<br>

## 二、主动移动

### 移动组件

移动组件主要用到Unity的移动组件**Character Controller**，主要属性如下

- `Height`角色的高度，通常和现实中的人物一样设置为2米左右。
- `Radius`角色的半径，用于控制人物的胖瘦。
- `Center`设置角色中心点的位置。
- `Slope Limit`限制角色能爬的最大坡度。通常设置为90度以下，这样角色就不会走到墙上。
- `Step Offset`移动步长。通常2米左右的人移动步长设置在0.1到0.4米.
- `Skin Width`皮肤厚度。如果这个值太小角色容易被卡住，太大角色容易抖动。通常将这个数据设为0.01到角色半径的10%之间。
- `Min Move Distance`最小移动距离。官方推荐把这个值设为0。
- `isGrounded`可以获取角色当前是否在地面。
- `velocity`可以获取角色当前的速度向量。

移动方法主要用到Move，Move方法需要自己实现重力的效果，看具体项目需求

```csharp
//用摇杆控制方向，当按下空格键时跳起。
using UnityEngine;
using System.Collections;

public class ExampleClass : MonoBehaviour {
    public float speed = 6.0F;
    public float jumpSpeed = 8.0F;
    public float gravity = 20.0F;
    private Vector3 moveDirection = Vector3.zero;

    void Update() {
        CharacterController controller = GetComponent<CharacterController>();
        if (controller.isGrounded) {
            moveDirection = GetTargetPos(nowPos) - nowPos;

            // 跳跃
            if (Input.GetButton("Jump"))
                moveDirection.y = jumpSpeed;
        }

        //Move方法需要自己写重力效果
        moveDirection.y -= gravity * Time.deltaTime;

        //移动控制器
        controller.Move(moveDirection * Time.deltaTime);
    }
}
```

### 每帧目标点的计算

速度模拟模拟现实中人物移动的特征，起步时加速，计算代码如下

```csharp
Vector3 GetTargetPos(Vector3 pos)
{
	//计算实时速度，addSpeed为加速度
	curMoveSpeed = Mathf.MoveTowards(curMoveSpeed, MaxSpeed, addSpeed * deltaTime);
	//计算终点，curMoveDir由输入决定
	pos += curMoveSpeed * curMoveDir * deltaTime
}
```

### 动画状态机

移动时需要动作配合，动作主要由Animator组件实现，通过设置不同的参数，实现状态之间的转换，还可以细化，加上准备跳跃，跳跃落地等等。

通过SetTrigger进行不同运动状态的切换，通过SetBool进行移动和静止状态间的切换。

![](https://raw.githubusercontent.com/dandkong/picgo/main/img/20221212195654.png)

### 管理组件

主动移动的管理组件主要功能如下

- 接收输入方向
- 状态机维护当前角色状态（普通，跳跃），设置对应动画状态
- 计算下一帧移动方向

<br>

## 三、自动寻路

- 自动寻路主要是计算出起始点到目标点的一连串的中间点，每次走一小段直线，从而完成寻路的过程，逻辑比较简单，
- 如果使用Unity自带的Nav Mesh Agent 可以使用参考[https://bbs.huaweicloud.com/blogs/303788](https://bbs.huaweicloud.com/blogs/303788)。

### 计算寻路点

计算寻路点主要用到Unity的[NavMesh](https://docs.unity3d.com/ScriptReference/AI.NavMesh.html)类中的CalculatePath方法，得到寻路点以后只要逐个进行寻路即可。

```csharp
public static bool CalculatePath(Vector3 sourcePosition, Vector3 targetPosition, int areaMask, AI.NavMeshPath path);
```

### 逐点寻路

计算出寻路点以后，在Update中执行

```csharp
private void UpdateMove(float deltaTime)
{
	float time = Vector3.Distance(_nowPos, _destList[index])/speed;
	If(deltaTime > time)
	{
			//如果到最后一个，结束

			//没有的话，index++
			//多出的时间留给下个寻路点
			UpdateMove(deltaTime - time);
	}
	else
	{
			//插值
			_nowPos = Vector3.Lerp(_nowPos, _destList[index], deltaTime / time);
	}
}
```

