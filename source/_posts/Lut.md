---
title: 美颜你的Unity--使用Lut为Unity调色
---

![](http://wx2.sinaimg.cn/mw690/bcd85caely1fg8gtni8luj21kw0vz1ex.jpg)



<!-- More-->

![](http://wx3.sinaimg.cn/mw690/bcd85caely1fg8h0i09ylj20u01hctdr.jpg) 
大家看了上面这张图片一定就明白本文的主题了，没错，今天的主题就是~~带大家一起舔小圆~~ 
（╯‵□′）╯︵┴─┴ 
哈哈，不开玩笑了。上图展示了相机的一个常用的功能–滤镜，相信大家在用手机拍照的时候都用过滤镜吧，相机自带的滤镜可以一键调整照片色调，让我们不用专门学习相关的摄影知识就能拍出想要的效果，非常的方便与实用。今天要研究的内容就是在Unity里使用类似滤镜的功能，快速的调整场景的色调。 
所需软件：Unity，PhotoShop 
**注意：本文所研究的内容基于Unity最新的后期屏幕渲染特效-Post Processing Stack，该插件重构了Unity5.6之前版本的后期屏幕渲染特效-Image Effects，如果原有的项目中使用了旧的后期屏幕渲染特效（Image Effects），那么导入Post Processing Stack就会出现脚本冲突，请注意备份项目。**

—–这是没什么用的说明的分割线——- 
通过在不同版本（5.6与5.6之前的版本）的Unity的Project面板上右键/导入Effects包，可以看得出Unity在5.6版本中已经移除了所有Image Effects相关的脚本，只留下了一个文档，打开文档可以看到官方推荐使用新的Post Processing Stack的说明，当然如果想使用旧版本的Image Effects也可以通过Asset Store免费下载。新版本的Post Processing Stack使用起来比之前的Image Effects要方便的多，之后我会抽时间写一写关于Post Processing Stack的文章，现在就先讲讲这个“滤镜”功能吧。 
![](http://wx4.sinaimg.cn/mw690/bcd85caely1fg8i8ngpvgj20de0b7mxe.jpg) 
![5.6之前的版本，带有Image Effect ](http://wx2.sinaimg.cn/mw690/bcd85caely1fg8i7cpjk7j20i30hs40h.jpg) 

![5.6的版本，移除了Image Effect相关脚本](http://wx4.sinaimg.cn/mw690/bcd85caely1fg8i7cewjkj20i30fitab.jpg) 
 
![官方对于组件更新的说明](http://wx4.sinaimg.cn/mw690/bcd85caely1fg8i7c3122j207m06ijre.jpg) 


———–这是萌萌哒的正文分割线——— 
首先，我们在Unity中打开Asset Store，搜索“Post Processing”，可以看到Post Processing Stack插件包。（另外可以从[GitHub](https://github.com/Unity-Technologies/PostProcessing)上下载。如果网络不好的话，也可以通过[百度云](http://pan.baidu.com/s/1boT08YN)进行下载，我会尽量跟着官方更新版本。）下载完成之后先别导入。 
![](http://wx1.sinaimg.cn/mw690/bcd85caely1fg8i54ksrcj20qv0imgqi.jpg) 
本文的示例场景是我从Asset Store上找的一个免费的环境包，大家可以下载这个环境包，也可以使用其他任意场景。

——如果使用其他场景，可以跳过下面这些操作—— 
打开Asset Store，搜索“Hong Kong”，可以下载到这个场景。（也可以通过[百度云](http://pan.baidu.com/s/1hr7BK4O)下载。） 
![](http://wx4.sinaimg.cn/mw690/bcd85caely1fg8knn8hhfj20nr0gagpd.jpg) 
导入这个环境包，根据提示升级API(点击Go Ahead!) 
![](http://wx2.sinaimg.cn/mw690/bcd85caely1fg8kv0b3ftj20az065aa7.jpg) 
升级完成之后可以看到该场景使用的是旧版的Image Effects，删除以下两个文件夹：Assets/Unity Set Builder/Editor/Image Effects，Assets/Unity Set Builder/Standard Assets/Image Effects(Pro Only) 
![](http://wx2.sinaimg.cn/mw690/bcd85caely1fg8kr9lsmsj20ad0aswet.jpg) 
打开Assets/Unity Set Builder/Set Builder文件夹下的Sample场景，找到场景中的Camera，将Camera上面的一些丢失的组件都删掉，这些丢失的组件就是刚才删除的Image Effects的后期屏幕渲染特效组件。 
![](http://wx3.sinaimg.cn/mw690/bcd85caely1fg8kyvhsaaj208f0r1myp.jpg)

接下来，导入Post Processing Stack插件包。 
![](http://wx1.sinaimg.cn/mw690/bcd85caely1fg8i550eh9j20pp0ilgsd.jpg) 
![](http://wx1.sinaimg.cn/mw690/bcd85caely1fg8i5613b2j20i30at3z9.jpg) 
然后，找到场景中的Camera，为它添加一个Post-ProcessingBehaviour的脚本。 
![](http://wx3.sinaimg.cn/mw690/bcd85caely1fg8l36pr6ej207x0bx74g.jpg) 
这个脚本只包含一个公共变量，那就是Post Processing的配置文件。 
![](http://wx2.sinaimg.cn/mw690/bcd85caely1fg8l4tgyt7j207n021746.jpg) 
在Project面板下，创建一个Post-Processing Profile文件，随意命名即可。 
![](http://wx2.sinaimg.cn/mw690/bcd85caely1fg8l6o8f0pj20gu0kkjs6.jpg) 
鼠标点选这个profile文件可以看得到，这个文件包含了所有的后期屏幕渲染特效的设置属性。之前的各种特效需要Image Effects的各种脚本来控制，现在的话只需要配置这一个profile文件就可以了。 
![](http://wx3.sinaimg.cn/mw690/bcd85caely1fg8l8l7gunj20h20dv0ta.jpg) 
接着把这个profile文件拖到Camera上Post-ProcessingBehaviour脚本的Profile变量上。 
![](http://wx3.sinaimg.cn/mw690/bcd85caely1fg8lb2nt68j208k02cwee.jpg) 
到这里为止，我们就做完了Unity的前置工作。 
接下来让我们处理一下Lut。唔，这里简单的介绍下Lut，Lut其实就是一些调色的预设，可以理解为相机自带的各种滤镜，可以用在各种图形图像处理软件上，如Photoshop，After Effect等。我从网上找了一些Lut文件，下面会用得到。[下载](http://pan.baidu.com/s/1nv2sfKd)之后解压出来。 
我们先在Unity中把当前的Game窗口截一下图，然后使用PhotoShop打开这张截图。 
![](http://wx3.sinaimg.cn/mw690/bcd85caely1fg8ma5i69oj20tk0gotz6.jpg) 
接着从Photoshop的菜单中选择“图像/调整/颜色查找”，接着选择“载入3D Lut”，找到刚才下载的Lut的文件夹，任意选择一个Lut文件来查看效果。 
![](http://wx2.sinaimg.cn/mw690/bcd85caely1fg8me209w4j20eq0gh0um.jpg) 
![](http://wx2.sinaimg.cn/mw690/bcd85caely1fg8mi1ye8xj20f107kmxa.jpg)![](http://wx2.sinaimg.cn/mw690/bcd85caely1fg8mi2aqqdj20fp0f7dkk.jpg)![](http://wx2.sinaimg.cn/mw690/bcd85caely1fg8mi2ly6uj20iu0io0vt.jpg)

或者可以从面板的快捷菜单中选择“颜色查找”，然后“载入3D Lut”，和上面的效果其实是一样的。 
![](http://wx3.sinaimg.cn/mw690/bcd85caely1fg8me2adqqj20eh0bcwew.jpg) 
![](http://wx4.sinaimg.cn/mw690/bcd85caely1fg8mki0c5nj20gd0b2gmf.jpg) 
![](http://wx4.sinaimg.cn/mw690/bcd85caely1fg8mjnhvlzj208t0fymxg.jpg)![](http://wx3.sinaimg.cn/mw690/bcd85caely1fg8mjnsd3uj20ol0iataw.jpg)

这里我选择了“DELUTS_CLOG3_Look_01.cube”，对比一下可以看出与之前的截图有明显的区别。 
![](http://wx4.sinaimg.cn/mw690/bcd85caely1fg8mn18dopj20tk0goqud.jpg) 
![](http://wx3.sinaimg.cn/mw690/bcd85caely1fg8ma5i69oj20tk0gotz6.jpg) 
大家可以再尝试更换其他的Lut文件来选择自己想要的色调。 
另外也可以添加多个Lut文件，并且调整每个Lut的不透明度，以此获得单个Lut所没有的效果。 
![](http://wx3.sinaimg.cn/mw690/bcd85caely1fg8mrhlqacj208y06jjri.jpg)![img](http://wx1.sinaimg.cn/mw690/bcd85caely1fg8mrhb3qej208z05s0ss.jpg) 
当截图的效果满足需求之后，记下当前使用的Lut文件，稍后会用到。 
接下来就需要制作Unity中使用的Lut贴图了，首先回到Unity中，在Project面板里找到贴图Assets/PostProcessing/Textures/LUTs/NeutralLUT_32.png，把它复制一份，使用PhotoShop打开。 
![](http://wx2.sinaimg.cn/mw690/bcd85caely1fg8mute5kdj209k05a748.jpg) 
然后按照前面的步骤，为这张图片添加最终效果的Lut文件。 
![](http://wx1.sinaimg.cn/mw690/bcd85caely1fg8mzxexy0j20dn03qdfq.jpg) 
添加完成之后，另存为Lut.png文件，导入Unity中。 
接着选中之前创建的Post Processing Profile文件，勾选“User Lut”选项，并把刚才导入的Lut.png拖到Lut上，可以看到一个错误提示，这是因为需要对图片格式进行设置，这里点击Fix按钮，Unity会自动设置贴图格式。现在就可以在Game窗口中看到最终效果了，是不是和Photoshop中的一样呢？ 
![](http://wx3.sinaimg.cn/mw690/bcd85caely1fg8n4ek3lpj207h037glh.jpg)![](http://wx2.sinaimg.cn/mw690/bcd85caely1fg8n5zifxrj20ub0httzo.jpg) 
运行之后，可以四处走动观察下效果，由于此次主要讲解Lut的内容，所以没有对灯光及其他后期屏幕渲染特效进行处理，场景整体看着会有些偏暗。 
以上就是为Unity添加“美颜滤镜”的全部过程了，最后扔几张效果图来结尾吧。 
![](http://wx1.sinaimg.cn/mw690/bcd85caely1fg8no6wynlj21kw0w2wwj.jpg)

![](http://wx3.sinaimg.cn/mw690/bcd85caely1fg8no7v8r3j21kw0w0aqm.jpg)

![](http://wx1.sinaimg.cn/mw690/bcd85caely1fg8gtmwa2lj21kw0vzh3e.jpg)

![](http://wx4.sinaimg.cn/mw690/bcd85caely1fg8g1amzfsj21kw0vz7qi.jpg)

![](http://wx2.sinaimg.cn/mw690/bcd85caely1fg8gtni8luj21kw0vz1ex.jpg)