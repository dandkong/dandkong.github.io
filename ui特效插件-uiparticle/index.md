# UI特效插件-UIParticle


## 前言

在UI上显示特效是一直是一个比较棘手的问题，一开始的方案是调整UI特效的渲染顺序，设置其`sortingLayer`和`sortingOrder+offset`，在每个单独的UI以后渲染。

这种方法能满足大部分的需求，但也有缺点。

如果需要在特效上再盖一层UI的话，由于特效并不属于Canvas的一部分，所以修改`Hierarchy`并不能改变其渲染顺序，特效会在渲染完`Canvas`的UI元素以后再进行渲染，所以需要把这个UI的层级通过添加`Canvas`继续往上抬。

再者无法被`RectMask2D`和`Mask`裁剪，除非改造Shader，并手动传入裁剪区域。

最好的方法就是让特效像UI一样，直接按`Hierarchy` 顺序渲染，并支持遮罩，就是这文章的主角UIParticle。

## 原理

> This package utilizes the new `APIsMeshBake/MashTrailBake`(introduced with Unity 2018.2) to render particles through CanvasRenderer.
>
> You can render, mask, and sort your ParticleSystems for UI without the necessity of an additional Camera, RenderTexture, or Canvas.

这是项目首页的原理，具体就是通过一个API让例子特效通过`CanvasRender`进行渲染，这样就和普通的UI没什么不同了。

## 使用

按官方的教程，直接引入包，添加组件到特效上，直接拖到UI中即可。

## 改造已有shader以支持遮罩

项目推荐我们使用UIShader，如UIDefault或者UIAdditive。

Mask通过模板测试实现遮罩，RectMask2D通过传入裁剪区域实现。

所以自定义的Shader只需要参考上面的UIShader使其拥有遮罩特效即可，注意`RenderQueue`设置为`Transparent`，下面贴一下关键Shader代码。

```jsx
Shader "Your/Custom/Shader"
{
    Properties
    {
        // ...
        // #### required for Mask ####
        _StencilComp ("Stencil Comparison", Float) = 8
        _Stencil ("Stencil ID", Float) = 0
        _StencilOp ("Stencil Operation", Float) = 0
        _StencilWriteMask ("Stencil Write Mask", Float) = 255
        _StencilReadMask ("Stencil Read Mask", Float) = 255
        _ColorMask ("Color Mask", Float) = 15
        [Toggle(UNITY_UI_ALPHACLIP)] _UseUIAlphaClip ("Use Alpha Clip", Float) = 0
    }

    SubShader
    {
        Tags
        {
            // ...
        }

        // #### required for Mask ####
        Stencil
        {
            Ref [_Stencil]
            Comp [_StencilComp]
            Pass [_StencilOp]
            ReadMask [_StencilReadMask]
            WriteMask [_StencilWriteMask]
        }
        ColorMask [_ColorMask]
        // ...

        Pass
        {
            // ...
            // #### required for RectMask2D ####
            #include "UnityUI.cginc"
            #pragma multi_compile __ UNITY_UI_CLIP_RECT
            float4 _ClipRect;

            // #### required for Mask ####
            #pragma multi_compile __ UNITY_UI_ALPHACLIP

            struct appdata_t
            {
                // ...
            };

            struct v2f
            {
                // ...
                // #### required for RectMask2D ####
                float4 worldPosition    : TEXCOORD1;
            };
            
            v2f vert(appdata_t v)
            {
                v2f OUT;
                // ...
                // #### required for RectMask2D ####
                OUT.worldPosition = v.vertex;
                return OUT;
            }

            fixed4 frag(v2f IN) : SV_Target
            {
                // ...
                // #### required for RectMask2D ####
                #ifdef UNITY_UI_CLIP_RECT
                    color.a *= UnityGet2DClipping(IN.worldPosition.xy, _ClipRect);
                #endif

                // #### required for Mask ####
                #ifdef UNITY_UI_ALPHACLIP
                    clip (color.a - 0.001);
                #endif

                return color;
            }
            ENDCG
        }
    }
}
```

## 总结

至此已经可以像UI一样使用UI特效了，具体的性能分析，参考一下首页。

## 参考

https://github.com/mob-sakai/ParticleEffectForUGUI

[Unity Mask 和RectMask2D原理和区别-CSDN博客](https://blog.csdn.net/qq_32756581/article/details/126510907)

[Unity的三级排序层级渲染Layer,sorting layer,order in layer_unity layer 和order-CSDN博客](https://blog.csdn.net/qq_42672770/article/details/109442043)

[UGUI：调整Unity中UI和特效的层级关系（特效穿透问题）_unity中特效显示在image背景上 其他ui元素覆盖在特效上 层级怎么处理-CSDN博客](https://blog.csdn.net/qq_33461689/article/details/106135399)

