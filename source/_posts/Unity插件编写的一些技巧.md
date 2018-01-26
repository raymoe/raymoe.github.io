---
title: Unity插件编写的一些技巧
date: 2017-11-04 16:35:14
tags: Unity
categories: Unity
---

### 创建自定义类型的Asset  
首先需要创建一个类继承自ScriptableObject


### 定制打开某个Asset的方法
添加UnityEditor.Callbacks.OnOpenAsset() Attribute  
示例
```
[UnityEditor.Callbacks.OnOpenAsset()]
public static bool OnOpenAsset( int instanceID, int line )
{
    var graph = EditorUtility.InstanceIDToObject( instanceID ) as Model.ConfigGraph;
    if(graph != null) {
        var window = GetWindow<AssetBundleGraphEditorWindow>();
        window.OpenGraph(graph);
        return true;
    }
    return false;
}
```
