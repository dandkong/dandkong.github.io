# 利用ActionManger拓展psd2gui


<!--more-->

## 前言

游戏的UI制作一直是程序员的考验，如果是程序员来拼UI的话，会遇到和美术效果不一样，如果是美术拼的话，又需要成本，美术一般对Unity不是很熟悉。之前也用过FGUI，但是bug不少，实际上用起来还是UGUI更为顺手。最后项目试用了PSD2GUI这个思路，把美术出的PSD直接转换成UGUI。理想的情况下是不用美术培训太多就能输出界面，所以没有做过多的组件支持，只是导出一些最基础的元素，比如图片和文字，程序在这基础上再进行加工，但也能省去大部分的重复工作了，而且不容易出错。如果流程更加成熟，可以培训美术理解更多prefab相关知识，可以支持导出一些列表，复选框之类的元素，但这要考虑成本。

代码基本来自https://github.com/zs9024/quick_psd2ugui/tree/master，基本没有用他的命名方式，只用了基本的图片和文字导出，支持了位置，大小，文本，文本颜色，透明度等基础信息。在实际实用中美术会标注一些不常规的情况，比如描边，渐变，图片的叠色，旋转，这些信息其实在PSD都有的，但是在API中并不能获取到，最后才参考[【CEP教程-8】Action Manager从好奇到劝退 - 中篇](https://blog.cutterman.cn/2021/12/19/action-manager-part2/)，直接读取了图层效果里的描边和图片叠色，因为是直接读取的，所以美术并没有学习成本，渐变的参数过于复杂，有点难支持，旋转的信息暂时没找到简单方法获取，但也省下了一点成本。



## 代码

主要就是利用ActionManger获取更多的PSD图层信息，PS在操作时会把数据都存到一个的结构中，只要能拿到这个结构，就能查到其中记录的数据，之后通过debug查看变量，或者通过示例代码照猫画虎即可。

```
/**
 * 这里就是拓展内容
 function exportLabel(obj,validFileName){
 	var desc = getLayerInfoByID(obj.id)
 	var json = ADToJSON(desc)
 	//这里就可以取到描边信息类似json.FrameX.solidFill.color，具体debug可以看到
 }
 */

/**
 * 这个函数接受一个AD的对象，返回这个对象所有属性值的JSON结构
 * @param desc [ActionDescriptor]
 * @constructor
 * @return JSON
 */
function ADToJson(desc) {
    var json = {};
    for (var i=0; i<desc.count; i++) {
        var typeID = desc.getKey(i);
        var stringID = typeIDToStringID(typeID);
        var typeString = (desc.getType(typeID)).toString();
        switch(typeString) {
            case "DescValueType.BOOLEANTYPE": 
                json[stringID] = desc.getBoolean(typeID);
            break;
            case "DescValueType.DOUBLETYPE": 
                json[stringID] = desc.getDouble(typeID);
            break;
            case "DescValueType.INTEGERTYPE": 
                json[stringID] = desc.getInteger(typeID);
            break;
            case "DescValueType.STRINGTYPE": 
                json[stringID] = desc.getString(typeID);
            break;
            case 'DescValueType.OBJECTTYPE':
                var objectValue = desc.getObjectValue(typeID);
                json[stringID] = ADToJson(objectValue);
		    break;
            case 'DescValueType.CLASSTYPE':
            case 'DescValueType.LISTTYPE':
            case 'DescValueType.REFERENCETYPE':
                // 剩下这些留给你去补充完成，这里还有UNITDOULETYPE之类的，像描边的size就是这类型
		    break;
            default: alert(typeString); break;
        }
    }
    return json;
}

/**
 * 根据图层ID来获取图层信息
 * @param layerID
 * @return {*}
 */
function getLayerInfoByID(layerID) {
    var ref1 = new ActionReference();
        ref1.putIdentifier(stringIDToTypeID( "Lyr ", layerID));
    var layerDescriptor = executeActionGet(ref1);
    var json = ADToJson(layerDescriptor);   
    return json;
}

/**
 * 根据图层的顺序，来获取图层信息
 * @param index
 * @return {*}
 */
function getLayerInfoByIndex(index) {
    var ref1 = new ActionReference();
        ref1.putIndex( stringIDToTypeID( "itemIndex" ), index); 
    var layerDescriptor = executeActionGet(ref1);
    var json = ADToJson(layerDescriptor);
    return json;   
}


/**
 * 根据图层的名称，来获取图层信息
 * @param index
 * @return {*}
 */
function getLayerInfoByName(name) {
    var ref1 = new ActionReference();
    ref1.putName( stringIDToTypeID( "layer" ), name );
    var layerDescriptor = executeActionGet(ref1);
    var json = ADToJson(layerDescriptor);
    return json
}


function getCurrentDocumentInfo() {
    var ref1 = new ActionReference();
    ref1.putEnumerated(charIDToTypeID('Dcmn'), charIDToTypeID('Ordn'), charIDToTypeID('Trgt'));
    var docDescriptor = executeActionGet(ref1);
    var json = ADToJson(docDescriptor);
    return json
}
```



## 总结

- 代码基本就是那样了，难点在于怎么知道json中的变量是对应自己想要的值，可以自己修改值，看看变化，或者通过参考[【CEP教程-8】Action Manager从好奇到劝退 - 上篇](https://blog.cutterman.cn/2021/12/12/action-manager-part1/)，装个插件，每一部的ps操作都会生成log，可以更快找到所需变量。
- jsx可以用vscode进行开发，配合插件ExtendScript可以输出日志和断点



## 参考

[【CEP教程-8】Action Manager从好奇到劝退 - 上篇](https://blog.cutterman.cn/2021/12/12/action-manager-part1/)

[【CEP教程-8】Action Manager从好奇到劝退 - 中篇](https://blog.cutterman.cn/2021/12/19/action-manager-part2/)

[【CEP教程-9】Action Manager从好奇到劝退 - 下篇](https://blog.cutterman.cn/2022/01/01/action-manager-part3/)

[Photoshop插件开发教程 - （4）开发工具选择和调试](https://zhuanlan.zhihu.com/p/559290141)

