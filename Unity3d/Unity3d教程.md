## Unity3d教程

[TOC]

Unity3d可以用来做游戏、做动画。

## 一.概念

**Unity3d的概念从大到小依次是：Project$\rightarrow$Scene$\rightarrow$Prefab$\rightarrow$GameObject$\rightarrow$Component**

通俗地说，一个游戏是一个Project，游戏中的一个关卡是一个Scene，场景中的一堆物体是一个Prefab，一堆物体中的每一个物体是一个GameObject，物体的每个性质或功能是一个Component。

Unity3d中的name应该避免重复，尽管不会报错；同样，不应该在同一个GameObject上放置多个同名Component。

#### 1.GameObject

GameObject可以看作一个`List<Component>`，Component（组件）属于GameObject，但Component才是重要的信息，各种Component代表着各种功能，如会发光，能跑能跳，Component们决定GameObject的用途。

总而言之，GameObject是一个挂名领导，不管实事，但是作为Component们的精神领袖牵一发而动全身，并且可以互相嵌套层级作为子GameObject。

在Unity的脚本系统中，GameObject提供了许多管理场景物体的静态方法，如`Instantiate`、`Destrory`、各种`Find`。

#### 2.Prefab

预制体（Prefab）是我们在Unity中生成物体实例的自定义模板，它跟复制粘贴的效果一样，但显然手法更加高级、更具有可维护性，并且可以在游戏运行时动态生成需要的GameObject。

拖拽Scene中的东西到Asset中就可以创建prefab，再次拖拽则可以更新。

#### 3.Component

常见的Component有

##### （1）Transform

​	最基本的组件，物体的尺寸，旋转，缩放。

##### （2）MeshRenderer/MeshFilter

​	能被看见的物体必备组件

##### （3）Box Collider

​	盒子碰撞器，有碰撞器才会有碰撞检测

#### 4.EditorInterface

Unity编辑器左边是Hierarchy面板，显示当前Scene中所有的GameObject，当选中了一个GameObject，其Component会在右侧的Inspector面板中显示，中间是当前Scene，下方是资源目录，与Windows资源管理器作用相似，可以新建文件夹，新建资源文件，新建GameObject。

#### 5.Deactivate

你可以让场景中任何一个物体变得不可用（deactivate），但不一定能够让任何物体变得可用，因为即使如果让一个物体不可用，其所有子物体都变得不可用，而且不能通过setActive的方式让子物体变得可用。

#### 6.Tag

用脚本可以读取Tag，分组管理场景中的物体。例如游戏中的可以拿起来的物体可以标注Tag为`Collection`通过Unity提供的GameObject下的静态方法`FindWithTag`索引，这样就不需要手动把物体拖拽到脚本中的已收集物体数组中。

```C#
GameObject collections = GameObject.FindWithTag("Collection");
```

Layer的原理和Tag类型，但主要是给编译器看来判断如何渲染，而不是供脚本使用。

Layer的右边是Static，如果打勾表示是不会动的东西，下拉选项可以选择在哪些系统中预计算，是性能优化的部分。

#### 7.Script

新建一个`.cs`C#文件，其中有一个公开的、与文件名同名的类，Unity将这个类编译为组件（Component），其中公开的属性会作为选项出现在Inspector面板。

#### 8.AssetStore

截至2020.4.25，访问Unity账户仍需要科学上网，而下载资源不需要（且速度很快）。

AssetStore下载的资源包在`C:\Users\用户名\AppData\Roaming\Unity\Asset Store`目录下

<https://docs.unity3d.com/Manual/EditingInPrefabMode.html>

#### 二.入门

`Ctrl + P`	运行/退出运行模式

`Ctrl + D`	复制