---
title: 小工具01-为Animator Controller添加Animation Clip
date: 2018-01-10 23:28:08
tags: Unity,小工具
categories: 小工具
---

![最终效果](http://wx2.sinaimg.cn/mw690/bcd85caely1fnbxnoa8rjg20ez0b3q72.gif)

<!-- More -->

通常使用Animation创建的Animation Clip与Animator都是分开的，如果用到的Animation Clip比较多的话,文件夹会比较乱，如下图所示。

![默认创建Clip效果](http://wx4.sinaimg.cn/large/bcd85caely1fnbxy2tg1jg20oo0hxk62.gif)

通过脚本来把Animator用到的所有Animation Clip都添加为其子资源，如文章开始的动图所示。

新建一个脚本 NestedAnimationCreator.cs，放到Editor文件夹下。

引入命名空间

```
using UnityEditor;
using UnityEngine;
using UnityEditor.Animations;
```

创建一个窗口类，用于填写要创建的Clip的名字

```
public class CreateNewAnimationClipWindow : EditorWindow
{
    //对话框标题
    public string CaptionText { set; get; }
    //按钮文本
    public string ButtonText { set; get; }
    //输入的Clip名字
    public string NewName { set; get; }
    //按钮点击的委托
    public System.Action<string> OnClickButtonDelegate { get; set; }

    void OnGUI()
    {
        NewName = EditorGUILayout.TextField(CaptionText, NewName);
        if (GUILayout.Button(ButtonText))
        {
            if (OnClickButtonDelegate != null)
            {
                OnClickButtonDelegate.Invoke(NewName.Trim());
            }
            Close();
            GUIUtility.ExitGUI();
        }
    }
}
```

创建工具类，用于创建与删除Clip

```
public class NestedAnimationCreator : MonoBehaviour
{
    [MenuItem("iTools/Animation/Create Nested Animation Clip")]
    public static void Create()
    {
        //获取在Project面板中选中的AnimatorController
        AnimatorController selectedAnimatorController =
            Selection.activeObject as AnimatorController;
        if (selectedAnimatorController == null)
        {
            Debug.LogError("Not Select Animator Controller");
            return;
        }
        //打开新建Clip的窗口
        CreateNewAnimationClipWindow createNewAnimationClipWindow =
            EditorWindow.GetWindow<CreateNewAnimationClipWindow>("Create Nested Animation Clip");
        createNewAnimationClipWindow.ButtonText = "Create";
        createNewAnimationClipWindow.CaptionText = "New Animation Name";
        createNewAnimationClipWindow.NewName = "New Clip";
        
        createNewAnimationClipWindow.OnClickButtonDelegate = (string newName) =>
        {
            if (string.IsNullOrEmpty(newName))
            {
                Debug.LogError("Invalid name");
                return;
            }
            //根据输入的名称创建Clip
            AnimationClip newClip =
                AnimatorController.AllocateAnimatorClip(newName);
            //将创建的Clip添加到选中的Controller子资源中
            AssetDatabase.AddObjectToAsset(newClip, selectedAnimatorController);
            //重新导入资源
            AssetDatabase.ImportAsset(
                AssetDatabase.GetAssetPath(selectedAnimatorController));
        };
    }

    [MenuItem("iTools/Animation/Delete Animation Clip")]
    public static void Delete()
    {
        Object[] selectedAssets = Selection.objects;
        if (selectedAssets.Length < 1)
        {
            Debug.LogError("Not select asset");
            return;
        }
        foreach (var asset in selectedAssets)
        {
            if (AssetDatabase.IsSubAsset(asset))
            {
                string path = AssetDatabase.GetAssetPath(asset);
                DestroyImmediate(asset, true);
                AssetDatabase.ImportAsset(path);
            }
        }
    }
}
```

