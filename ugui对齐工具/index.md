# UGUI对齐工具


![](https://raw.githubusercontent.com/dandkong/picgo/main/img/20221116234335.png)

## 前言

上半年的大半年时间都在用FGUI，最近又用回了UGUI，发现FGUI的一些工具还是很高效的，比如对齐工具，想着说移植到UGUI里面去，上网搜了一圈还真有，原理并不复杂，就是进行一些数学计算出最终的位置。

不过也没有完全适合项目需求，针对原项目基础上添加了三个功能，下面介绍一下。

<br/>

## 一、撤销功能

原项目的工具在修改后，习惯性就按下Ctrl+Z，发现并不能撤销原来的操作，查询了Unity的API后，发现有个Undo的类，可以实现撤销功能，有很多方法，主要用到了RecordObject，代码如下

```c#
private static void SetTranPos(Transform tran, Vector3 pos)
{
    Undo.RecordObject(tran, "modify posstion");
    tran.position = pos;
}
```

<br/>

## 二、Selection.GameObjects不是按选择顺序的

API中的Selection.GameObjects用于获取当前选中的物件，数组的顺序和我选择的顺序无关，所以不能实现灵活的对齐，查询网上文章可以用Selection.Objects替代，里面的第一个元素就是Ctrl选中的第一个元素，以此类推，代码如下

```C#
//按选中顺序获取GameObjects
private static GameObject[] GetOrderedSelctionObjs()
{
    return Selection.objects.OfType<GameObject>().ToArray();
}
```

<br/>

## 三、添加对父节点对齐

功能就是和标题一样，原项目是没有的，代码如下

```c#
//如果只选中一个，对父节点对齐
if (rects.Count == 1)
{
    if (rects[0].parent != null && rects[0].parent.GetComponent<RectTransform>() != null)
    {
        rects.Insert(0, rects[0].parent.GetComponent<RectTransform>());
    }
}
```

<br/>

## 四、代码

### UGUIAlign

```C#
using System.Collections.Generic;
using System.Linq;
using UnityEditor;
using UnityEngine;

public enum AlignType
{
    Top = 1,
    Left = 2,
    Right = 3,
    Bottom = 4,
    HorizontalCenter = 5,       //水平居中
    VerticalCenter = 6,         //垂直居中
    Horizontal = 7,             //横向分布
    Vertical = 8,               //纵向分布
}

public class UGUIAlign : Editor
{
    [MenuItem("GameObject/UI/Align/Left 【左对齐】")]
    static void AlignLeft()
    {
        Align(AlignType.Left);
    }
    [MenuItem("GameObject/UI/Align/HorizontalCenter 【水平居中】")]
    static void AlignHorizontalCenter()
    {
        Align(AlignType.HorizontalCenter);
    }
    [MenuItem("GameObject/UI/Align/Right 【右对齐】")]
    static void AlignRight()
    {
        Align(AlignType.Right);
    }
    [MenuItem("GameObject/UI/Align/Top 【顶端对齐】")]
    static void AlignTop()
    {
        Align(AlignType.Top);
    }
    [MenuItem("GameObject/UI/Align/VerticalCenter 【垂直居中】")]
    static void AlignVerticalCenter()
    {
        Align(AlignType.VerticalCenter);
    }
    [MenuItem("GameObject/UI/Align/Bottom 【底端对齐】")]
    static void AlignBottom()
    {
        Align(AlignType.Bottom);
    }
    [MenuItem("GameObject/UI/Align/Horizontal 【横向分布】")]
    static void AlignHorizontal()
    {
        Align(AlignType.Horizontal);
    }
    [MenuItem("GameObject/UI/Align/Vertical 【纵向分布】")]
    static void AlignVertical()
    {
        Align(AlignType.Vertical);
    }

    public static void Align(AlignType type)
    {
        List<RectTransform> rects = new List<RectTransform>();
        GameObject[] objects = GetOrderedSelctionObjs();
        if (objects != null && objects.Length > 0)
        {
            for (int i = 0; i < objects.Length; i++)
            {
                RectTransform rect = objects[i].GetComponent<RectTransform>();
                if (rect != null)
                    rects.Add(rect);
            }
        }

        //如果只选中一个，对父节点对齐
        if (rects.Count == 1)
        {
            if (rects[0].parent != null && rects[0].parent.GetComponent<RectTransform>() != null)
            {
                rects.Insert(0, rects[0].parent.GetComponent<RectTransform>());
            }
        }

        if (rects.Count > 1)
        {
            Align(type, rects);
        }
    }

    //按选中顺序获取GameObjects
    private static GameObject[] GetOrderedSelctionObjs()
    {
        return Selection.objects.OfType<GameObject>().ToArray();
    }

    public static void Align(AlignType type, List<RectTransform> rects)
    {
        RectTransform tenplate = rects[0];
        float w = tenplate.sizeDelta.x * tenplate.lossyScale.x;
        float h = tenplate.sizeDelta.y * tenplate.lossyScale.y;

        float x = tenplate.position.x - tenplate.pivot.x * w;
        float y = tenplate.position.y - tenplate.pivot.y * h;

        switch (type)
        {
            case AlignType.Top:
                for (int i = 1; i < rects.Count; i++)
                {
                    RectTransform trans = rects[i];
                    float th = trans.sizeDelta.y * trans.lossyScale.y;
                    Vector3 pos = trans.position;
                    pos.y = y + h - th + trans.pivot.y * th;
                    SetTranPos(trans, pos);
                }
                break;
            case AlignType.Left:
                for (int i = 1; i < rects.Count; i++)
                {
                    RectTransform trans = rects[i];
                    float tw = trans.sizeDelta.x * trans.lossyScale.x;
                    Vector3 pos = trans.position;
                    pos.x = x + tw * trans.pivot.x;
                    SetTranPos(trans, pos);
                }
                break;
            case AlignType.Right:
                for (int i = 1; i < rects.Count; i++)
                {
                    RectTransform trans = rects[i];
                    float tw = trans.sizeDelta.x * trans.lossyScale.x;
                    Vector3 pos = trans.position;
                    pos.x = x + w - tw + tw * trans.pivot.x;
                    SetTranPos(trans, pos);
                }
                break;
            case AlignType.Bottom:
                for (int i = 1; i < rects.Count; i++)
                {
                    RectTransform trans = rects[i];
                    float th = trans.sizeDelta.y * trans.lossyScale.y;
                    Vector3 pos = trans.position;
                    pos.y = y + th * trans.pivot.y;
                    SetTranPos(trans, pos);
                }
                break;
            case AlignType.HorizontalCenter:
                for (int i = 1; i < rects.Count; i++)
                {
                    RectTransform trans = rects[i];
                    float tw = trans.sizeDelta.x * trans.lossyScale.x;
                    Vector3 pos = trans.position;
                    pos.x = x + 0.5f * w - 0.5f * tw + tw * trans.pivot.x;
                    SetTranPos(trans, pos);
                }
                break;
            case AlignType.VerticalCenter:
                for (int i = 1; i < rects.Count; i++)
                {
                    RectTransform trans = rects[i];
                    float th = trans.sizeDelta.y * trans.lossyScale.y;
                    Vector3 pos = trans.position;
                    pos.y = y + 0.5f * h - 0.5f * th + th * trans.pivot.y;
                    SetTranPos(trans, pos);
                }
                break;
            case AlignType.Horizontal:
                float minX = GetMinX(rects);
                float maxX = GetMaxX(rects);
                rects.Sort(SortListRectTransformByX);
                float distance = (maxX - minX) / (rects.Count - 1);
                for (int i = 1; i < rects.Count - 1; i++)
                {
                    RectTransform trans = rects[i];
                    Vector3 pos = trans.position;
                    pos.x = minX + i * distance;
                    SetTranPos(trans, pos);
                }
                break;
            case AlignType.Vertical:
                float minY = GetMinY(rects);
                float maxY = GetMaxY(rects);
                rects.Sort(SortListRectTransformByY);
                float distanceY = (maxY - minY) / (rects.Count - 1);
                for (int i = 1; i < rects.Count - 1; i++)
                {
                    RectTransform trans = rects[i];
                    Vector3 pos = trans.position;
                    pos.y = minY + i * distanceY;
                    SetTranPos(trans, pos);
                }
                break;
        }
    }

    private static void SetTranPos(Transform tran, Vector3 pos)
    {
        Undo.RecordObject(tran, "modify posstion");
        tran.position = pos;
    }

    private static int SortListRectTransformByX(RectTransform r1, RectTransform r2)
    {
        float w = r1.sizeDelta.x * r1.lossyScale.x;
        float x1 = r1.position.x - r1.pivot.x * w;
        w = r2.sizeDelta.x * r2.lossyScale.x;
        float x2 = r2.position.x - r2.pivot.x * w;
        if (x1 >= x2)
            return 1;
        else
            return -1;
    }

    private static int SortListRectTransformByY(RectTransform r1, RectTransform r2)
    {
        float w = r1.sizeDelta.y * r1.lossyScale.y;
        float y1 = r1.position.y - r1.pivot.y * w;
        w = r2.sizeDelta.y * r2.lossyScale.y;
        float y2 = r2.position.y - r2.pivot.y * w;
        if (y1 >= y2)
            return 1;
        else
            return -1;
    }

    private static float GetMinX(List<RectTransform> rects)
    {
        if (null == rects || rects.Count == 0)
            return 0;
        RectTransform tenplate = rects[0];
        float minx = tenplate.position.x;
        float tempX = 0;
        for (int i = 1; i < rects.Count; i++)
        {
            tempX = rects[i].position.x;
            if (tempX < minx)
                minx = tempX;
        }
        return minx;
    }

    private static float GetMaxX(List<RectTransform> rects)
    {
        if (null == rects || rects.Count == 0)
            return 0;
        RectTransform tenplate = rects[0];
        float maxX = tenplate.position.x;
        float tempX = 0;
        for (int i = 1; i < rects.Count; i++)
        {
            tempX = rects[i].position.x;
            if (tempX > maxX)
                maxX = tempX;
        }
        return maxX;
    }

    private static float GetMinY(List<RectTransform> rects)
    {
        if (null == rects || rects.Count == 0)
            return 0;
        RectTransform tenplate = rects[0];
        float minY = tenplate.position.y;
        float tempX = 0;
        for (int i = 1; i < rects.Count; i++)
        {
            tempX = rects[i].position.y;
            if (tempX < minY)
                minY = tempX;
        }
        return minY;
    }

    private static float GetMaxY(List<RectTransform> rects)
    {
        if (null == rects || rects.Count == 0)
            return 0;
        RectTransform tenplate = rects[0];
        float maxY = tenplate.position.y;
        float tempX = 0;
        for (int i = 1; i < rects.Count; i++)
        {
            tempX = rects[i].position.y;
            if (tempX > maxY)
                maxY = tempX;
        }
        return maxY;
    }
}

```

<br/>

## 五、代码下载

<https://github.com/dandkong/UGUIAlign>

<br/>

## 六、参考

[Unity编辑器拓展之十：UI对齐工具](https://blog.csdn.net/qq_26999509/article/details/80779395)




