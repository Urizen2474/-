

# UnrealEngine



## 1.世界场景构建

##### 1.1空关卡光照构建:大气雾，定向光，天空球，天空光

## 2.UEC++

### 1.使一个变量在UE编辑器中显示(用UP::查找)

```c++
UPROPERTY(EditAnywhere,BlueprintReadWrite)

   classType value_name;//VS里面要保存,然后在UE编辑器中点击编译
```

```c++
 UPROPERTY(EditAnywhere,BluprintReadWrite,Catergory="Name")
     .....;//将这个值放置在一个自己定义的目录下
```

```C++
GEngine->AddOnScreenDebugMessage(-1, 10.0f, FColor::Red, TEXT("ajwcb"));//第一个值-1与否代表后续语句
是否会替换前一个语句，第二个是语句显示时间，字体颜色，文本内容
```

### 2.注意事项

   1.VS编辑器要记得切英文键盘，不然没有提示

2. .generated.h文件必须是引用的最后一个文件

3. C++完全写好了才能用蓝图去继承他，否则需要把继承了的蓝图类删了再

   继承一次
   
   4.Box组件要放在StaticMesh组件下面,不然模型不会显示(但是这样会出现两个相同的StaticMesh,目前未解决)
   
   5.实测Box组件在这种情况下，StaticMesh会显示
   
   <img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220715105620889.png" alt="image-20220715105620889" style="zoom:50%;" />
   
   <img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220715105646928.png" alt="image-20220715105646928" style="zoom:50%;" />
   
   组件的层级关系
   
   
   
   <img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220715125010127.png" alt="image-20220715125010127"  />
   
   ![image-20220715125950178](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220715125950178.png)
   
   6.设置了Scene组件作为根组件，每次为这个actor添加组件都要用
   
   setupattachment()把他绑定到根组件上(不要偷懒)
   
   7.函数名称不能和变量相同
   
   #### 2.1初始化组件
   
   ```c++
   Saveloc_MD_Comp = CreateDefaultSubobject<UStaticMeshComponent>("Saveloc_MD");//用等号赋值
   Saveloc_MD_Comp->CreateDefaultSubobject<UStaticMeshComponent>("Saveloc_MD");//不要用->赋值，UE会直接崩溃，还不报错
   ```
   
   

### 3.使一个函数在UE编辑器中显示(用UF::查找)

```c++
UFUNCTION(BlueprintCallable)
		void showMessage()const;
```

### 4.只有继承自Actor类的类才可以具有组件

组件实质上也是一个类的成员

组件必须要在C++类里提前实例化,构造器里面初始化他

#### 4.1定义一个自己的组件:

```c++
UPROPERTY(EditAnywhere, BlueprintReadWrite)
		UStaticMeshComponent* SphereMesh;
```

```c++
SphereMesh = CreateDefaultSubobject<UStaticMeshComponent>("SphereBase");//在构造函数里面实例化它,不要用->
```

#### 4.2把一个组件附加到一个组件上

```c++
Camera->SetupAttachment(CameraArm);//将一个组件附加到一个组件下方(调用的附加到被调用)
```



### 4.2宏

#### 4.2.1标准C++定义宏

​    可以理解为简单的字符串替换，如 #include <xxx.h> 实际上是把头文件 xxx.h的内容展开，然后替换到声明处而已。

```c++
// xxx.h
int a = 1;
int b = 2;
```

```c++
int func()//与下面那个等效
{
    #include "xxx.h"
    return a + b;
}
```

```C++
int func()
{
    int a = 1;
	int b = 2;
    return a + b;
}
```

注意宏替换是简单的字符串替换，不会进行任何运算。比如想用宏定义一个求平方数的函数，以下的宏定义就是错误的：

```c++
#define area(x) x*x  

void main()
{
    int y = area(2+2);//本该输出16，实则输出8
    printf("%d",y);
}
```

正确的宏定义如下所示：

```c++
#define area(x) ((x)*(x)) 
```

#### 4.2.2.2UE4中的C++

1. UE的特殊宏
   由于C++没有反射和垃圾回收机制，UE封装了一套以宏为基础的反射与垃圾回收架构。

**UPROPERTY：**

**UFUNCTION：**

**UCLASS：**

**USTRUCT :**

**GENERATED_BODY:**

### 4.3获取玩家输入

```c++
void ASphereBase::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)//每个Pawn都自带的
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);
	PlayerInputComponent->BindAxis("MoveForward", this, &ASphereBase::MoveForward);//第一个是在引擎里面自己在输入里面定义的轴映射的名字(string),第二个参数是调用这个函数的对象，第三个是要调用的函数符的地址，轴向绑定玩家输入
    PlayerInputComponent->BindAction("SpeedUp", IE_Pressed, this, &ASphereBase::SpeedUp);
}
PlayerInputComponent->BindAction("SpeedUp", IE_Released, this, &ASphereBase::SpeedLow);
```

蓝图的实现

![image-20220712091037421](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712091037421.png)

### 4.4给pawn一个角速度

```c++
SphereMeshComp->SetPhysicsAngularVelocityInDegrees(FVector(100.0f,0,0));//分XY轴
```

#### 4.4.1给一个角速度的缓冲

```c++
SphereMeshComp->SetAngularDamping(2.5f);
```

### 4.5创建一个碰撞盒子

```c++
UPROPERTY(EditAnywhere,BlueprintReadWrite)
    class UBoxComponet*HitBox;//盒体组件
HitBox=CreateDefaultSubobject<UBoxComponent>("HitBox");
```

#### 4.5.1把盒子和重叠事件相关联(等同于蓝图把他们连起来)

```c++
HitboxComp->OnComponentBeginOverlap.AddDynamic(this,&AHitBox::BeginHit);
void BeginHit(UPrimitiveComponent* OverlappedComponent, AActor*OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, 
		bool bFromSweep, const FHitResult& SweepResult);//参数列表在OnComponentBeginOverLap的源码里
```

```C++
DECLARE_DYNAMIC_MULTICAST_SPARSE_DELEGATE_SixParams( FComponentBeginOverlapSignature, UPrimitiveComponent, OnComponentBeginOverlap, <font color='red'>UPrimitiveComponent*, OverlappedComponent, AActor*, OtherActor, UPrimitiveComponent*, OtherComp, int32, OtherBodyIndex, bool, bFromSweep, const FHitResult &, SweepResult</font>);//源码里的宏
```

```c++

Cast<class>(value);//类型转换
```

### 5.在C++类里面创建一个文件夹

编译器发挥正常

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220714101259093.png" alt="image-20220714101259093" style="zoom:50%;" />

#### 5.1在资源管理器里面创建

在项目文件里面的Source文件目录里的项目名文件夹里面创建(记得往里面放东西，不然不会显示)

#### 5.2对[项目名].uproject右键点击Generate Visual Studio

project files

超常发挥：

![image-20220714101812677](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220714101812677.png)



正常发挥：什么都没有

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220714102024904.png" alt="image-20220714102024904" style="zoom:50%;" />

#### 5.1解决方案

cmd界面找到Unreal Build Tool.exe(路径: "D:\UE_4.26.2\UE_4.26\Engine\Binaries\DotNET\UnrealBuildTool.exe")

<font color='green'>//注意引擎版本</font>

输入并运行下面代码: UnrealBuildTool.exe -projectfiles -project="E:\UE_ProgramLoad\Clearn_r\Clearn_r.uproject" -game -rocket -progress

进入VS界面会弹窗是否重载，点击重载，虚幻编译器点击编译，文件夹显示

#### 5.2删除一个C++类

在项目文件里面删除，再出去删掉项目文件里面的binarys文件，然后右键项目.uproject文件点Generate Visual Studio project files(找不到看上面)

### 6.小技巧

#### 6.1摄像机

把SpringArm组件的Do collision test(碰撞测试)关闭，可以让SpringArm不会被物体阻挡而进行伸缩

## 3.基础概念(3+2)

**在UE4中，1虚幻单位（UU）等于1厘米（cm）。**

**在虚幻引擎中，Z轴 为纵轴。**

虚幻引擎术语表：[虚幻引擎 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/unreal-engine-glossary/)

###  5.1Actor

​       **Actor** 是可以放到关卡中的任何object

​       在C++中，`AActor` 是所有Actor的基类

### 5.2Pawn(NPC)

​    继承自Actor

​    赋予了移动功能

### 5.3character

​    继承自Pawn

​    赋予了与场景互动功能

​    角色子类包括碰撞设置、双足运动的输入绑定以及用于玩家控制动作的其他代码

### 5.4投射

   **投射（Casting）** 是一种动作，将会提取特定类的Actor并尝试将其作为其他类进行处理。投射可能成功，也可能失败。如果投射成功，则可以在你投射到的Actor上访问特定于类的功能

   例如，如果你要制作一款游戏，在其中具有能够以不同方式影响玩家角色的多种体积类型。其中一个体积是 **火焰（Fire）**，可以随着时间降低玩家血量。当角色与关卡中的任何体积重叠时，就可以将该体积 **投射（Cast）** 到 **火焰（Fire）** 上，以尝试访问其"损害玩家血量"功能

### 5.5组件

 **组件（Component）** 是一种可以添加到Actor的功能。

将组件添加到Actor时，Actor可以使用组件提供的功能

### 5.6玩家控制器

**玩家控制器（Player Controller）** 获取玩家输入，并将其转换到游戏内的互动中。每个游戏内部都至少具有一个玩家控制器。玩家控制器通常操控一个Pawn或角色作为玩家在游戏中的呈现方式

关联的C++类是 `PlayerController`

### 5.7AI控制器

**AI控制器（AI Controller）** 操控Pawn在游戏中呈现非玩家角色（NPC）。默认情况下，Pawn和角色都以基本AI控制器终结，除非它们被玩家控制器专门操控或者收到指令不允许为自己创建AI控制器

关联的C++类是 `AIController`

### 5.8玩家状态(玩家数据)

**玩家状态（Player State）** 是游戏参与者在游戏中的状态，例如人类玩家或模拟玩家的机器人

玩家状态可能包含的玩家信息示例包括：

- 名称
- 当前级别
- 血量
- 得分
- 它们当前是否在"夺旗"游戏中扛旗。

### 5.9游戏模式

**游戏模式（Game Mode）** 设置要运行的游戏的规则。这些规则可以包括：

- 玩家如何加入游戏。

- 游戏是否可以暂停。

- 任何游戏特定行为，例如获胜条件。

- 可以在

  [项目设置](understanding-the-basics/projects-templates/unreal-engine-project-settings)

  中设置默认游戏模式，并针对不同的关卡重载游戏模式。无论你选择以何种方式实施，每个关卡都只能有一个游戏模式。

  

  在多人游戏中，游戏模式新存在于服务器上，而规则将复制（发送）到每个连接的客户端。

  关联的C++类是 `GameMode`。

### 5.10游戏状态

**游戏状态（Game State）** 是一个容器，包含你要在游戏中复制到每个客户端的信息。简而言之，它是每个连接的人的"游戏状态"。

游戏状态可能包含的内容示例包括：

- 与游戏得分相关的信息。
- 比赛是否开始。
- 根据世界中的玩家数量确定生成AI角色的数量。

对于多人游戏，每个玩家的机器上都有一个本地游戏状态实例。本地游戏状态实例从游戏状态的服务器实例获取更新的信息。

关联的C++类是 `GameState`。

### 5.11笔刷(staticmesh)

  **笔刷（Brush）** 是用于描述3D形状的Actor，例如立方体或球体。可以将笔刷放置在关卡中以定义关卡几何体（这些几何体称为二进制空间分区或BSP笔刷）。例如，如果要快速封锁关卡，此功能非常有用

### 5.12体积(触发事件)

**体积（Volumes）** 是带有边界的3D空间，根据连接到体积的效果，具有不同的使用方法。例如：

- **阻挡体积（Blocking Volumes）** 是可见的，用于阻止Actor通过它们。
- **施加伤害体积（Pain Causing Volume）** 对与其重叠的任何Actor造成持续伤害。
- **触发器体积（Trigger Volumes）** 的编程方式为，在Actor进入或退出体积时触发事件。

### 5.13关卡

**    关卡（Level）** 是你定义的gameplay区域。关卡包含玩家可以看到并与其交互的所有内容，例如几何体、Pawn和Actor。

​         虚幻引擎将每个关卡保存为单独的 `.umap` 文件，这也是为什么你在某些情况下会看到它们被称为 **地图（Maps）** 。

### 5.14世界

**世界（World）** 是构成游戏的所有关卡的容器。它处理关卡的流送和动态Actor的生成（创建）

### 5.2.1空间

   

1. | **虚幻中的空间**                    | **其他名称**                                                 | **描述**                                                     |                                     |
   | ----------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------- |
   | **切线（Tangent）**                 |                                                              | 正交（可能在插值之后偏离），可以是左旋或右旋。TangentToLocal变换仅包含旋转，因此它是OrthoNormal（可以通过换位反转）。 |                                     |
   | **局部（Local）**                   | **对象空间（Object Space）**                                 | 正交，可以是左旋或右旋（三角形剔除顺序需要调整）。LocalToWorld变换包含旋转、非等分缩放（包括可能改变缠绕顺序的负非等分缩放）和平移。 |                                     |
   | **世界场景（World）**               |                                                              | WorldToView变换仅包含旋转和平移，因此视图空间中的距离与世界场景空间中的距离相同。 |                                     |
   | **平移世界场景（TranslatedWorld）** | 世界场景                                                     | 世界场景 - 预览平移                                          |                                     |
   |                                     | 平移世界场景                                                 | 世界场景 + 预览平移                                          |                                     |
   |                                     |                                                              | 平移的矩阵用于从组合的变换矩阵中移除摄像机位置，可在变换顶点时提高精度。 |                                     |
   | **视图（View）**                    | ***摄像机空间（CameraSpace）****                             | ViewToClip变换包含x轴和y轴上的缩放，但不包含平移（如果平移的话将会是偏心投影）。它缩放并平移z轴。它还会应用投影来转换为齐次裁剪空间。 |                                     |
   | **裁剪（Clip）**                    | **齐次坐标（HomogeniousCoordinates）**, **后投影空间（PostProjectionSpace）**, **投影空间（ProjectionSpace）** | 应用透视投影矩阵之后。请注意，裁剪空间中的W与视图空间Z中的相同。 |                                     |
   | **屏幕（Screen）**                  | OpenGL 中的 **标准化设备坐标（NormalizedDeviceCoordinates）** | 经过透视分割之后：                                           |                                     |
   |                                     |                                                              | 左/右                                                        | -1,1                                |
   |                                     |                                                              | 上/下                                                        | 1,-1                                |
   |                                     |                                                              | 近/远                                                        | 0,1（OpenGL RHI需要将此变换为-1,1） |
   | **视口（Viewport）**                | **视口坐标（ViewportCoordinates）**、**窗口坐标（WindowCoordinates）** | 以像素计：                                                   |                                     |
   |                                     |                                                              | 左/右                                                        | 0, 宽-1                             |
   |                                     |                                                              | 上/下                                                        | 0, 高-1                             |

### 5.2.2工具和编辑器

- **工具** 即你用来执行特定任务的用具，如在关卡中放置Actor，或绘制地形。

- *编辑器** 即你用来实现更复杂目标的工具集合。例如， **关卡编辑器** 可让你构建游戏关卡，或者你可以在 **材质编辑器** 中改变材料的外观体验。

- **系统** 是功能大合集，这些功能会协同产生游戏或应用程序各方面内容。例如， **蓝图** 是用于视觉化脚本Gameplay元素的系统。

  

#### 5.2.2.1关卡编辑器



​    **关卡编辑器** 是你构建Gameplay关卡的主要编辑器。在这里通过添加不同类型的[Actor和几何体](https://docs.unrealengine.com/5.0/zh-CN/actors-and-geometry-in-unreal-engine)、[蓝图可视化脚本](https://docs.unrealengine.com/5.0/zh-CN/blueprints-visual-scripting-in-unreal-engine)、[Niagara](https://docs.unrealengine.com/5.0/zh-CN/overview-of-niagara-effects-for-unreal-engine)等来定义播放空间。在默认情况下，当你创建或打开项目时，虚幻引擎5会打开关卡编辑器。

如需了解更多信息，请参阅[关卡编辑器](https://docs.unrealengine.com/5.0/zh-CN/level-editor-in-unreal-engine)

#### 5.2.2.2材质编辑器



   **材质编辑器** 是你创建和编辑材质的地方。***材质是可应用于网格体以控制其视觉效果的资产***。例如，你可以创建污垢材质，并将其应用到关卡中的各个地板上，从而创建看似有污垢覆盖的表面。

如需了解详细信息，请参阅[材质编辑器参考](https://docs.unrealengine.com/5.0/zh-CN/unreal-engine-material-editor-user-guide)。

#### 5.2.2.3蓝图编辑器

​    **蓝图编辑器** 是你使用和修改蓝图的地方。这些特殊资产可用来创建Gameplay元素（如控制Actor或对事件编写脚本），修改材质或执行其他虚幻引擎功能，省去编写任何C++代码的过程。

如需更多信息，请参阅[蓝图编辑器参考](https://docs.unrealengine.com/5.0/zh-CN/user-interface-reference-for-the-blueprints-visual-scripting-editor-in-unreal-engine)。

#### 5.2.2.4物理资源编辑器

你可以使用 **物理资产编辑器** 创建物理资产，以配合[骨骼网格体](https://docs.unrealengine.com/5.0/zh-CN/skeletal-meshes)使用。在实践中，你可以使用此方法实现变形和碰撞等物理特性。你可以从零开始，构建完整的布娃娃设置，或使用自动化工具来创建一套基本物理形体和物理约束。

如需更多信息，请参阅[物理资产编辑器](https://docs.unrealengine.com/5.0/zh-CN/physics-asset-editor-in-unreal-engine)。

#### 5.2.2.5静态网格体编辑器

可以使用 **静态网格体编辑器（Static Mesh Editor）** 来预览外观、碰撞和UV贴图，以及设置和操控[静态网格体](https://docs.unrealengine.com/5.0/zh-CN/static-mesh-actors-in-unreal-engine)。在静态网格编辑器中，你也可以针对你的静态网格体资产设置[LOD](https://docs.unrealengine.com/5.0/zh-CN/creating-and-using-lods-in-unreal-engine)（或细节级别设置），以根据你的游戏运行方式和地点控制静态网格体资产出现的简洁程度或详细程度。

如需更多信息，请参阅[静态网格体编辑器UI](https://docs.unrealengine.com/5.0/zh-CN/static-mesh-editor-ui-in-unreal-engine)。

#### 5.2.2.6行为树编辑器

**行为树编辑器** 是你通过一种可视化的基于节点脚本系统（类似于蓝图）为关卡中的Actor编写人工智能（AI）脚本的地方。你可以为敌人、非游戏角色（NPC）、载具等创建任意数量的不同行为。

如需更多信息，请参阅[行为树用户指南](https://docs.unrealengine.com/5.0/zh-CN/behavior-tree-in-unreal-engine---user-guide)。

#### 5.2.2.7Niagara编辑器

**Niagara编辑器** 利用由分离粒子发射器组成的全套模块化粒子效果系统，为每个效果创建特殊效果。可将发射器保在内容浏览器中，以备后用，并将作为当前和未来项目中的新发射器基础使用。

如需了解详细信息，请参阅[Niagara关键概念](https://docs.unrealengine.com/5.0/zh-CN/key-concepts-in-niagara-effects-for-unreal-engine)。

#### 5.2.2.8UMG界面编辑器

**虚幻示意图形UI编辑器** 是视觉UI创作工具，可用来创建UI元素，如在游戏内头顶显示、菜单或其他界面相关的图形。

如需更多信息，请参阅[UMG UI设计器用户指南](https://docs.unrealengine.com/5.0/zh-CN/umg-ui-designer-quick-start-guide)。

#### 5.2.2.9字体编辑器

使用 **字体编辑器** 添加、组织和预览字体资产。你也可以定义字体参数，如字体资产布局和提示策略（*字体提示*是一种数学方法，可确保文本在任意尺寸的显示屏中都可读）。

如需更多信息，请参阅[字体资源和编辑器](https://docs.unrealengine.com/5.0/zh-CN/font-asset-and-editor-in-unreal-engine)。

#### 5.2.2.10Sequencer编辑器(动画)

利用 **Sequencer编辑器** 可通过专用多轨迹编辑器创建游戏过场动画。通过创建 **关卡序列（Level Sequences）** 和添加 **轨迹** （Tracks），你可以定义各个轨迹的组成，这样将确定场景的内容。轨迹可以包含**动画**（Animation）（*用于将角色动画化*）、**变形**（Transformation）（*在场景中移动各个东西*）、**音频**（Audio）（*用于包括音乐或音效*）等等。

如需了解更多信息，请参阅[Sequencer概述](https://docs.unrealengine.com/5.0/zh-CN/unreal-engine-sequencer-movie-tool-overview)。

#### 5.2.2.11动画编辑器

**动画编辑器** 是虚幻引擎5中的动画编辑器。你可以使用该工具来编辑[骨骼资产](https://docs.unrealengine.com/5.0/zh-CN/skeletons-in-unreal-engine)、[骨骼网格体](https://docs.unrealengine.com/5.0/zh-CN/skeletal-meshes)、[动画蓝图](https://docs.unrealengine.com/5.0/zh-CN/animation-blueprints-in-unreal-engine)，以及其他各种动画资产。

如需更多信息，请参阅[动画编辑器](https://docs.unrealengine.com/5.0/zh-CN/animation-editors-in-unreal-engine)。

#### 5.2.2.12Control Rig编辑器

**Control Rig** 是动画工具套件，可以用于直接在引擎中操纵角色并实现其动画。使用Control Rig，你无需在外部工具中进行操纵和制作动画，而是***直接在虚幻编辑器中制作动画***。使用此系统，你可以**在角色上创建和操纵自定义控制点**，在 [Sequencer](https://docs.unrealengine.com/5.0/zh-CN/cinematics-and-movie-making-in-unreal-engine) 中制作动画，并使用各种其他动画工具来帮助完成动画制作过程。

如需了解更多信息，请参阅[控制绑定](https://docs.unrealengine.com/5.0/zh-CN/control-rig-in-unreal-engine)。

#### 5.2.2.13Sound Cue编辑器(音频)

虚幻引擎5中的音频播放的行为在Sound Cue中得到定义，可使用 **Sound Cue编辑器** 对其进行编辑。在此编辑器中，你可以组合多个声音资产后混音，以此生成单混音输出，另存为一个Sound Cue。

如需了解更多信息，请参阅[Sound Cue编辑器](https://docs.unrealengine.com/5.0/zh-CN/sound-cue-editor-in-unreal-engine)。

#### 5.2.2.14媒体编辑器

#### 外部媒体播放

使用 **媒体编辑器** 来定义媒体文件或URL，以作为虚幻引擎5内部播放的源媒体使用。

你可以定义源媒体播放方式设置，如自动播放、播放速度和循环，但不能直接编辑媒体。

如需了解更多信息，请参阅[媒体编辑器参考文档](https://docs.unrealengine.com/5.0/zh-CN/media-editor-reference-for-unreal-engine)。

#### 5.2.2.15nDisplay 3D配置编辑器

#### 虚拟制作和实时事件

[nDisplay](https://docs.unrealengine.com/5.0/zh-CN/rendering-to-multiple-displays-with-ndisplay-in-unreal-engine)在多个同步显示设备上渲染虚幻引擎场景，如能量墙、穹顶和曲面界面。你可以使用 **nDisplay配置编辑器** 创建nDisplay设置，并使所有显示设备上的内容渲染方式可视化。

如需了解更多信息，请参阅[nDisplay 3D配置编辑器](https://docs.unrealengine.com/5.0/zh-CN/ndisplay-3d-config-editor-in-unreal-engine)。

#### 5.2.2.16DMX库编辑器

#### 实时事件

[![DMX实际效果。此截图来自Moment Factory的示例项目。](ue4-dmx-library-editor.png "DMX in action.此截图来自Moment Factory的示例项目。)(w:600)](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/foundational-knowledge/tools-editors/ue4-dmx-library-editor.png)

DMX实操。此截图来自Moment Factory的[示例项目](https://www.youtube.com/watch?v=UXodNAcyFj8)。点击查看完整视图。

**DMX（数字多路复用）** 是在整个实时事件行业中用来控制各种设备的数字通信标准，如照明灯具、激光、烟雾机、机械设备和电子广告牌。在 **DMX库编辑器** 中，你可以自定义相关设备及其命令。

如需了解更多信息，请参阅[DMX](https://docs.unrealengine.com/5.0/zh-CN/dmx-in-unreal-engine)。

## 4.内容浏览器

### 4.1访问

![从Windows菜单打开内容浏览器](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/content-browser/content-browser-windows-menu.jpg)

### 4.2大致

![内容浏览器窗口的区域](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/content-browser/interface/content-browser-ui-areas.png)



| 1    | 导航栏（Navigation Bar）           |
| ---- | ---------------------------------- |
| 2    | 源面板（Sources Panel）            |
| 3    | 集合（Collections）                |
| 4    | 搜索和筛选器（Search and Filters） |
| 5    | 资产视图（Asset View）             |
| 6    | 设置按钮（Settings Button）        |



### 4.3源面板

#### 4.3.1小技巧

使用以下鼠标和按键组合来选择多个文件夹。

- **左键点击** 可将当前所选项替换为你点击的文件夹。

- **Shift + 左键点击** 可选择开始和结束点之间的一系列文件夹。

- **Ctrl + 左键点击** 可选择或取消选择单个文件夹。

- **高级复制到此处（Advanced Copy Here）**

  在目标文件夹中创建所选文件夹及其依赖性的副本。还将修复因移动而破坏的所有依赖性

#### 4.3.2右键菜单

[虚幻引擎中的源面板参考 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/sources-panel-reference-in-unreal-engine/)（源面板）

### 4.4高级搜索语法

在[内容浏览器](https://docs.unrealengine.com/5.0/zh-CN/content-browser-in-unreal-engine)中可以使用高级搜索运算符来查找内容。这些运算符可以进行更细致的搜索，让你更快地找到需要的内容。你还可以用它来搜索资产元数据的"键-值"对，并访问特殊键值。

| **Equal（二元运算）**           | = ` `== ` `: | 判断指定键的返回值是否等于指定值。                           |           Name="Blast"` `Name==Blast` `Name:Bla...           |
| ------------------------------- | ------------ | ------------------------------------------------------------ | :----------------------------------------------------------: |
| **NotEqual（二元运算）**        | !=` `!:      | 判断指定键的返回值是否不等于指定值。                         |                 Name!=Blast` `Name!:"Blast"                  |
| **Less（二元运算）**            | <            | 判断指定键返回的值是否小于指定值。该运算符仅支持数字类型的值。 |                         Triangles<92                         |
| **LessOrEqual（二元运算）**     | <=` `<:      | 判断指定键返回的值是否小于等于指定值。该运算符仅支持数字类型的值。 |                Triangles<=92` `Triangles<:92                 |
| **Greater（二元运算）**         | >            | 判断指定键返回的值是否小于等于指定值。该运算符仅支持数字类型的值。 |                Triangles<=92` `Triangles<:92                 |
| **GreaterOrEqual（二元运算）**  | >=` `>:      | 判断指定键返回的值是否大于等于指定值。该运算符仅支持数字值。 |                Triangles>=92` `Triangles>:92                 |
| **Or（二元运算）**              | OR` `||` `\| | 测试两个值，任意一个为 `true` 即返回 `true`。                | Blast OR Type:Blueprint` `!Blast || Path:Testing` `Name:"Blast" \| Path:Testing... |
| **And（二元运算）**             | AND` `&&` `& | 测试两个值，两个值均为 `true` 则返回 `true`。                | Blast AND Type:Blueprint` `!Blast || Path:Testing` `Name:"Blast" \| Path:Testing... |
| **Not（一元运算）**             | NOT` `!      | 测试运算符后面的值，返回反转结果。                           |                    NOT Blast` `! "Blast"                     |
| **TextCmpInvert（一元运算）**   | -            | 修改文本值，以便返回其所参与的运算的反转结果。               |                      -Blast` `-"Blast"                       |
| **TextCmpExact（一元运算）**    | +            | 修改文本值，以便执行"精确"文本比较。                         |                      +Blast` `+"Blast"                       |
| **TextCmpAnchor（一元运算）**   | ...          | 修改文本值，以便执行"结尾"文本比较。                         |                      ...ast` `..."ast"                       |
| **TextCmpAnchor（后一元运算）** | ...          | 修改文本值，以便执行"开头"文本比较。                         |             修改文本值，以便执行"开头"文本比较。             |

#### 4.4.1特殊健

大多数可用于搜索的键来自于从资产注册表提取的资产元数据（Asset metadata）。不过，有几个特殊键适用于所有资产类型。这些特殊键仅支持 **`Equal` 或 `NotEqual`** 比较运算符。

| **Name**   | N/A  | 资产名称                 |
| ---------- | ---- | ------------------------ |
| Path       | N/A  | 资产路径                 |
| Class      | Type | 资产类                   |
| Collection | Tag  | 包含资产的任何集合的名称 |

#### 4.4.2字符串

   字符串可以带引号（单引号或双引号），也可以不带引号。带引号的字符串可以包含嵌套引号；但是，必须使用反斜杠（\）表示嵌套引号结束。使用无引号和带引号字符串的主要差异在于带引号字符串允许在搜索词中使用空格和特殊字符。默认情况下，它们将执行部分字符串匹配，除非使用了 `TextCmpExact` 或 `TextCmpAnchor` 运算符来修改此行为。

```c++
以下是使用单引号和双引号以及反斜杠的部分示例：

"Foo\"bar"  ->  Foo"bar
'Foo\'bar'  ->  Foo'bar
"Foo\'bar"  ->  Foo'bar
'Foo\"bar'  ->  Foo"bar
"Foo\\bar"  ->  Foo\bar
'Foo\\bar'  ->  Foo\bar
```

#### 4.4.3资产元数据

![纹理资产元数据示例](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/content-browser/advanced-search-syntax/example-asset-metadata.jpg)

你可以使用任意一种元数据来搜索带有该特征的资产。搜索时应使用以下语法：

**[元数据名称] [运算符] [字符串或数字值]** 

例如：

```c++
Triangles>=10500
Type==Skeletal
UVChannels>2 
CollisionPrims!=0
```

**高级搜索示例**

```C++
BlendMode==Translucent AND ShadingModel==DefaultLit
```

## 5.自定义引擎

### 5.1插件

####   5.1.1插件启用

​       在主菜单中，前往 **编辑（Edit）> 插件（Plugins）** 。这将打开 **插件   （Plugins）** 窗口。

![虚幻引擎5中的"插件"窗口](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/customizing-unreal-engine/working-with-plugins/ue5-plugins-window.jpg)

####     5.1.2插件安装位置

- Windows上的 `C:\Program Files\Epic Games\UE_[version]\Engine\Plugins`
- macOS上的 `/Users/Shared/Epic Games/UE_[version]/Engine/Plugins`

### 5.2自定义按键快捷键

**按键快捷键**，也叫做 **键绑定**，本质上是一种组合按键，可以执行特定命令或操作。你可以为一些常用命令和工具设置快捷键，以便满足你的个人习惯。设置方法如下：打开 **编辑器偏好设置** 窗口，在主菜单中，进入 **编辑 > 编辑器偏好设置**，然后选择 **键盘快捷键**。

![image-20220711171021615](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220711171021615.png)

### 5.3编辑器偏好设置

[虚幻引擎编辑器偏好设置 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/unreal-editor-preferences/)

## 6.使用项目和模板

### 6.1自定义模板(*)

​     自定义模板是一个虚幻引擎项目，设置为在创建新项目时作为模板和其它的模板一起在引擎中显示。

​     自定义模板可以包含内容、设置和代码，并且可以设置默认启用或禁用指定的插件。

要将已有的项目转换为模板：

1. 将 **整个** 项目文件夹复制到你的引擎安装目录的 `Templates` 文件夹啊。如果你从Epic Games启动器安装了虚幻引擎，那么 `Templates` 文件夹会位于：

   - Windows系统： ***`C:\Program Files\Epic*** Games\UE_[version]\Templates`
   - Mac： `/Users/Shared/Epic Games/UE_[version]/Templates`

   如果你从源代码编译了虚幻引擎，那么 `Templates` 文件夹会位于 `[ForkLocation]\UE4\Templates` 。

2. 打开 `[ProjectName]\Config\DefaultGame.ini 文件` 。然后添加或更新 **ProjectName** 变量。这是在创建新项目时显示在模板选择区域内的名称。

   示例：

```c++
[/Script/EngineSettings.GeneralProjectSettings]
  ProjectID=E6468D0243A591234122E38F92DB28F4
  ProjectName=MyTestTemplate//主要改这个
```

​       注意 **ProjectID** 变量是为每一个项目生成的一个独特的ID。

​      3.在你的虚幻引擎安装目录中，找到 `Templates\TP_FirstPerson\Config\` 。将 `TemplateDefs.ini` 文件复制到 `[ProjectName]\Config` 文件夹。//除了 `TP_FirstPerson` 以外，你还可以使用任何已有的模板文件夹，只要其中包含一个 `TemplateDefs.ini` 文件即可。

4.打开上一步复制的 `TemplateDefs.ini` 文件并且更新 **LocalizedDisplayNames** 和 **LocalizedDescriptions** 变量。一共有四组变量，每组对应一个虚幻引擎支持的语言：英文 (en)、韩文 (ko)、日文 (ja) 以及简体中文 (zh-Hans)。

示例：

```c++
[/Script/GameProjectGeneration.TemplateProjectDefs]
    LocalizedDisplayNames=(Language="en",Text="My Test Template")
    LocalizedDescriptions=(Language="en",Text="This is a custom template that includes a first-person character and uses Blueprint.")
```

5.

当你创建新项目时，你的模板会在 `TemplateDefs.ini` 文件中定义的类别里出现。这由 **Categories** 变量所控制。不管变量名称是什么，一个模板只能分配 **一个** 类别。

可以使用的类别有：

- **Games** - 游戏
- **ME** - 电影、电视和现场活动
- **AEC** - 建筑、工程和建造
- **MFG** - 汽车、产品设计和生产

更多信息可以参考 `[虚幻引擎安装目录]\UE_[版本]\Templates` 文件夹下的 `TemplateCategories.ini` 文件。

6.你可以在 `[ProjectName]\Media` 文件夹中添加一个图标和预览图。这些图片必须为PNG格式并且满足以下命名要求：(自己创建一个Media文件)

- 图标： `[ProjectName].png`.
- 预览图： `[ProjectName]_Preview.png`.

## 7.资产和内容包

### 7.1导入资产

import按钮

直接拖入

### 7.2使用资产

#### 7.2.1查看资产引用

要查看资产的引用，请在 **内容浏览器** 中右键单击该资产。然后，从出现的上下文菜单中，选择 **引用查看界面**。将打开一个新窗口，显示资产引用的视觉效果

![image-20220711200154926](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220711200154926.png)

#### 7.2.2复制资产引用

要将一个或多个资产的引用复制到剪贴板，请在 **内容浏览器** 中选择一个或多个资产。然后，右键单击你的选择，然后从出现的上下文菜单中选择 **复制引用（Copy Reference）**。

引用包含资产类型和 `.uasset` 文件的路径。类似于以下示例所示：

```C++
Material'/Game/StarterContent/Materials/M_Metal_Brushed_Nickel.M_Metal_Brushed_Nickel'
    Material'/Game/StarterContent/Materials/M_Metal_Burnished_Steel.M_Metal_Burnished_Steel'
    Material'/Game/StarterContent/Materials/M_Metal_Chrome.M_Metal_Chrome'
```

#### 7.2.3替换引用工具

**替换引用工具** 提供了一种将多个资产组合为一个资产的方法。

有关此工具的更多信息，请参阅[替换引用工具](https://docs.unrealengine.com/5.0/zh-CN/consolidating-assets-in-unreal-engine)页面。

#### 7.2.4管理资产

| 操作                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **在文件夹视图中显示（Show in Folder View）**                | 在文件夹树中高亮显示资产的父文件夹。这对查找属于集合的资产的实际位置很有用。 |
| **在资源管理器中显示（Show in Explorer）（Windows）/在Finder中显示（ Show in Finder） (Mac)** | 从资产在磁盘上的位置打开一个Windows资产管理器或Finder的实例。这是 `.uasset` 文件的位置。 |

**切勿直接在磁盘上移动、复制或删除资产，这可能会破坏项目中的功能并导致数据损坏或丢失。应始终通过虚幻编辑器管理 `.uasset` 文件。**

**如果你要将资产从一个项目移动到另一个项目，请参阅[迁移资产](https://docs.unrealengine.com/5.0/zh-CN/migrating-assets-in-unreal-engine)页面了解如何执行此操作。**

### 7.3迁移资产

###### 如何将资产从一个项目复制到另一个项目。

如果想在多个项目中使用相同的资产，可以使用 **迁移工具** 复制资产及其引用和依赖项。例如，如果迁移材质，任何定义该材质的纹理资产都将自动与材质一起复制。当需要合并或派生项目，或者从测试环境过渡到生产项目时，此方法非常有用。

要使用迁移工具，请按照以下步骤操作：

1. 从 **内容浏览器（Content Browser）** 中选择要迁移的资产。

![Selecting Assets to migrate from the Content Browser](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/assets-content-packs/migrating-assets/select-assets-to-migrate.jpg)

   2.**右键单击** 任意一项选择的资产。在出现的上下文菜单中，选择 **资产操作（Asset Actions）>迁移（Migrate）**。

这样将打开 **资产报告（Asset Report）** 窗口，其中显示即将在迁移过程中复制的所有资产。

![Asset Report window](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/assets-content-packs/migrating-assets/asset-report.jpg)

​       如果不想迁移此列表中的某项资产，请取消选择该资产旁边的复选框。需要注意的是，这样可能会破坏尝试迁移的其他资产（例如，如果一种材质的某个纹理缺失，则该材质将无法正确显示）。

​     3.单击 **确定（OK）** 确认你希望迁移资产。这样将打开一个文件浏览器窗口，你可以在其中选择要接受资产迁移的项目（也称为目标项目）。

![Selecting a destination folder](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/assets-content-packs/migrating-assets/select-destination.jpg)

选择目标项目中的 `Content` 文件夹，然后单击 **选择文件夹（Select Folder）** 按钮。

4.在确认迁移后，将显示一个进度条，用来跟踪迁移进度。

如果目标项目的 `Content` 文件夹中包含与要迁移的资产同名的资产，你将看到以下警告消息：

![An asset already exists at (location), would you like to overwrite it?](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/assets-content-packs/migrating-assets/asset-migration-warning.jpg)

### 7.4资产元数据

​        你可以将元数据（Metadata）指定给虚幻引擎项目中的任何资产，以便记录资产的信息。元数据是一组键值对，可以根据用途自由定义。

元数据可以包含这些信息：资产创建者姓名、资产在项目中的预期用途、资产在工作流程中的状态（例如正在进行、已完成、已批准等）等等。

你可以用元数据筛选内容浏览器中的资产，或者识别蓝图或Python脚本中的资产。

#### 7.4.1在虚幻编辑器UI中使用元数据

​      虽然目前无法在UI虚幻编辑器中修改元数据，但你可以查看与资产绑定的元数据，并且可以使用元数据的键来筛选在内容浏览器中显示的资产。

#### 7.4.2查看资产上的元数据(为什么都是灰的)

要查看分配给任何资产的元数据，请在内容浏览器中右键单击该资产，并选择 **资产操作（Asset Actions）> 显示元数据（Show Metadata）**。

![Show Metadata in Unreal](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/assets-content-packs/asset-metadata/fbx-metadata-show-menu.jpg)

#### 7.4.3过滤内容浏览器

要在内容浏览器中按特定的元数据标签过滤资产，请执行以下操作：

1. 在主菜单中选择 **编辑（Edit）> 项目设置（Project Settings）**，打开 **项目设置（Project Settings）** 窗口。

2. 选择 **游戏（Game）> 资产管理器（Asset Manager）** 部分，然后找到 **资产注册表（Asset Registry）> 资产注册元数据标签（Metadata Tags For Asset Registry）** 设置。 将你希望能够被用于过滤资产的所有键的名称添加到此列表中。

   ![image-20220711205500145](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220711205500145.png)

​    3.在内容浏览器的 **过滤（Filters）** 栏中，键入标签名称，后跟`=`，再后跟要搜索的值。资产列表将自动进行过滤，仅显示包含你指定的元数据标签的资产，对于这些资产，该标签的值与你在`=`后面键入的值匹配。

![Filter the Content Browser by metadata](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/assets-content-packs/asset-metadata/fbx-metadata-content-browser-filter.jpg)

7/11

### 7.5自动重新导入

借助虚幻引擎4的自动重新导入功能，当你在外部程序中工作时，所有改动内容可以自动同步到虚幻引擎4中，无需用户执行任何处理。当你需要迭代开发资产并立即查看效果时，该功能能大幅提高你的开发效率。

UE4会对一组文件夹中的内容进行监视（该目录由用户定义），并且检测该源内容是否发生变化。当某个源文件发生改动，并且该文件被用于导入资产到游戏中，则虚幻4会自动将改动后的文件重新导入。

![image-20220712094121252](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712094121252.png)

详情[自动重新导入 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/reimporting-assets-automatically-in-unreal-engine/)

### 7.6查看资产引用

##### 搜索选项

在"引用查看器（Reference Viewer）"的左上角，你可以看到虚幻编辑器用于构建图表的两个搜索相关选项。

![image-20220712095340160](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712095340160.png)

| 项目                                     | 说明                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| **搜索深度限制（Search Depth Limit）**   | 引擎搜索引用的深度。例如，如果值为2，则图表不仅会显示与选中的资产相关的资产，还会显示与那些相关的资产相关的资产。 |
| **搜索广度限制（Search Breadth Limit）** | 给定列中列出的引用（引用或被引用）的数量。例如，如果某个资产引用了20个资产，但是由于"搜索广度限制（Search Breadth Limit）"设置为10，该列中仅显示10个资产。 |

### 7.7替换引用工具(*)

###### 通过将多个资产合并成单个资产并修复引用来删除重复资产的工具。

**替换引用工具（Replace References Tool）** 在编辑器中提供了一种将多个资产合并成单个资产的简单方法。例如，想象一下，某张纹理在开发过程中复制了多次，导致重复保存纹理，造成资产浪费。替换引用工具允许用户根据需要选择所有这类纹理，并让它们都指向同一个纹理实例。(引用)

尽管通过重新导入源资产能极大减少这种问题，但如果尝试添加相同文件两次（相同的名称和路径），当多人同时开发游戏时，仍会发生这类问题。

#### 7.7.1调用替换引用工具(和高级搜索语法配合)

​     若要访问该工具，只需在 **内容浏览器（Content Browser）** 中选择至少一项你希望在合并过程中使用的资产。然后，**单击右键** 并在出现的上下文菜单中的**资产操作**，单击"**替换引用（Replace References）**"。替换引用（Replace References）对话框随即出现，其中填充了在召唤该工具时选择的所有资产。你可以通过将其他资产从 **内容浏览器（Content Browser）** 拖至该对话框的主要部分来添加它们。

   合并通常仅限于选择**同一类型的对象**，但对纹理和材质则存在一些例外。如果没有看到替换引用（Replace References）选项或者不允许拖放操作，那么你应确保你只选择了**同一类型的资产**！如果你不小心添加了不希望添加的资产，你可以通过选择它并按下键盘上的 **删除（Delete）** 键来将其从对话框中删除。

#### 7.7.2合并资产

​       当你在对话框中填充了想要在合并过程中使用的所有资产后，选择其中一个资产作为"目标合并资产"，然后单击 **合并资产（Consolidate Assets）**。对列表中没有选择的资产的所有引用将被替换为对你所选择的资产的引用，**同时将在流程中删除所有未选择的资产**。

![Consolidate2.PNG](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/assets-content-packs/consolidating-assets/Consolidate2.jpg)

#### 7.7.3替换引用工具的工作方式

​       合并过程分为多个步骤。首先，对于要合并的任何有效对象，该工具尝试在已加载的且位于内存中的对象/UAsset中将这些对象的所有引用替换为"目标合并对象"的引用。这意味着，如果你已经打开了一个图或UAsset，而它引用了其中一个要合并的对象，该工具将尝试立即更新它。接下来，该工具将尝试直接删除要合并的对象（可能会失败，请参阅[**限制和警告**](https://docs.unrealengine.com/5.0/zh-CN/consolidating-assets-in-unreal-engine#限制和警告)）。最后，如果删除成功，该工具将 **对象重定向器** 保留在被删除对象的位置。它们会将包含对已删除对象之引用的卸载UAsset重定向到目标合并对象。

### 7.8类查看器

###### 用于查看UE4类的工具

​    用户可以使用 **类查看器（Class Viewer）** 来查看编辑器的类的层级结构。借助该工具，你可以创建蓝图并打开蓝图进行修改。你还可以打开关联的C++头文件，或选择某个类然后新建C++类。

#### 7.8.1打开类查看器

​    你可以点击 **窗口（Window） -> 开发者工具（Developer Tools） -> 类查看器（Class Viewer）** 打开"类查看器（Class Viewer）"。 

### 7.9属性矩阵

​    **属性矩阵（Property Matrix）** 允许对大量对象或Actor进行批量编辑 和值比对。它允许以表格形式列出一组对象的各种可配置属性， 并且表格可以按任意列排序。属性矩阵还提供标准的属性编辑器， 显示该表格视图中设置的当前选择的所有属性。

#### 7.9.1功能

| 功能               | 优势                                                         |
| ------------------ | ------------------------------------------------------------ |
| 批量对象编辑       | 一种更轻松的工作流程，用于为一组对象设置一系列可变值，同时又能够 将一组对象上的属性设置为同一个值。 同时处理数千个对象。 能够同时编辑多种对象类型。 |
| 批量细粒度对象比较 | 一次对数千个对象的值排序。 快速查找设置不正确的资产和Actor。 |
| 深度属性和数组支持 | 可以对数组和结构体类型的属性执行所有上述操作。 可以公开任意属性的列。 甚至可以处理数组索引。 |

#### 7.9.2访问属性矩阵

当前有两种方法可以访问 **属性矩阵**：

**细节（Details）** 面板中提供的 ![属性矩阵（Property Matrix）](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/assets-content-packs/property-matrix/button_property_matrix.jpg) 按钮， 位于 **搜索（Search)** 框旁边，可启动与当前选择关联的 **属性矩阵**。

[属性矩阵 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/property-matrix-in-unreal-engine/)

## 8.关卡

###### 关卡包含玩家可以看到并与之交互的所有内容，例如环境、可用对象、其他角色，等等。

虚幻引擎将每个关卡保存为单独的 `.umap` 文件，这就是为什么你有时会看到关卡被称为 **贴图（Maps）** 。

下面是组合起来创建关卡至少所需的元素列表：

- 一个 `.umap` 文件，即关卡本身。可将其视为保存其他所有内容的容器。
- 一个由 **静态网格体Actor（Static Mesh Actors）** 组成的环境。这些可以是树木、岩石、墙壁或其他环境Fixture。某些场景还使用其他类型的Actor，例如地形Actor或者水体Actor。
- 一个由 **骨骼网格体Actor（Skeletal Mesh Actor）** 表示的玩家角色。
- 一个或多个不同类型的 **光源（lights）** 。
- 环境声音和音效（例如，脚步声）。

### 8.1使用关卡

###### 如何在虚幻引擎中创建、保存和打开关卡

[在虚幻引擎中使用关卡 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/working-with-levels-in-unreal-engine/)

### 8.2管理多个关卡

关卡可以通过 **关卡（Levels）** 窗口来管理。你可以在 **窗口（Windows）** 菜单中打开它。

游戏始终有一个 **持久关卡（Persistent Level）**，并且能添加多个子关卡（Sublevel）；子关卡可以通过关卡流送体积（Level Streaming Volume）、蓝图（Blueprint）或C++代码加载。 **关卡（Levels）** 窗口会显示所有的关卡，允许你打开关卡（以粗体蓝色文本表示）、保存关卡，以及访问关卡蓝图（Level Blueprint）。只需在关卡编辑器视口中进行修改，就能修改当前关卡。你可以用此窗口批量处理地图（只要它们都是可写入的）。

![LevelsWindow.png](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/levels/managing-multiple-levels/LevelsWindow.jpg)

右键点击 **持久关卡（Persistent Level）** 后，会显示多个操作选项，包括将该关卡设置为当前关卡、更改可视性和锁定状态、选中关卡中的所有Actor。

![RightClickPersistent.png](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/levels/managing-multiple-levels/RightClickPersistent.jpg)

子关卡也有类似的选项，此外，还有一些用于移除子关卡和更改流送方法的额外选项。

![RightClickStreaming.png](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/levels/managing-multiple-levels/RightClickStreaming.jpg)

更改关卡的可视性只会影响它的显示，不会对关卡能否加载进游戏产生影响。不过，当你重新生成关卡时，此处不可见的关卡将不参与构建过程；如果你的关卡很复杂，这样能大大节省时间。

#### 8.2.1添加新的子关卡

持久关卡或子关卡的一部分可以拆分出来，作为新的子关卡。你也可以新建关卡或添加现有关卡来创建子关卡。 添加新的子关卡后，该关卡会自动成为"当前关卡"。因此，如果你想继续使用之前的关卡，请记得右键单击之前的关卡，在菜单中 **设为当前（Make Current）**。

##### 8.2.1.1添加已有关卡

1.单击 **关卡（Levels）** 下拉菜单，然后选择 **添加现有（Add Existing）**，添加一个新的子关卡。

![image-20220712165334218](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712165334218.png)

   2.在 **打开关卡（Open Level）** 对话框中选择要添加的关卡，然后单击 **打开（Open）**

![image-20220712165611337](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712165611337.png)

#### 8.2.2新建空白子关卡

  1.单击 **关卡（Levels）** 下拉菜单，然后选择 **新建（Create New）**，新建一个空白子关卡。

![image-20220712165810874](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712165810874.png)

#### 8.2.3拆分子关卡

如果你想把关卡的一部分拆分出来，以便单独加载它或与团队协作编辑），你可以选中要用的Actor，用它们新建一个关卡。

1. 在 **场景大纲视图（Scene Outliner）** 或 **视口（Viewport）** 中选中所有要移到新关卡的Actor。
2. 在 **关卡（Levels）** 窗口中，单击 **关卡（Levels）** 下拉菜单，然后选择 **使用选定Actor新建（Create New with Selected Actors）** 以创建一个新子关卡。
3. ![image-20220712170020413](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712170020413.png)

​         所有选中的Actor都会在原有关卡中被移除，并被添加到新关卡中。新关卡将作为当前持久关卡的一个子关卡，并被设置为当前关卡，供你在视口中处理。

如果你移动的Actor被某个保留在持久关卡中的Actor所引用，引擎会弹出消息，询问你是否真的要将其从持久关卡中删除。

#### 8.2.4在关卡间迁移Actor

你可以先在当前关卡中复制Actor，然后切换到目标关卡并粘贴Actor时。不过，有一种更简单的方法。

1. 在 **场景大纲视图（Scene Outliner）** 中或 **视口（Viewport）** 中选中要移至新关卡的Actor。
2. 在 **关卡（Levels）** 窗口中，**右键单击** 关卡，然后在右键菜单中选择 **移动所选Actor至关卡（Move Selected Actors to Level）**。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712170958577.png" alt="image-20220712170958577" style="zoom:50%;" />

3.按下 **Ctrl+S**，保存所有关卡。

#### 8.2.5关卡细节

​      在 **关卡（Levels）** 窗口中，点击图中的的放大镜标识可以打开 **关卡细节（Level Details）** 窗口，它允许你访问当前关卡的更多信息。若要设置关卡流送体积（Level Streaming Volumes），你需要打开关卡的 **关卡细节（Level Details）**；详细操作过程，请参阅[关卡流送指南](https://docs.unrealengine.com/5.0/zh-CN/level-streaming-using-volumes-in-unreal-engine)。

#### 8.2.6子关卡可视化选项

你可以在主 **关卡（Levels）** 窗口中或 **关卡细节（Level Details）** 窗口中设置子关卡的颜色。

若要切换关卡颜色，请点击视口上的 **显示（Show）** 按钮，然后选择 **高级（Advanced）> 关卡颜色（Level Coloration）**。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712171637490.png" alt="image-20220712171637490" style="zoom:50%;" />

持久关卡将用白色显示，而所有子关卡将用它们的选定颜色表示。**关卡颜色（Level Coloration）** 只能在透视和正交视口下工作；在 **游戏模式（Game Mode）**下会被关闭。

![image-20220712171739425](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712171739425.png)



### 8.3世界设置(世界场景设置)

###### 在世界设置面板中可以设置和重载关卡专用设置。

每个关卡都有从 **世界设置（World Settings）** 面板应用到关卡中的独有设置。使用此面板可以执行各种操作：从确保运行关卡时激活正确的 **游戏模式（Game Mode）** ，到调整全局光照在该关卡中的运行方式。

要打开世界设置面板，请在主菜单中转到 **窗口（Window）** ，然后选择 **世界设置（World Settings）** 。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712181405647.png" alt="image-20220712181405647" style="zoom:50%;" />

##### 8.3.1预先计算的可视性

​     预先计算的可视性体积可以减少渲染操作线程时间，但需要占用运行时内存。在***处理较小的关卡*或针对由于硬件（例如移动设备）限制而无法执行动态遮蔽剔除目标的平台**时，此设置有助于优化游戏的性能。对于较大的、更复杂的环境，无法很好地伸缩。

如需了解更多信息，请参阅[预计算可视性体积](https://docs.unrealengine.com/5.0/zh-CN/precomputed-visibility-volumes-in-unreal-engine)。

##### 8.3.2游戏模式

在这里可以为当前关卡选择和配置游戏模式。**游戏模式定义游戏规则**，例如玩家数量、得分或获胜条件。可以从你使用的项目模板随附的现有游戏模式中进行选择，或创建自定义游戏模式。

从 **GameMode重载（GameMode Override）** 下拉菜单中选择游戏模式之后，就可以配置特定于该模式的设置。

如需了解更多信息，请参阅[设置游戏模式](https://docs.unrealengine.com/5.0/zh-CN/setting-up-a-game-mode-in-unreal-engine)。

##### 8.3.3Lightmass

在此部分中，可以指定Lightmass设置，例如间接光照的细节和质量，是否使用环境光遮蔽（即模拟间接光照形成的软阴影，可以增加场景的深度）。

如需详细了解Lightmass以及可以配置的不同设置，请参阅[全局光照](https://docs.unrealengine.com/5.0/zh-CN/global-illumination-in-unreal-engine)。

##### 8.3.4世界(场景)

这些设置会影响游戏世界的核心方面，例如关卡边界、导航系统、Actor在损坏之前可以掉落的深度。

如需了解此部分中不同领域的信息，请参阅：

- [World Composition](https://docs.unrealengine.com/5.0/zh-CN/world-composition-in-unreal-engine)
- [关卡流送体积参考](https://docs.unrealengine.com/5.0/zh-CN/level-streaming-volumes-reference-in-unreal-engine)

##### 8.3.5物理

使用此部分可以重载世界重力，而世界重力会影响某些Z轴动作，例如角色可以跳起的高度或物体的掉落速度。

此外，还可以在这里指定更多的高级设置，例如默认的物理体积类和物理碰撞处理程序类。

如需了解虚幻引擎5中物理的更多信息，请参阅[物理](https://docs.unrealengine.com/5.0/zh-CN/physics-in-unreal-engine)。

##### 8.3.6Broadphase(粗测阶段)

此部分介绍Broadphase碰撞的设置，这是NVIDIA的PhysX系统的功能。你可以指定是使用Broadphase客户端还是服务器端。

虚幻引擎实施了多盒体修剪，此功能将Broadphase分割成盒体网格，而你可以控制这些盒体的设置。**MBPBounds** 和 **MBPOuter边界（MBPOuter Bounds）** 部分控制多盒体的边界。

**MBPBounds** 中的空间按照 **MBPNumSubDivs** 值进行划分，以创建网格。例如：

- 如果 MBPNumSubDivs = 2，将创建4单元格(2 x 2)网格。
- 如果 MBPNumSubDivs = 3，将创建9单元格(3 x 3)网格。

如果在物理上活跃的object掉落到 **MBPOuterBounds** 指定的边界之外，将不再为其考虑碰撞情况。启用 **使用MBPOuter边界（Use MBPOuter Bounds）** 选项将在多盒体网格边缘上创建四个专用的单元格。

如需详细了解该系统，请参阅NVIDIA关于[刚体碰撞](https://docs.nvidia.com/gameworks/content/gameworkslibrary/physx/guide/Manual/RigidBodyCollision.html)的文档。//看不懂(暂时)

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712183642079.png" alt="image-20220712183642079" style="zoom: 67%;" />

##### 8.3.7HLOD系统(UE4的叫LOD系统)

在此部分中，可以启用分层细节级别（HLOD）。

HLOD可以使用在长视野距离上具有单个组合静态网格体Actor，以取代多个静态网格体Actor。这有助于减少需要为场景渲染的Actor数量，通过减少每个帧的绘制调用数量来提高性能。//在远处看时仅渲染那些单个组合的静态网格体

如需详细了解如何使用HLOD，请参阅[HLOD](https://docs.unrealengine.com/5.0/zh-CN/hierarchical-level-of-detail-in-unreal-engine)。

##### 8.3.8世界分区设置和世界分区(新技术*)

**世界分区（World Partition）** 是新的数据管理和基于距离的关卡流送系统，可以为大型世界管理提供完整的解决方案。以前需要通过将单个持久关卡中的世界存储到网格单元格中，以便将大型关卡划分成子关卡，但现在的系统已经不需要如此操作，并且能够自动流送系统，从而根据与流送源的距离来加载和卸载这些单元格。

如需详细了解世界分区以及如何配置其设置，请参考虚幻引擎5抢先体验版文档中的[世界分区](https://docs.unrealengine.com/5.0/en-US/building-virtual-worlds/world-partition/)部分。

##### 8.3.9导航

配置关卡中使用的导航网格。

##### 8.3.10VR

使用 **米级世界缩放（World to Meters）** 变量来调整虚拟世界的大小规模。提高或降低此数字将会使用户感觉自己相对于周围的世界而言变得更大或更小了。此设置用虚幻单位（UU）来表示。**在UE4中，1虚幻单位（UU）等于1厘米（cm）。**

假设你的内容使用1虚幻单位=1厘米进行构建，将 **米级世界缩放（World to Meters）** 设置为 **10** 将会使世界看起来非常大，而将米级世界缩放设置为 **1000** 将使世界变得非常小。//没什么区别

如需详细了解如何缩放VR体验，请参阅[VR世界规模](https://docs.unrealengine.com/5.0/zh-CN/xr-best-practices-in-unreal-engine)。

如需了解在UE5中开发XR的一般性说明，请参阅[XR开发](https://docs.unrealengine.com/5.0/zh-CN/developing-for-xr-experiences-in-unreal-engine)。

##### 8.3.11渲染

在此部分中，可以配置与距离场环境光遮蔽相关的多项配置，以及动态间接阴影。

如需了解更多信息，请参阅[距离场环境光遮蔽](https://docs.unrealengine.com/5.0/zh-CN/distance-field-ambient-occlusion-in-unreal-engine)。

##### 8.3.12音频

使用此部分中的设置可以配置项目宏的默认音效行为，例如音量、混响和消退时间。

如需详细了解虚幻引擎5中的音频和音效，请参阅[处理音频](https://docs.unrealengine.com/5.0/zh-CN/working-with-audio-in-unreal-engine)。

##### 8.3.13更新决策

**更新（Ticking）** 指的是以固定的间隔在Actor或组件上运行一段代码或蓝图脚本，通常是每帧一次。对于每个Actor或组件，更新通常单独启用。

除非游戏正在运行专门要求在一次性初始化 `BeginPlay()` 函数之前运行每帧 `Tick()` 更新函数的旧代码，否则应该**禁用**此选项以确保Object正确更新。

如需详细了解更新和Actor行为，请参阅[Actor Ticking](https://docs.unrealengine.com/5.0/zh-CN/actor-ticking)。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712190328450.png" alt="image-20220712190328450" style="zoom: 67%;" />

##### 8.3.14AI

在此部分中，可以启用虚幻引擎4的人工智能（AI）系统。

如需详细了解此系统，请参阅[人工智能](https://docs.unrealengine.com/5.0/zh-CN/artificial-intelligence-in-unreal-engine)。

##### 8.3.15烘焙

**烘焙（Cooking）** 是构建游戏并将其部署到平台（例如PC或移动设备）的过程的一部分。这些设置影响着如何将场景中的内容包含在已构建的游戏中。

如需详细了解此过程，请参阅[游戏的打包（Packaging）和烘焙（Cooking）](https://docs.unrealengine.com/5.0/zh-CN/packaging-and-cooking-games-in-unreal-engine)。

### 8.4更改默认关卡

![image-20220712191709601](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712191709601.png)

## 9.Actor和几何体

所有可以放入关卡的对象都是 **Actor**

Actor支持三维变换，例如平移、旋转和缩放。

在C++中，AActor是所有Actor的基类。

### 9.1选择Actor

组合键（对单个Actor的）

| **鼠标和键盘快捷键**                    | **操作**                              |
| --------------------------------------- | ------------------------------------- |
| **左键单击并拖动**                      | 将当前选择替换为方框内包含的Actor。   |
| **左键单击并拖动** + **Shift**          | 将方框内包含的Actor添加到当前选择中。 |
| **左键单击并拖动** + **Ctrl** + **Alt** | 从当前选择中移除方框内的Actor。       |

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712195214298.png" alt="image-20220712195214298" style="zoom:50%;" />（UE5独有，左上角）



### 9.2变换Actor

在 **虚幻引擎** 中 **变换（Transforming）** Actor是指对其进行移动、旋转或缩放（也就是，调整Actor的位置、方向和/或大小）。本页描述了如何执行这些操作，以及一些常用的Actor操作快捷键。

在虚幻编辑器中，有两种变换Actor的方法：

- 手动变换
- 交互式变换

##### 9.2.1手动变换（懂得都懂）

变换（Transform）属性默认 *相对（relative）* 坐标空间，这意味着变换相对于Actor的父节点进行。通过单击属性标签旁边的下拉箭头，可在 *相对（relative️）* 和 *世界（world️）* 变换之间切换。*世界（world️）* 变换相对于世界坐标进行，而不是相对于Actor的父节点。有关更多信息，请参阅此页面上的 **世界和局部变换模式** 小节。

#### 9.2.2交互式变换

你可以使用一种名为 **小工具（gizmo️）** 的可视化工具，直接在 **关卡视口（Level Viewport）** 内进行 **交互式变换（Interactive transformation）**。小工具（gizmo）有时也称为 **控件（widget）**；在虚幻引擎中，这些术语的含义相同。//就E，R控制的那些东西



![image-20220712200512325](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712200512325.png)

选择一个或多个Actor后，你可以通过在键盘按下 **空格键** 在不同类型的小工具之间切换。

#### 9.2.3世界和局部变换模式

使用交互式变换方法时，你可以选择执行变换时所使用的参考坐标系。这意味着你可以按以下方式变换Actor：

- 世界空间，也就是沿世界轴；
- Actor的局部空间，也就是沿其局部轴。
- **世界空间：平移小工具的XYZ轴与世界的XYZ轴相同。沿Z轴拖动可相对于地面上下移动立方体。**
- **平移小工具的XYZ轴使用立方体的局部坐标。沿Z轴拖动也会上下移动立方体，但会倾斜一个角度。**
- 默认情况下，虚幻编辑器的启动模式为世界变换模式。要切换到局部变换模式，请单击 **关卡视口（Level Viewport）** 工具栏中的 **地球** 图标。地球变成一个立方体图标，表示你现在处于局部变换模式。单击立方体可切换回世界坐标。

![image-20220712201224929](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712201224929.png)

#### 9.2.4调整Actor的枢轴点(坐标轴)

变换Actor时，你通常会从Actor的基础枢轴点执行变换。如果你启用了变换小工具，则会在该小工具的三个轴相交处看到 **枢轴点**。

你可以临时调整Actor枢轴点的位置，方法是鼠标中键单击 **变换** 小工具的中心处的球体并拖动，即可移动枢轴点。然后，你可以围绕新的枢轴点变换对象。

枢轴点可位于Actor内部或外部。

取消选择该Actor后，枢轴点会立即跳回原位。若要**永久更改枢轴点**，在调整枢轴点后，右键单击该静态网格体，并选择 **枢轴点（Pivot）> 设置为枢轴点偏移（Set as Pivot Offset）**。

若要将枢轴点重置为其默认位置，右键单击该静态网格体，然后选择 **枢轴点（Pivot）> 重置枢轴点偏移（Reset Pivot Offset）**。

![image-20220712201954900](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712201954900.png)

| **功能按钮**                               | **工具或操作**                                               |
| ------------------------------------------ | ------------------------------------------------------------ |
| W                                          | 选择移动工具。                                               |
| E                                          | 选择旋转工具。                                               |
| R                                          | 选择缩放工具。                                               |
| V（使用平移小工具时按住）                  | 切换顶点对齐。                                               |
| **左键单击并拖动**（在变换小工具上）       | 移动、旋转或缩放选定的Actor，具体取决于当前活动的变换小工具。 |
| **中键单击并拖动**（在枢轴点上）           | 移动选定Actor的枢轴点。                                      |
| **Ctrl + W**（在Actor上）                  | 将选定的Actor复制到原始Actor所在的相同坐标处。               |
| **Alt + 左键单击并拖动**（在平移小工具上） | 复制选定的Actor。                                            |
| **H**（在Actor上）                         | 隐藏选定的Actor。                                            |
| **Ctrl + H**                               | 显示所有隐藏的Actor。                                        |
| **Shift + E**（在Actor上）                 | 选择关卡中与所选Actor类型相同的所有匹配Actor。               |
| **Ctrl + 左键单击**（在Actor上）           | 将该Actor添加到当前Actor选择中。                             |

#### 9.2.5键盘快捷键

### 9.3Actor移动性(文档全英文，机翻)

###### 控制 Actor 在游戏过程中是否可以以某种方式移动或更改的设置。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712203126765.png" alt="image-20220712203126765" style="zoom:67%;" />

#### 9.3.1静态

**静态**移动性是为在游戏过程中不会以任何方式移动或更新的Actor保留的。

- **静态网格体**移动性设置为“静态”的Actor的阴影会参与预先计算的光照贴图，这些光照贴图使用[Lightmass](https://docs.unrealengine.com/5.0/zh-CN/lightmass-basics-in-unreal-engine)来生成和处理它们。这种移动性使它们成为结构或装饰网格的理想选择，这些网格在游戏过程中**永远不需要重新定位**。但请注意，*他们的材质仍然可以进行动画处理*。

- **光**移动性设置为“静态”的Actor会参与使用[Lightmass](https://docs.unrealengine.com/5.0/zh-CN/lightmass-basics-in-unreal-engine)的预先计算的光照贴图。它们照亮了静态和静止演员的关卡。对于可移动的Actor，他们使用间接照明方法，如[体积光照贴图](https://docs.unrealengine.com/5.0/zh-CN/volumetric-lightmaps-in-unreal-engine)。

  有关详细信息，请参阅 [静态光源的移动性](https://docs.unrealengine.com/5.0/zh-CN/static-light-mobility-in-unreal-engine)页面。



#### 9.3.2固定

**固定**移动性是为Actor保留的，这些Actor可以在游戏过程中改变，但不能移动。

- **静态网格体**移动性设置为“静止”的演员可以更改，但不能移动。它们不会对使用[光质量](https://docs.unrealengine.com/5.0/zh-CN/lightmass-basics-in-unreal-engine)预先计算的光照贴图做出贡献，并且在被静态或静止光照亮时像可移动的Actor一样被照亮。但是，当被可移动光源照亮时，当光线不移动时，他们将使用[缓存阴影贴图](https://docs.unrealengine.com/5.0/zh-CN/movable-light-mobility-in-unreal-engine)在下一帧中重复使用，这可以提高使用动态光照的项目的性能。

- **光**移动性设置为静止的演员在游戏过程中可能会以某种方式发生变化，例如改变其颜色或强度（更亮或更暗），甚至完全关闭。静止光源仍有助于使用 [Lightmass](https://docs.unrealengine.com/5.0/zh-CN/lightmass-basics-in-unreal-engine) 预先计算出的光照贴图，但也可以为移动对象投射动态阴影。

  影响同一 Actor 的静止灯光过多会对性能产生重大影响。有关更多信息，请参阅 []）（建筑虚拟世界/光照和阴影/光照类型和移动性/固定照明）页面。

#### 9.3.3可移动

**可移动**移动性是为在游戏过程中需要添加、删除或移动的Actor保留的。

- **静态网格体**移动性设置为“可移动”的Actor会投射一个完全动态的阴影，该阴影不会将预先计算的阴影投射到光照贴图中。当被具有静态移动性的光源照亮时，它们将使用间接照明方法（如[间接照明样本）](https://docs.unrealengine.com/5.0/zh-CN/indirect-lighting-cache-in-unreal-engine)来照亮它们。对于具有静止或可移动移动性的灯光，它们只会**投射动态阴影**。这是需要在场景中添加、移除或移动的任何非变形网格元素的典型设置。
- **光**移动性设置为“可移动”的演员只能投射动态阴影。除了能够在游戏过程中移动光线外，它们还可以在游戏过程中更改其颜色和强度。但是，在使这些灯光投射阴影时必须小心，因为它们的阴影方法是**性能最密集的**。应该注意的是，由于虚幻引擎的延迟渲染系统，非阴影可移动光源的计算成本很低。

### 9.4Actor分组

###### 如何在虚幻引擎中创建和处理Actor组

Actor分成一组，你就可以同时管理多个Actor。利用Actor组，你可以执行以下操作：

- 将分成一组的Actor作为一个整体变换，比如旋转、平移、缩放。多个Actor属于一个组时，单个Actor变换将锁定。
- 临时解锁一个组，变换单个Actor，然后重新锁定该组以冻结组中的各个Actor，防止对单个Actor的变换进行整个组的更改。<font color='brown'>//仍然是一个组，只是可以对单个Actor进行变换</font>
- 在组中添加和删除Actor。

**一个Actor一次只能属于一个组。**

要将多个Actor分成一组，你需要首先确保启用了 **允许组选择（Allow Group Selection）**。此选项位于 **主工具栏（Main Toolbar）** 上的 **设置（Settings）** 菜单中。你还可以使用 **Ctrl + Shift + G** 键盘快捷方式切换此选项。（UE4和UE5）

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712204842750.png" alt="image-20220712204842750" style="zoom:50%;" /><img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712205117132.png" alt="image-20220712205117132" style="zoom:50%;" />

要将两个或多个Actor分成一组，请选择相应Actor，然后执行以下任一操作：

- 右键点击任一所选Actor，弹出 **上下文菜单（context menu）**，然后选择 **组（Group）**。
- 使用 **Ctrl + G** 键盘快捷方式。

Actor组通过 **视口（Viewport）** 中的绿色括号定界。

![一个Actor组包含四个静态网格体Actor。](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/actors/grouping-actors/LockedGroup.jpg)

*一个Actor组包含四个静态网格体Actor。只要进行变换，就会同时影响此组中的所有Actor。*

###### Note: 处于不同关卡中的Actor不能分成一组。将当前位于组中的Actor从一个关卡移到另一个关卡，就会将其从现有组中删除。但是，你可以在不同关卡之间移动整个组。

将多个Actor分成一组时，你将在 **世界大纲视图（World Outliner）** 中创建 **组Actor（Group Actor）**。要选择该组中的所有Actor，你可以选择组Actor或组中的成员。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220712205950236.png" alt="image-20220712205950236" style="zoom:50%;" />

### 9.5合并Actor

###### 如何在虚幻引擎中将两个或更多静态网格体Actor合并为单个Actor。

在 **虚幻引擎** 中，可通过 **Actor合并** 操作将两个或更多 **静态网格体Actor** 结合为新的单个Actor。这样可以减少绘制调用，有助于项目优化。

#### 9.5.1Actor合并工作流

要将两个或更多静态网格体Actor并入关卡，请按照以下步骤操作：

1. 在 **关卡视口（Level Viewport）** 或 **世界大纲视图（World Outliner）** 中，选择要合并的静态网格体Actor。

2. 在顶部菜单栏中，选择 **Actor >合并Actor（Merge Actors）。**

   ![image-20220713083249175](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220713083249175.png)

3. 选择要执行的合并类型：**合并（Merge）**、**简化（Simplify）**、**批量（Batch）**、或 **近似（Approximate）**。有关更多信息，请参阅"合并类型"小节。

4. <img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220713140833280.png" alt="image-20220713140833280" style="zoom:50%;" />

   所选选项将使用默认或最后配置的设置自动执行合并。要配置合并选项，请在顶部菜单栏中，选择 **Actor > 合并Actor（Merge Actors） > 合并Actor设置（Merge Actor Settings）。**

5. 命名新的Actor，选择创建它的文件夹，随后点击 **保存（Save）**。

#### 9.5.2合并类型

1. **合并（Merge）**

在默认情况下，该方式会合并全部选中的静态网格体Actor，为每个材质创建一个网格体分段。可以指定要使用的 **LOD** 级别。

绘制调用数量等于 **材质** 数量。这种方式保留UV。

您也可以选择合并全部材质，这将为整个网格体烘焙单一材质。其结果是单一分段和单一绘制调用，但 **不** 保留UV。将针对每一分段进行遮挡剔除。



**2.简化（Simplify）**

该合并方式将全部选中的静态网格体Actor合并成一个单一网格体，它被称为 **代理网格体（Proxy Mesh）**。它使用选定每个网格体的**最少细节LOD**，并根据设置简化网格体形状。顶点数量也会减少。

其结果是单一绘制调用。它为整个网格体烘焙单一材质，且 **不** 保留UV。将针对整个网格体进行遮挡剔除。

有关更多信息，请参阅[代理几何体概述](https://docs.unrealengine.com/5.0/zh-CN/proxy-geometry-tool-overview-in-unreal-engine)页面。



**3.批量（Batch）**

该方式将基于**相同的静态网格体组件**创建实例化静态网格体组件。它**保留UV**，且不影响遮挡剔除。



**4.近似（Approximate）**（简化plus)

这是 **虚幻引擎5** 的新功能，与"简化（Simplify）"合并方式相似。两者区别在于"近似（Approximate）"可以处理更复杂的源网格体（例如Nanite网格体），而"简化（Simplify）"在简化或纹理烘焙步骤中都会失败。

#### 9.5.3合并设置

在 **合并Actor设置（Merge Actor Settings）** 面板中，可对单个合并设置进行配置。要打开该面板，请在顶部菜单栏中，选择 **Actor >合并Actor（Merge Actors）>合并Actor设置（Merge Actor Settings）**。

![image-20220713142545419](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220713142545419.png)

[在虚幻引擎中合并Actor | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/merging-actors-in-unreal-engine/)(详情)

#### 9.5.4替换源Actor

如果启用 **替换源Actor（Replace Source Actors）** 选项，在视口中选中的Actor会被移除，并替换为新合并的Actor。这不会从内容浏览器或项目文件夹中删除源Actor资产。

该选项对于本页描述的所有合并方法通用。****

### 9.6Actor参考

###### 介绍虚幻引擎中最常用Actor的作用，以及你可在哪里进一步了解相关信息。

#### 9.6.1网格体Actor

**网格体** 定义了环境道具或玩家角色的形状和大小。虚幻引擎使用两种类型的网格体Actor：

- 静态网格体Actor，用于构建关卡和环境。
- 骨骼网格体Actor，通常用于玩家角色和动画非玩家角色（NPC）。

#### 9.6.2静态网格体Actor

**静态网格体Actor（Static Mesh Actor）** 是一种简单的Actor类型，用于在关卡中显示网格体。尽管名称暗示Actor是静态的（或无法移动），但这里"静态"指的是**所使用的网格体类型**，而不是指Actor能否移动。*如果网格体的几何体不会改变，该网格体就是 静态的。*否则，Actor本身可以在运行期间以其他方式移动或更改。

**静态网格体Actor常用于创建游戏世界或其他类型的<font color='green'>环境。</font>

请参阅[静态网格体](https://docs.unrealengine.com/5.0/zh-CN/static-meshes)小节了解更多。

#### 9.6.3骨骼网格体Actor

**骨骼网格体Actor（Skeletal Mesh Actor）** <font color='red'>显示动画网格体</font>，其几何体可以**变形**，通常是通过使用动画序列期间的控制点来变形。这些Actor可以从外部3D动画应用程序创建和导出，也可以直接在虚幻引擎中编程来实现。

骨骼网格体Actor常用于表示玩家角色或NPC，以及其他动画生物和复杂的机制。

请参阅[骨骼网格体](https://docs.unrealengine.com/5.0/zh-CN/skeletal-mesh-actors-in-unreal-engine)小节了解更多。

#### 9.6.4笔刷Actor（几何体Actor）

**笔刷Actor（Brush Actors）** 是一种基本类型的Actor，用于显示场景中的简单3D几何体，例如球体、立方体和楼梯。这些Actor可以使用关卡编辑器中的几何体编辑模式来更改。//UE提供的一些3D几何体

虚幻引擎中提供了以下类型的笔刷Actor：

- 盒体
- 椎体
- 圆柱体
- 弧形楼梯
- 线性楼梯
- 螺旋楼梯
- 球体

笔刷Actor常用于快速对环境制作原型并**粗略规划关卡**。这些Actor可用于获得对环境道具的大小和布置的**大致映像**。

   <font color='red'>不建议将几何体笔刷用作关卡设计的单一方法。它应视情况使用，在关卡早期搭建阶段较为有用。</font>

理论上说，建议将几何体笔刷用于在关卡中填充、雕刻空间体积。现在，静态网格体取代了几何体笔刷作为关卡设计的主要模块，其效率远高于几何体笔刷。但在产品的早期阶段，几何体笔刷依旧有用——它可以快速设置关卡和对象的原型，也可用于无法使用3D建模工具的关卡构建。

通常，可把几何体笔刷看作关卡设计阶段中用于创建基本形状的方法。它可以作为始终存在的用具，也可作为美术师测试用的临时工具。

##### 9.6.4.1几何体笔刷的用途

###### 9.6.4.1.1规划关卡

创建关卡的标准工作流程大致为：

- 规划关卡和设计关卡路径
- 游戏玩法测试流程
- 修改布局并重复测试
- 初始建模阶段
- 初始光照阶段
- 碰撞和性能问题的测试
- 完善阶段
- ![ElementalBSP.png](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/actors/actors-reference/brush-actors/ElementalBSP.jpg)



![ElementalComplete.png](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/actors/actors-reference/brush-actors/ElementalComplete.jpg)

第一步是在使用静态网格体和其他成品美术资源填充关卡前，先规划关卡以确定布局和流程。**通常使用几何体笔刷创建关卡的轮廓**，然后通过测试和迭代，完成最终布局。由于无需美术团队，因此在关卡设计的这方面最适用几何体笔刷。关卡设计师可直接使用虚幻编辑器中的内置工具创建和修改几何体笔刷，以达到满意的关卡布局和效果。

测试结束后便开始建模，几何体笔刷将逐渐被静态网格体取代。此过程可更快初始迭代，同时也为美术团队的建模工作提供了良好参考。

###### 9.6.4.1.2简单填充几何体

关卡设计师创建关卡时，时常需要使用较为简单的几何体填充间隙或空间。若无用于填充控件的现成静态网格体，设计师可直接使用几何体笔刷进行填充，无需美术团队创建自定义网格体。尽管静态网格体性能更好，但对于简单几何体而言，偶尔使用几何体笔刷也不会造成严重影响。

#### 9.6.5创建笔刷

使用 **模式（Mode）** 面板中的 **几何体（Geometry）** 选项卡创建笔刷：

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220713172203418.png" alt="image-20220713172203418" style="zoom:50%;" />

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220713172244785.png" alt="image-20220713172244785" style="zoom:50%;" />







请参阅[几何体笔刷Actor](https://docs.unrealengine.com/5.0/zh-CN/geometry-brush-actors-in-unreal-engine)页面了解更多。

#### 9.6.5光源Actor

顾名思义，**光源Actor（Light Actors）** 用于将不同类型的光源放置在你的关卡中。一些光源Actor在周围有一个**有限的效果范围**，而其他光源Actor则影响你的整个关卡。

虚幻引擎中提供了以下类型的光源Actor：

- 定向光源
- 点光源
- 聚光源
- 矩形光源
- 天空光照

你可以从以下虚幻在线学习课程中进一步了解有关虚幻引擎的光照基础知识：

- [光照基本概念和效果](https://www.unrealengine.com/en-US/onlinelearning-courses/lighting-essential-concepts-and-effects)
- [介绍全局光照](https://www.unrealengine.com/en-US/onlinelearning-courses/introducing-global-illumination)

##### 9.6.5.1定向光源Actor

**定向光源（Directional Light）** 将模拟从**无限远的光源发射的光线**。这意味着此光源投射的所有阴影都将是平行的，从而使其非常适合模拟太阳光。

请参阅[定向光源](https://docs.unrealengine.com/5.0/zh-CN/directional-lights-in-unreal-engine)页面了解更多。

##### 9.6.5.2点光源Actor

**点光源（Point Lights）** 类似于现实世界的灯泡。点光源从**中心（即关卡中的单个点）沿所有方向发射光线**。

请参阅[点光源](https://docs.unrealengine.com/5.0/zh-CN/point-lights-in-unreal-engine)页面了解更多。

##### 9.6.5.3聚光源Actor

**聚光源（Spot Lights）** 类似于舞台灯光或手电筒。聚光源**从单个点以圆锥形的形状向外发射光线**。聚光源的形状由两个单独的锥角定义：

- 内锥角，光源在其中实现完全亮度。
- 外锥角，它定义了聚光源的外边界。

从内锥角延伸到外锥角的限值，光源的强度会衰减，在聚光源的光照盘面周围柔化。

请参阅[聚光源](https://docs.unrealengine.com/5.0/zh-CN/spot-lights-in-unreal-engine)页面了解更多。

##### 9.6.5.4矩形光源Actor

**矩形光源（Rect Lights）** 从**定义了宽度和高度的矩形平面**将光线发射到关卡中。你可以使用Actor这些来模拟具有矩形表面的各种光源，如窗户、电视或显示器屏幕。

请参阅[矩形光源](https://docs.unrealengine.com/5.0/zh-CN/rectangular-area-lights-in-unreal-engine)页面了解更多。

##### 9.6.5.5天空光照Actor//远处的光

**天空光照（Sky Light）** 可捕获**关卡的远处**，并将其作为光源**应用于场景**。这意味着，无论你的天空是来自大气、天空盒顶部的云层或者远山，天空的外观及其光照/反射都将匹配。

如果你在开发增强现实（AR）应用程序，请考虑改用 **AR天空光照（ARSky Light）**Actor。

AR天空光照是天空光照类的子类，使用现实世界环境探头来更新反射。每当对应探头的纹理基于现实世界光照更新时，它会重新生成光照和反射。这一切都在渲染线程上发生。

请参阅[天空光照](https://docs.unrealengine.com/5.0/zh-CN/sky-lights-in-unreal-engine)页面了解更多。

#### 9.6.1摄像机Actor

与现实世界的对等物一样，**摄像机Actor（Camera Actors）** 用于**查看你的关卡并创建过场动画序列**。此外，有一些起到支持作用的Actor可用于模拟现实世界摄像机镜头。

请参阅[摄像机Actor](https://docs.unrealengine.com/5.0/zh-CN/camera-actors-in-unreal-engine)了解详情。

#### 9.6.2音频和声音Actor（*）

**音频和声音Actor（Audio and Sound Actors）** 用于向你的关卡添加音乐、语音录制和声音效果。

#### 9.6.3环境声音Actor

使用 **环境声音Actor（Ambient Sound Actor）** 可在关卡中的特定位置播放循环（连续）声音。

请参阅[环境声音Actor用户指南](https://docs.unrealengine.com/5.0/zh-CN/ambient-sound-actor-user-guide-in-unreal-engine)页面了解更多。

#### 9.6.4音频音量

你可以使用 **音频音量Actor（Audio Volume Actor）** 在蓝图图表中定义可用于处理声音的区域，并将其设置用于应用混响效果、设置音量、定义受影响的区域、模拟声音上的遮蔽并定义声音音量的形状。

请参阅[音频音量](https://docs.unrealengine.com/5.0/zh-CN/audio-volumes-in-unreal-engine)页面了解更多。

#### 9.6.5Gameplay Actor

**Gameplay Actor** 将触发交互式功能。别受其名称误导，它们其实在所有种类的交互式应用程序中有广泛的用途，不仅仅是游戏。

下面是最常用的Gameplay Actor类型。

<font color='brown'>Tip</font>

**凡是能放进关卡中的东西，都称为Actor。体积（Volume）是有三维形状（例如立方体或球体）的Actor类型。**

#### 9.6.6玩家出生点

**玩家出生点（Player Start）** 是放置在关卡中的一种Actor，用于指定在玩家开始关卡时，玩家角色在何处生成。

请参阅[玩家出生点](https://docs.unrealengine.com/5.0/zh-CN/player-start-actor-in-unreal-engine)页面了解更多。

#### 9.6.7触发器体积

**触发器（Triggers）** 这种Actor会在关卡中的其他事物（如玩家角色或另一个对象）与之交互时导致某个事件发生。例如，玩家可以与触发器交互以开启光源。

虚幻引擎中提供了以下类型的触发器：

- 盒体触发器

- 胶囊体触发器

- 球体触发器

  <img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220713152854881.png" alt="image-20220713152854881" style="zoom:50%;" />

所有这三种触发器类型有相同的功能。唯一不同的是形状。你可以根据触发器在关卡中的视觉表示种类来选择不同的形状。

此外，虚幻引擎还有泛型触发器体积。就像几何体Actor那样，你可以使用此体积快速对关卡中的交互**制作原型**。

请参阅[触发器Actor](https://docs.unrealengine.com/5.0/zh-CN/trigger-volume-actors-in-unreal-engine)页面了解更多。

#### 9.6.8阻挡体积(空气墙)

顾名思义，**阻挡体积（Blocking Volumes）** 用于防止玩家穿过。例如，你可以使用阻挡体积不让玩家从游戏世界的边缘掉落。

#### 9.6.9Kill Z体积

**Kill Z体积Actor（Kill ZVolume Actor）** 会在玩家角色进入体积或与之交互时立即将其"杀死"（销毁）。你可以在Kill Z体积Actor的细节面板中**指定销毁条件**。

请注意，Kill Z体积Actor不同于 **世界设置（World Settings）** 面板中的"Kill Z"选项，后者会在玩家越过Z轴（垂直轴）上的特定点之后立即摧毁玩家。

#### 9.6.10施加伤害体积

**施加伤害体积（Pain Causing Volume）** 会对进入体积的玩家或对象**逐渐**造成伤害。例如，如果玩家站在火焰中，你可以设置施加伤害体积来对其生命值产生对应的伤害。

不要站在火焰中。

请参阅[伤害施加体积](https://docs.unrealengine.com/5.0/zh-CN/pain-causing-volume-actor-in-unreal-engine)页面了解更多。

#### 9.6.11角色和Pawn Actor

**Pawn** 和 **角色（Characters）** 这两种Actor都可以用于表示玩家和AI控制的角色。

##### 9.6.11.1Pawn

**Pawn** 是世界中的玩家或AI实体的物理呈现。Pawn不仅决定了玩家或AI实体的视觉外观，还决定了它在碰撞和其他物理交互方面如何与世界交互。

根据你所构建的项目种类，Pawn可以是玩家头像、汽车，也可以根本没有物理呈现。

虚幻引擎中提供了两种类型的Pawn：

| **Pawn类型** | **说明**                                                     |
| ------------ | ------------------------------------------------------------ |
| Pawn         | 这是空的Pawn Actor。它没有附加视觉效果呈现（网格体）或功能按钮。 |
| 默认Pawn     | 这是一种简单的Pawn Actor，带有球形碰撞和内置飞行动作。请注意，你需要向其**附加控制器组件**，然后它才能响应玩家输入。 |

##### 9.6.11.2角色

**角色（Character）** 是一种特定类型的Pawn，设计用于垂直方向的玩家角色，该角色可以在世界中行走、奔跑、跳跃、飞行和游泳。换言之，如果你的玩家控制了一个双足头像（例如，一个人类），该头像将为角色，而不是Pawn。

虚幻引擎中提供了多种类型的角色Actor。设置项目时，请选择最符合项目类型的角色Actor。

| **角色类型**                       | **说明**                                                     | **延伸阅读**                                                 |
| ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 角色（Character）                  | 这是一个简单的角色Actor，带有胶囊体碰撞、空的网格体和角色运动。照现状，此Actor在关卡中不可见，但可以控制。 |                                                              |
| Arch Vis角色（Arch Vis Character） | Arch Vis角色基于虚幻引擎的第一人称角色类。其中某些参数（如扭曲和转身速度）进行了调整，以方便更流畅地移动。此角色针对架构可视化项目进行了优化。 | 使用[Archviz角色Pawn](https://www.youtube.com/watch?v=tLWVDEUtysc) (YouTube) |
| 平面角色（Paper Character）        | 平面角色用于2D项目。平面角色与通用角色之间的区别是，平面角色使用平面图像序列视图组件（一种Sprite）来直观表示角色，而泛型角色使用三维网格体。 | [Paper 2D Sprites](https://docs.unrealengine.com/5.0/zh-CN/paper-2d-sprites-in-unreal-engine) |

#### 9.6.12视觉效果Actor

**视觉效果Actor（Visual Effects Actors）** 用于更改关卡的外观体验。这些Actor只有有限的效果区域，由有限的三维体积定义。

#### 9.6.13后期处理体积

**后期处理体积（Post Process Volume）** *将一个或多个视觉效果应用于其中包含的对象*。虚幻引擎提供了广泛的效果供你应用，从泛光和渐晕到全局光照和光线追踪反射，不一而足。

根据关卡的大小和复杂程度，以及你的设备规格，后期处理可能是一个**极度耗费资源**的过程，并且可能对运行时的性能产生重大影响。

请参阅[后期处理效果](https://docs.unrealengine.com/5.0/zh-CN/post-process-effects-in-unreal-engine)了解更多。

#### 9.6.14反射捕获Actor

**反射捕获Actor（Reflection Capture Actors）** *用于捕获关卡的内容进行反射*。它们将捕获周围区域的静态图像，并根据Actor的形状将该图像映射到球体或盒体形状。在反射捕获Actor的体积中，所有内容的静态图像都会被其周围的反射表面所反射。

虚幻引擎中提供了以下类型的反射捕获Actor：

- 球体反射捕获
- 盒体反射捕获

请参阅[反射环境](https://docs.unrealengine.com/5.0/zh-CN/reflections-environment-in-unreal-engine)页面了解更多。

#### 9.6.15平面反射Actor

**平面反射Actor（Planar Reflection Actor）** 将捕获场景的2D镜像。它非常适合用于创建动态镜面反射，以及捕获不在当前摄像机视图中的事物。

请参阅[平面反射](https://docs.unrealengine.com/5.0/zh-CN/planar-reflections-in-unreal-engine)页面了解更多。

#### 9.6.16贴花Actor

**贴花Actor（Decal Actors）** 可以*放在网格体表面以渲染其顶部的材质*，非常类似于现实世界的**"贴纸"**。你可以使用贴花来向使用相同纹理的多个表面添加细节和变体，例如向模块化墙壁添加漏水或油漆溅污。

请参阅以下页面了解更多：

- [贴花Actor用户指南](https://docs.unrealengine.com/5.0/zh-CN/decal-actors-in-unreal-engine)

- 

  [网格体贴花](working-with-content/MeshDecals/)

#### 9.6.17世界构建Actor

使用 **世界构建Actor（Worldbuilding Actors）** 可向你的关卡添加逼真细节，例天空大气、雾和体积云。

#### 9.6.18天空大气Actor

**天空大气（Sky Atmosphere）** **Actor** 是基于物理的天空和大气渲染技术。它足够灵活，可创建类似地球的大气，展现出以**日出和日落**为特色的昼夜变换，也可以创建具有异世界性质的地外大气。它还提供了空中视角，你可以通过使用*恰当的行星曲率模拟从地面到天空再到外太空的过渡*。

请参阅[天空大气](https://docs.unrealengine.com/5.0/zh-CN/sky-atmosphere-component-in-unreal-engine)页面了解更多。

#### 9.6.19体积云Actor

**体积云Actor（Volumetric Cloud Actor）** 是基于物理的**云**渲染系统，该系统使用材质驱动方法，让美术师和设计师可以自由地创建项目所需的任意类型的云。该系统提供可伸缩的云，适用于使用地面视角、空中视角以及地面到外太空过渡的项目。

请参阅[体积云](https://docs.unrealengine.com/5.0/zh-CN/volumetric-cloud-component-in-unreal-engine)页面了解更多。

#### 9.6.20指数高度雾Actor

**指数高度雾（Exponential Height Fog）** 将在*关卡中海拔较低处创建密度较大的雾，在海拔较高处创建密度较低的雾*。过渡很平滑，因此绝不会随着海拔的增加而出现生硬的界限。

指数高度雾还提供了两种雾颜色，其中一种颜色用于面向主导定向光源的半球（如果不存在这样的光源，则直接向上），另一种颜色用于另一半球。

请参阅[指数高度雾用户指南](https://docs.unrealengine.com/5.0/zh-CN/exponential-height-fog-in-unreal-engine)页面了解更多。

#### 9.6.21文本渲染Actor

**文本渲染Actor（Text Render Actor）** 提供了向关卡添加**文本**的简单方法。要快速了解用法示例，请创建新的第三人称项目。地板上显示的"第三人称（THIRD PERSON）"蓝色文本是使用文本渲染Actor渲染的。

你可以使用 **文本3D（Text 3D）** 插件创建更复杂的3D文本并对其进行动画处理。请参阅[3D文本Actor](https://docs.unrealengine.com/5.0/zh-CN/3d-text-actor-in-unreal-engine)页面了解更多。

#### 9.6.22目标点Actor

**目标点Actor（Target Point Actors）** 提供了世界中的一个泛型点，你可以从中生成项目。如果你熟悉其他3D应用程序（例如3Ds Max或Maya），你就会知道，目标点Actor与这些程序中的虚拟Actor非常相似。

请参阅[目标点Actor](https://docs.unrealengine.com/5.0/zh-CN/target-point-actors-in-unreal-engine)页面了解更多。

#### 9.6.23  3D文本Actor

###### 本文介绍如何在虚幻编辑器中放置3D文本并用它创建动态图形。

你可以使用基于几何体的 **文本3D（Text 3D）** Actor将高分辨率 **3D文本** 添加到关卡中。可在任何想要在虚拟世界中显示清晰、高品质文本的项目中使用3D文本对象，例如直播和虚拟场景。

你可以使用[Sequencer编辑器](https://docs.unrealengine.com/5.0/zh-CN/real-time-compositing-with-sequencer-in-unreal-engine)为3D文本对象制作动画，在虚幻编辑器中直接创建动态图形。

##### 9.6.23.1启用3D文本插件

 插件栏里面搜索

重启编辑器然后在**放置Actor**里面搜索

##### 9.6.23.2文本3D Actor设置

以下选项在 **细节（Details）** 面板的 **3D文本（3D Text）** 分段中可用，可控制3D文本的显示方式：

| **属性**                               | **说明**                                                     |
| -------------------------------------- | ------------------------------------------------------------ |
| **文本（Text）**                       | **Shift+Enter**                                              |
| **挤压（Extrude）**                    | 设置几何体的深度：即文字从前到后的厚度。                     |
| **斜面（Bevel）**                      | 设置沿着字幕边缘的斜面大小。                                 |
| **斜面类型（Bevel Type）**             | 设置边缘斜面的类型：用于锐利、平直斜面的线性或是用于圆形边缘的半圆形。 |
| **半圆分段数（Half Circle Segments）** | 设置创建半圆斜面所使用的分段数。                             |
| **正面材质（Front Material）**         | 选择给字母正面表面着色的材质。                               |
| **斜面材质（Bevel Material）**         | 选择给斜面表面着色的材质。                                   |
| **挤压材质（Extrude Material）**       | 选择给字母侧面着色的材质。                                   |
| **背面材质（Back Material）**          | 选择给字母背面着色的材质。                                   |
| **字体（Font）**                       | [导入字体](https://docs.unrealengine.com/5.0/zh-CN/importing-fonts-in-unreal-engine) |
| **水平对齐（Horizontal Alignment）**   | 根据Actor在3D空间中的位置，将文本水平向左、居中或向右对齐。  |
| **垂直对齐（Vertical Alignment）**     | 根据Actor在3D空间中的位置，将文本在垂直方向上与控件的顶线、顶部、底部或中心对齐。 |
| **字距调整（Kerning）**                | 设置各个字符之间的额外空间。                                 |
| **行距（Line Spacing**                 | 设置各行之间的额外空间。                                     |
| **字距（Word Spacing）**               | 设置各个词之间的额外空间。                                   |
| **最大宽度（Max Width）**              | 设置文本的最大宽度。                                         |
| **最大高度（Max Height）**             | 设置文本的最大高度。                                         |
| **按比例缩放（Scale Proportionally）** | 将字母的高度和宽度锁定为当前比例。一旦启用，对字幕的高度或宽度的任何改动都会同时影响到两者。 |

##### 9.6.23.3逐字母动画处理

你可以让文本3D Actor中的字母的3D平移、旋转和缩放属性在起始值和最终值（可配置）之间进行内插值。你可以设置动画在文本字母间的播放顺序（从左到右、从右到左、从中间字母往外或从外侧字母往内），以及每个字母的动画与相邻字母动画的重叠程度。当这与[Sequencer](https://docs.unrealengine.com/5.0/zh-CN/real-time-compositing-with-sequencer-in-unreal-engine)工具结合使用时，你就能设计出拥有逐字母动画效果的动态示意图形。

文本3D Actor中的逐字母动画由 **Text3DCharacterTransform** 组件控制。你需要将这类组件添加到Actor，并设置其值。

若要设置逐字母动画：

1. 在视口或 **世界大纲视图（World Outliner）** 中选择文本3D Actor。
2. 在 **细节（Details）** 面板中，单击 **添加组件（Add Component）**，并选择 **Text3DCharacterTransform**。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220713165041766.png" alt="image-20220713165041766" style="zoom:50%;" />

   3.在 **细节（Details）** 面板顶部，选择新的 **Text3DCharacterTransform**，以访问其设置。

​    4.启用位置、旋转和/或缩放变换，并调整其设置，以生成所需的动画效果。有关每种设置的说明，请参见下表。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220713165548866.png" alt="image-20220713165548866" style="zoom:50%;" />

| 设置                                    | 说明                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| **启用（Enabled）**                     | 确定 **Text3DCharacterTransform** 组件是否根据此分段中的值更新字母的位置、旋转或缩放。<font color='brown'>启用此选项会需要CPU进行一些额外计算。通常，你应仅为实际需要动画处理的变换类型启用此设置。</font> |
| **进度（Progress）**                    | 确定文本动画在 **开始（Begin）** 和 **结束（End）** 状态之间的总进度。位于 `0` 时，文本的位置、旋转或缩放处于其 **开始（Begin）** 状态。位于 `100` 时，文本的位置、旋转或缩放处于其 **结束（Begin）** 状态。两者之间的值在 **开始（Begin）** 和 **结束（End）** 状态之间按比例内插位置、旋转或缩放。<font color='green'>如果你想创建关卡序列，让字母逐个随着时间播放动画，那么你通常可以在关卡序列中使用这种设置处理设置。</font> |
| **顺序（Order）**                       | **确定文本字母在播放动画时的顺序**。**正常（Normal） -** 动画从最左侧字母开始，并向右移动。 **从中心开始（From Center） -** 动画从文本中心的字母开始，并朝两个方向向外移动。 **向中心靠拢（To Center） -** 动画从文本的最外侧字母开始，并朝中间的字母向内移动。 **反向（Opposite） -** 动画从最右侧字母开始，并向左移动。 |
| **范围（Range）**                       | 确定相邻字母之间的动画同步程度。位于 `0` 时，每个字母会在下一个字母开始变换之前，完成从 **开始（Begin）** 状态到 **结束（End）** 状态的变换。位于 `100` 时，所有字母会同时开始和结束其转换。介于两者之间的值，例如位于 `50` 时，则会让相互衔接的字母在动画播放的时间上产生部分重叠。 |
| **开始（Begin）**                       | **开始（Begin）**                                            |
| **结束（End）** 或 **距离（Distance）** | 为字母的位置、旋转或缩放设置所需的结束状态。                 |
|                                         |                                                              |

5.通常，你需要用 **关卡序列（Level Sequence）** 播放你设计的动画效果。这通常涉及将文本3D Actor添加到关卡序列中，为 **进度（Progress）** 设置创建新轨迹，然后在这些轨迹上创建关键帧，让数值随着时间在0到100之间变化。例如：

![关卡序列中的动画进度设置](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/actors/actors-reference/3d-text-actor/text3d-sequence.jpg)

有关创建关卡序列和在Sequencer编辑器中操作的细节，请参见[Sequencer](https://docs.unrealengine.com/5.0/zh-CN/real-time-compositing-with-sequencer-in-unreal-engine)文档。

#### 9.6.24贴花Actor

延迟贴花（Deferred Decal）提供了更好的性能和维护性。通过写入GBuffer而非重新计算光照，它具备了几点优势：

- 许多光源的性能变得更加可预测，因为对光源数量或类型没有限制（所有光源都使用相同的代码路径）
- 使用屏幕空间遮罩时，还可以实现一些在其他情况下很难实现的效果（比如潮湿层）。

贴花的渲染方式是，**在贴花影响区域的周围的一个方框中渲染**。

##### 9.6.24.1在关卡中添加贴花

在场景中添加贴花的方法是：在 **内容浏览器（Content Browser）** 中选择合适的**贴花材质**，然后在视口中 **右键单击**，在上下文菜单中选择 **放置Actor（Place Actor）**。然后可以使用变形工具调整贴花的大小和方向。

[贴花Actor用户指南 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/decal-actors-in-unreal-engine/)

#### 9.6.25玩家出生点

无论是哪款游戏，能在场景任意位置生成玩家都是一项重要功能。虚幻引擎 4 通过一个特殊 Actor 来实现此功能，称之为 **玩家出生点（Player Start）**。所谓"玩家出生点"，就是指在游戏场景中的玩家初始位置。

**放置玩家出生点 Actor**

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220713173056797.png" alt="image-20220713173056797" style="zoom:50%;" />

##### 9.6.25.1玩家出生点的使用技巧

玩家出生点的使用方法非常简单，了解以下技巧能使开发过程更容易。

**No Player Start（无玩家出生点）：** 如开始游戏时世界场景中不存在玩家出生点，玩家角色在世界场景中出现的坐标将为 0,0,0。正因如此，请务必在世界场景中放置玩家出生点。

**Play From Here（从此处开始）：** 也许有时需要从非玩家出生点的其他位置开始游戏。在编辑器视口中 **单击右键**，选择 **Play From Here（从此处开始）** 选项，即可实现该功能。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220713185304286.png" alt="image-20220713185304286" style="zoom:50%;" />

**Bad Size（尺寸错误）:** 有时玩家出生点的控制器图标可能会变成一个 "BADsize" 字样的图标。出现此情况时，须在世界场景中移动玩家出生点，避免其与场景对象重叠。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220713185519122.png" alt="image-20220713185519122" style="zoom:50%;" />

#### 9.6.26触发器Actor

可用于激活并触发关卡事件的Actor。

**触发器（Triggers）** 属于Actor，当它们与关卡其他对象交互时，可以触发关卡事件。换而言之，它们负责响应关卡对象的动作并触发事件。所有触发器都差不多，区别在于形状不同——有盒体、胶囊体和球体——**触发器通过这些形状来判断其他对象是否碰撞并激活了它**。

##### 9.6.26.1触发事件

触发器用于激活放置在[关卡蓝图](https://docs.unrealengine.com/5.0/zh-CN/level-blueprint-in-unreal-engine)中的事件。触发器可以激活几种不同类型的事件。主要类型的事件用于响应与另一个对象的某种类型的碰撞，例如某物与触发器碰撞或重叠，或响应来自玩家的输入。

当在 **视口（Viewport）** 中选择了触发器时：

- 在 **关卡蓝图事件图表** 中 **单击右键**，并在 **为[触发器Actor名称]添加事件（Add Event for [Trigger Actor Name]）** 下选择一个事件。

![触发器时间情境菜单](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/actors/actors-reference/trigger-actors/trigger_event_context.jpg)

通过这两种方法中的任何一种选择一个事件，都会将一个[事件节点](https://docs.unrealengine.com/5.0/zh-CN/events-in-unreal-engine) 添加到当前关卡的关卡蓝图中：

<img src="https://docs.unrealengine.com/5.0/Images/understanding-the-basics/actors/actors-reference/trigger-actors/trigger_event.jpg" alt="蓝图中的触发器事件" style="zoom: 67%;" />

每当发生该事件，都会触发该事件节点的执行引脚

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220713190632070.png" alt="image-20220713190632070" style="zoom:50%;" />（蓝图里面的)

#### 9.6.27骨骼网格体Actor

**骨架网格体 Actor** 显示一个动画网格体，其几何体可以变形，通常是通过在动画序列期间使用控制点来实现的。这些可以从外部3D动画应用程序创建和导出，也可以直接在虚幻引擎中编程。

顾名思义，骨骼网格体具有由许多**骨骼**组成的**骨架**。这些用于动画过程。

骨架网格体 Actor 通常用于表示玩家角色、NPC、其他动画生物和复杂机械。

##### 9.6.27.1为骨架网格体 Actor 制作动画

在虚幻引擎中，有两种基本方法可以制作骨架网格体 Actor 的动画效果：

- 使用[动画蓝图](https://docs.unrealengine.com/5.0/zh-CN/animation-blueprints-in-unreal-engine)将多个动画播放和混合在一起。
- 使用动画资源播放单个[动画序列](https://docs.unrealengine.com/5.0/zh-CN/animation-sequences-in-unreal-engine)一次或在循环中播放。

要了解有关为骨架网格体制作动画的详细信息，请参阅[骨架网格体动画系统](https://docs.unrealengine.com/5.0/zh-CN/skeletal-mesh-animation-system-in-unreal-engine)页面。

##### 9.6.27.2骨架网格体Actor碰撞

正常的碰撞创建和检测不适用于骨架网格体 Actor。要让你的骨架网格体与关卡中的对象发生碰撞，你的骨架网格体Actor需要有一个专门为他们创建**的物理资源**。

要了解有关物理资源及其使用的更多信息，请参阅 [制作交互式体验/物理/物理资产编辑器）文档

要创建物理资源并将其分配给骨架网格体 Actor，请执行以下步骤：

1. 在**内容浏览器中**找到**骨架网格体**，然后右键单击它。
2. 在**上下文菜单中**，选择**创建>物理场资产>创建和分配**”。

![创建物理资源并将其分配给骨架网格体](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/actors/actors-reference/skeletal-mesh-actors/creating-physics-asset.jpg)

#### 9.6.28静态网格体Actor

**静态网格体 Actor** 是一种简单类型的 Actor，可在关卡中显示 3D 网格体。虽然名称暗示Actor是静态的（或无法移动），但“静态”是指使用的网格体类型，而不是Actor的移动能力。**如果网格的几何图形不更改，则该网格是静态的。否则，Actor本身可以在游戏过程中以其他方式移动或更改。**

静态网格体Actor通常用于创建游戏世界或其他类型的环境。

##### 9.6.28.1在游戏过程中移动静态网格体 Actor

Actor的**移动性**设置控制Actor在游戏过程中是否可以以某种方式移动或更改。

默认情况下，静态网格体 Actor 具有**静态**移动性。要在播放过程中移动、旋转或缩放 StaticMeshActor，必须先将其设置为**可移动**。为此，请在“使用者**的”详细信息“**面板中，将其**”移动性****“设置为”可移动**”。

##### 9.6.28.2为静态网格体 Actor 启用物理特性

如果希望静态网格体 Actor **受重力和碰撞的影响**，请在“Actor **的”详细信息“**面板中启用**”模拟物理**场“属性。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220713192831437.png" alt="image-20220713192831437" style="zoom:50%;" />

#### 9.6.29摄像机Actor

了解虚幻引擎中摄像机的运用

[虚幻引擎中的摄像机Actor | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/camera-actors-in-unreal-engine/)

虚幻引擎中有如下几种摄像机Actor：

- **摄像机Actor（Camera Actor）**, 通用摄像机，可以用作固定或者移动的视口。
- **电影摄像机Actor（Cine Camera Actor）**, 用于制作过场动画的专用摄像机。详情参阅[电影摄像机Actor（Cine Camera Actor）](https://docs.unrealengine.com/5.0/zh-CN/cinematic-cameras-in-unreal-engine)。

#### 9.6.30体积Actor

**体积** 是一种三维 **Actor**，用于调整关卡中指定区域的行为。例如，体积可用于下述操作：

- 对体积中的玩家或其他Actor造成伤害。
- 作为碰撞体积，阻止特定Actor进入体积。
- 当某个Actor进入体积时打开门。
- 更改关卡计算其光照或可视性的方式。


1.体积是一种辅助型Actor，通常用于检测某类Actor是否进入了指定区域，并触发效果作为响应。

2.体积有时自带一些效果（通过代码或蓝图提供），但多数时候，它们只负责为其他Actor提供提示。一般，应将它们视为更大的系统的组成部分（以某些元素作为视觉提示）。

##### 9.6.30.1阻挡体积

阻挡体积（Blocking Volume）可用于表示静态网格体的碰撞表面，比如用于墙壁。它可以增强场景的可预测性，因为物理对象能避免与地板、墙壁等表面上的凸起细节碰撞。它还能降低物理模拟开销，提高性能。

##### 9.6.30.2摄像机阻挡体积

**摄像机阻挡体积（Camera Blocking Volume）** 采用了预置的碰撞设置，可阻挡摄像机，并忽略摄像机以外的所有Actor体积。它们用于定义不可见边界，可以将摄像机限制在合理范围内。例如，在第三人称游戏中，游玩区域的墙壁上具有多叶藤蔓等装饰物。在这些情况下，可贴着墙壁放置一些很薄的摄像机阻挡体积（Camera Blocking Volume），确保摄像机不会撞进藤蔓或伸到叶片后面，使得它流畅运动，确保游戏画面不会被干扰。

##### 9.6.30.3销毁Z体积

**销毁Z体积（Kill Z Volume）** 旨在防止某些游戏的对象越界，例如，避免从悬崖上掉落或掉进深坑，或者在科幻设定中在不穿宇航服的情况下离开太空船。每当有Actor进入销毁Z体积（Kill Z Volume）的任何Actor，该体积都将调用 `FellOutOfWorld` 函数。默认情况下，这些Actor会快速执行清理程序并销毁自身。你也可以根据游戏需求，**重载Actor的销毁行为**。例如，如果钥匙掉进了熔岩坑（玩家必须收集才能继续游戏），游戏可能需要将钥匙传送回玩家可以到达的区域而非销毁该物品，或者告知玩家该物品已销毁并重新加载上一个检查点，而非任由游戏停留在无法获胜的状态。

##### 9.6.30.4伤害施加体积

**伤害施加体积（Pain-Causing Volume）**[伤害施加体积参考](https://docs.unrealengine.com/5.0/zh-CN/pain-causing-volume-actor-in-unreal-engine)

##### 9.6.30.5物理体积

**物理体积（Physics Volume）** 是一种可以对范围内角色及对象产生物理效果的体积。它们的一个常见用处是创建**供玩家游动的水环境**。物理体积（Physics Volume）的效果可以是可见的。角色移动组件（Character Movement Component）类会根据体积的力场来调整 `角色` **在环境中的移动方式**。[物理体积参考](https://docs.unrealengine.com/5.0/zh-CN/physics-volume-actor-in-unreal-engine)

##### 9.6.30.6触发器体积

**触发器体积（Trigger Volume）** 可以在玩家或其他对象进入或离开它们时触发事件。你可以在 **关卡蓝图** 中用它们来快速测试事件效果和游戏逻辑，而无需其他蓝图。

例如，你可以在关卡中放置一个触发器体积，然后在 **关卡蓝图** 中为该体积创建一个重叠事件，以此播放声音，打开门，或启动过场动画。

<font color='red'>请务必检查碰撞设置，以确保触发器会响应碰撞并对目标Actor做出反应。</font>

##### 9.6.30.7音频体积

**音频体积（Audio Volume）**[音频体积参考](https://docs.unrealengine.com/5.0/zh-CN/audio-volume-actor-in-unreal-engine)

##### 9.6.30.8剔除距离体积

**剔除距离体积（Cull Distance Volume）** 是一种优化工具，可根据对象距摄像机的距离及其尺寸，对对象进行剔除（即不绘制到屏幕上）。当对象小到可被视为不重要时，即可不绘制对象，从而优化场景。尺寸是按照边界框的最长边计算的，而剔除距离是根据与该尺寸最接近的距离计算的。

剔除距离体积（Cull Distance Volume）设置取决于 **剔除距离（Cull Distances）** 属性，如以下图中的 **细节（Details）** 面板截图所示。

![CullDistancesProperty.png](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/actors/actors-reference/volume-actors/CullDistancesProperty.jpg)

属性定义了下列行为：

- 体积内大小最接近50单位（距离等于或小于85单位）的对象会在它们距摄像机500单位或更远时被剔除（消失）。
- 体积内大小最接近120单位（距离介于85单位和210单位之间）的对象会在它们距摄像机1000单位或更远时被剔除（消失）。
- 体积内大小最接近300单位（距离等于或大于210单位）的对象将永不会被剔除，因为在这种情况下，0被当作无穷大，这意味着永远不可能距摄像机足够远。

设置时，请先向"剔除距离（Cull Distances）"数组中添加一个新元素，方法是单击 ![button_Plus.png](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/actors/actors-reference/volume-actors/button_Plus.jpg) 按钮。接下来，只需填入给定体积内对象的大小以及你希望该尺寸或小于该尺寸的对象被剔除时所处的距离。无需按照特定的顺序创建这些属性。

##### 9.6.30.9HLOD体积

**层级LOD体积（Hierarchical LOD Volume）** 被[层级细节](https://docs.unrealengine.com/5.0/zh-CN/hierarchical-level-of-detail-in-unreal-engine)（HLOD）系统用来将Actor划分到单个HLOD群集。生成群集时，虚幻引擎将覆盖常规的生成过程，转而遵循手动放置的体积。

##### 9.6.30.10Lightmass角色间接细节体积

**Lightmass角色间接细节体积（Lightmass Character Indirect Detail Volume** 与Lightmass重要体积（Lightmass Importance Volume）相似，它会在玩家高度（相对于地面）以及在整个体积中生成间接光线样本。此类体积的一个用处就是放置在电梯井中，确保沿电梯井的所有间接光照都正确，而非只有底部的间接光照正确。

##### 9.6.30.11Lightmass重要体积

**Lightmass重要体积（Lightmass Importance Volume）**[Lightmass全局光照](https://docs.unrealengine.com/5.0/zh-CN/global-illumination-in-unreal-engine)

##### 9.6.30.12网格体合并剔除体积

**网格体合并剔除体积（Mesh Merge Culling Volume）** 标记它们包含的网格体对象，以使这些对象合并成单个网格体。**通过将合并区域内的一系列小型网格体作为一个网格体**进行剔除或引起[HLOD](https://docs.unrealengine.com/5.0/zh-CN/hierarchical-level-of-detail-in-unreal-engine)生成以通过更优的方式减少几何体的数量，它可以提高性能。

##### 9.6.30.13后期处理体积

通过调整 **细节（Details）** 面板中的属性应用给摄像机的后期处理设置，可以被 **后期处理体积（Post Process Volume）**[后期处理文档](https://docs.unrealengine.com/5.0/zh-CN/post-process-effects-in-unreal-engine)

##### 9.6.30.14预计算可视性体积

**预计算可视性体积（Precomputed Visibility Volume）** 主要用于性能优化。这类体积会保存Actor的可视性，以了解它们在场景中的位置。应仅将这些体积放置在玩家可以到达的区域。

##### 9.6.30.15预计算可视性覆盖体积

如果预计算可视性体积（Precomputed Visibilty Volume）的自动生成结果不理想，你可以使用 **预计算可视性覆盖体积（Precomputed Visibility Override Volume）** 手动覆盖Actor的可视性，以了解它们在场景中的位置。这些体积也用于性能优化，而且应仅将这些体积放置在玩家可以到达的区域。

##### 9.6.30.16关卡流送体积

**关卡流送体积（Level Streaming Volume）** 用于协助 [关卡流送](https://docs.unrealengine.com/5.0/zh-CN/level-streaming-in-unreal-engine) 过程。它们提供了一种封装关卡以及基于玩家进入或离开体积的时间控制它何时切入和切出**内存**的简便方法。

##### 9.6.30.17寻路网格体边界体积

可使用 **寻路网格体边界体积（Nav Mesh Bounds Volume）** 来控制在关卡中构建寻路网格体的位置。**寻路网格体用于计算在关卡不同区域的寻路路径。**

在该体积中，会在所有表面上构造寻路网格体，其角度合理，适于在其上行走。你可以按照需要重叠所需数量的此类体积以生成所需的寻路网格体。

使用寻路网格体边界体积（Nav Mesh Bounds Volume）的方法是创建一个或多个将关卡的**可寻路区域**包围在其中的寻路网格体边界体积（Nav Mesh Bounds Volume）。寻路网格体将自动被构建。

通过在视口中按 **P** 键，你可以随时使寻路网格体可见。

[寻路网格体内容示例](https://docs.unrealengine.com/5.0/zh-CN/content-examples-sample-project-for-unreal-engine)

#### 9.6.31目标点Actor(*)

目标点 Actor 的创建和使用指南。

在为游戏创建场景时，有时你需要在其中放置和生成各种物体，以便玩家与之互动。**目标点 Actor（Target Point Actor）** 的作用正在于此，**在世界场景中设定一个一般点，作为物体生成的点**。如你对其他 3D 软件（如 3Ds Max 或 Maya）有所了解，便会发现目标点 Actor 与这些软件中的虚拟 Actor 十分相似。

##### 9.6.31.1使用目标点

目标点 Actor 在虚幻引擎 4 中的用途十分广泛。下面是它的部分用途：

- 设定动画序列中摄像机对准的目标。
- 设定 AI 路径点。
- 设定 VFX（视觉特效）生成点。
- 设定可拾取道具（如回复品和物品）生成点。
- 设定世界场景中道具所在点/应放置点的视觉提示。

下述蓝图示例介绍了如何使用目标点 Actor 来指定生成点的位置。

![image-20220713203809229](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220713203809229.png)

## 10.组件

**组件（Component）** 是可以添加给Actor的一项功能。

当你为Actor添加组件后，Actor就可以使用组件提供的功能。例如：

- "聚光源（Spot Light）组件"将使Actor像聚光源一样发光。
- "旋转运动（Rotating Movement）组件"将使Actor旋转。
- "音频（Audio）组件"将使Actor能够播放声音。

与Actor不同，**组件不能单独存在**。它们只能**绑定**在Actor身上。

为Actor添加组件的过程，<font color='brown'>相当于为Actor添加各个零件</font>。例如，一辆汽车（Actor）由车轮、方向盘、车身、车灯等（组件）组成。再例如，玩家角色Actor通常包含一个单独的"骨骼网格体（Skeletal Mesh）"组件（表示角色外观）、跟随角色移动的摄像机，以及接收玩家输入的控制器。

添加完构成Actor的不同组件后，即使不提供指示Actor应如何运行的任何 **蓝图（Blueprint）** 脚本（或C++代码），也可以将Actor放置在关卡中。在上面的示例中，汽车（Actor）可以作为对象存在于世界（关卡）中，无需任何驱动程序（蓝图或C++代码）告诉它要执行什么操作。然后，可以使用蓝图或C++代码单独访问汽车的每个组件（例如，如果按下油门组件，蓝图逻辑可以使汽车加速）。

#### 组件实例化

与一般子对象的默认行为不同，**Actor中充当子对象的各种组件都是实例化的**，所有某个类的Actor实例都有着单独的组件实例。

为了理解这点，想象一下上面的汽车示例。一个"汽车（Car）"类使用组件来表示汽车的车轮。四个"车轮（Wheel）"组件是该类的默认属性中的四个子对象，并指定给了"车轮（Wheels）"数组。当创建新的汽车实例时，会专门为该汽车实例新建"车轮（Wheel）"组件实例。否则，当世界中一辆汽车移动时，所有汽车的车轮都会转动；这显然不是预期的行为。让组件默认进行实例化，免去了为Actor添加唯一子对象的麻烦。

<font color='brown'>如果没有组件实例化，所有组件变量都需要用 `Instanced` [属性说明符](https://docs.unrealengine.com/5.0/zh-CN/unreal-engine-uproperty-specifiers)进行声明。</font>

#### 10.1AI组件

##### 10.1.1AI感知组件

在 **AIPerceptionSystem** 中，**AIPerceptionComponent** 相当于刺激源的监听器，用于收集已注册的刺激信号。当组件获得新的刺激信号（批量）时，将会调用 **UpdatePerception**。

##### 10.1.2Pawn噪声发生器组件

**PawnNoiseEmitterComponent** 追踪 **SensingComponents** 使用的噪声事件数据来监听Pawn。该组件主要存在于 Pawn 或其控制器上。它在网络客户端上不进行任何操作。

##### 10.1.3Pawn感应组件

Pawn的感应组件（Sensing Component）用于封装Actor的感知（例如视觉和听觉）设置及功能，以便Actor在游戏世界中观察/监听Pawn。其在网络客户端上不进行任何操作。

#### 10.2音频组件

**音频组件（AudioComponent）** 用于创建和控制游戏内部声音的实例。

音频组件允许你在Actor下面添加一个 [Sound Wave](https://docs.unrealengine.com/5.0/zh-CN/audio-files-in-unreal-engine) 或 [Sound Cue](https://docs.unrealengine.com/5.0/zh-CN/audio-engine-overview-in-unreal-engine#soundcue) 作为子对象，从而提供声源。

例如，假如你想用粒子效果表现关卡中的火焰。除了用粒子表现火焰的视觉效果，你还可以为火焰添加一个音频组件作为它的子Actor，播放熊熊烈火的音效，使得着火效果更加逼真。

在运行时，你还能用蓝图或 C++ 来动态更改音频组件的属性或设置，例如，实现音频的淡入淡出效果、播放或停止播放音效、调节音量和其他选项（可在各类属性分段的[Sound Actors](https://docs.unrealengine.com/5.0/zh-CN/ambient-sound-actor-user-guide-in-unreal-engine) 页面中查看）。

#### 10.3缆绳组件

假如能以较低的开销添加来回晃荡的缆绳或锁链，**仿佛被风吹动一般**，那么虚幻引擎的项目会更加栩栩如生。本文将介绍如何使用 **缆绳组件（Cable Component）** 插件创建和控制缆绳的外观、响应方式，甚至是让它和关卡中的物体发生碰撞。

##### 10.3.1模拟和渲染

在模拟线缆的实际效果时，我们用到了游戏开发中的一项经典技巧——**韦尔莱积分算法（Verlet Integration）**。这种算法的理念是用一系列粒子来表示缆绳，且粒子之间拥有 **距离约束**。两端的粒子 **固定不动**，只跟随它的绑定对象移动。中间的粒子则 **自由移动**，随重力下垂。在每一步（积分）中，每个粒子的速度和位置都会更新，并相应移动以满足约束。缆绳的 **刚性** 由我们（每步）强制执行约束时采用的迭代次数决定。

![image-20220717102847810](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717102847810.png)

生成完粒子锁链后，我们需要对它进行渲染。渲染缆绳时，将创建一个名为 **FCableSceneProxy** 的类来表示缆绳的渲染效果。粒子的两端的位置（在主线程上的 TickComponent 中执行）会通过 **SendRenderDynamicData_Concurrent** 函数传递给此代理。接下来，更新（update）会在渲染线程锁定，之后将更新索引和顶点缓存，进而生成一个 **管状** 网格体。管状网格体上的每个顶点都要计算一个位置、一个纹理UV 和三个切线基础（Tangent Basis）向量。执行此操作时，**X轴** 的方向同缆绳，**Z轴** 垂直缆绳表面（相当于法线），而 **Y轴** 则与 **X轴** 和 **Z轴** 垂直。这些属性对组件公开，以便用户控制面的数量，管状的半径、以及 UV 沿缆绳进行平铺的次数。

![image-20220717110854803](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717110854803.png)

##### 10.3.2启用插件

缆绳组件插件会默认启用。如未启用，可在主工具栏中选择 **编辑（Edit）** > **插件（Plugins）**。然后在插件列表中选择 **Rendering**，勾选 **Cable Component** 的 **Enabled** 勾选框。

![image-20220717111111464](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717111111464.png)

##### 10.3.3使用缆绳组件

可以两种不同方式将缆绳组件添加到项目关卡。在以下部分，我们将说明为项目关卡添加缆绳的两种不同方法。

##### 10.3.4用放置Actor面板添加缆绳组件

如需使用放置Actor面板添加缆绳组件，请执行以下操作：

1. 首先确保 **放置Actor（Place Actors）** 面板为显示状态，然后在 **Search Classes** 框中输入 `Cable`。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717111410431.png" alt="image-20220717111410431" style="zoom:50%;" />

2.  如要将缆绳 Actor 添加到世界场景，请选中"放置Actor"面板中的缆绳 Actor 并将其拖入关卡。

##### 10.3.5在蓝图中使用缆绳组件

如需在蓝图中使用缆绳组件，请执行以下操作：

1. 首先新建一个名为 **BP_Cable** 的蓝图，它的父类是 **Actor** 。

​    2.然后在 BP_Cable 蓝图的 **Components** 分段中点击 **Add Component** 按钮，然后在列表中找到 **Cable** 组件。找到组件后，点击将其添加至组件列表。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717111921961.png" alt="image-20220717111921961" style="zoom:50%;" />

##### 10.3.6在缆绳末端附加物体

你可以在缆绳的任意一端附加物体，使其沿缆绳晃动。需要执行以下步骤在 UE4 中实现此功能：

1. 首先需要添加一个 **缆绳 Actor** 和一个 **静态网格体** 到关卡中。

必须将附加到末端的静态网格体的 **Mobility** 设为 **可移动（Moveable）**

   2.在世界大纲视图中，找到需要附加到缆绳 Actor 末端的静态网格体，然后将其拖至缆绳 Actor 的下方。执行此操作后将显示以下输入窗口。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717112513074.png" alt="image-20220717112513074" style="zoom:50%;" />

   3.选择 **Cable End** 选项，即可在视口中看到静态网格体附着到缆绳 Actor 的末端。 

4.在关卡中选中缆绳 Actor。然后在 **Details** 面板的 **Cable** 部分中，取消勾选 **Attach End** 框。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717130632526.png" alt="image-20220717130632526" style="zoom:50%;" />

**Attach Start** 和 **Attach End** 选项并非将缆绳附加到 Actor 的唯一方法。也可指定一个用作附着点的套接字。

可在运行时开启/关闭 **Attach Start** 和 **Attach End** 两个布尔值，制造出一些有趣的效果。

##### 10.3.7碰撞和刚性

```
启用碰撞（Collision）和刚性（Stiffness）将极大增加缆绳 Actor 的开销。建议只在以下情况下启用：缆绳需要和场景中的物体碰撞，或需要为缆绳赋予刚性以获得更佳效果。如无此类需求，则最好将这些选项禁用，以降低开销。
```

缆绳组件有一个选项可让缆绳**和场景对象发生碰撞**，并控制缆绳的刚度。为启用此功能，请按以下步骤操作。

1. 首先，在缆绳组件细节面板的 Cable 分段中按下白色小三角形，展开高级选项。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717142542358.png" alt="image-20220717142542358" style="zoom:50%;" />

2.勾选 **Enable Stiffness(启用刚度)** 和 **Enable Collision(启用碰撞)** 选项启用这些功能。

3.现在，当缆绳 Actor 或物体相遇时，应该会发生碰撞效果。(实测有效)

##### 10.3.8属性详解

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717145344649.png" alt="image-20220717145344649" style="zoom:50%;" />

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717145426087.png" alt="image-20220717145426087" style="zoom:50%;" />

| **Attach Start**              | 附加到物体的头端，或放任不管。如为 false，组件变形只用于缆绳头端的初始位置。 |
| ----------------------------- | ------------------------------------------------------------ |
| **Attach End**                | 修复到物体的末端（使用 AttachEndTo 和 EndLocation），或放任不管。如为 false，AttachEndTo 和 EndLocation 只用于缆绳末端的初始位置。 |
| **Attach End To**             | 定义缆绳最终位置的 Actor 或组件。                            |
| **Component Property**        | 附加缆绳的组件的命名。                                       |
| **Attach End To Socket Name** | 进行附加的 AttachEndTo 组件上的套接字命名。                  |
| **End Location**              | 缆绳的最终位置，如经指定则相对于 AttachEndTo（或 AttachEndToSocketName），否则则相对于缆绳组件。 |
| **Cable Length**              | 缆绳的静止长度。                                             |
| **Num Segments**              | 缆绳拥有的分段数量。                                         |
| **Solver Iterations**         | 解算器迭代的数量，控制缆绳的刚度。                           |
| **Substep Time**              | 控制缆绳的模拟分步时间。                                     |
| **Enable Stiffness**          | 为缆绳添加刚性约束。                                         |
| **Enable Collision**          | 在每个分步为每个缆绳粒子执行清扫，避免与世界场景发生碰撞。使用组件上的 Collision Preset 决定碰撞的对象。这会极大提高缆绳模拟的开销。 |

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717150629018.png" alt="image-20220717150629018" style="zoom:50%;" />

##### 10.3.9缆绳力

| **属性名称**            | **描述**                                   |
| ----------------------- | ------------------------------------------ |
| **Cable Forces**        | 应用到缆绳中所有粒子的力向量（世界空间）。 |
| **Cable Gravity Scale** | 应用到此缆绳的世界重力比例。               |



![image-20220717150738741](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717150738741.png)

##### 10.3.10缆绳渲染

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717151732990.png" alt="image-20220717151732990" style="zoom:80%;" />

| **属性名称**      | **描述**                 |
| ----------------- | ------------------------ |
| **Cable Width**   | 缆绳几何体的宽度。       |
| **Num Sides**     | 缆绳几何体的面数。       |
| **Tile Material** | 沿缆绳长重复材质的次数。 |

#### 10.4摄像机组件

**摄像机组件（CameraComponent）**（用于添加摄像机视角）和 **弹簧臂组件（SpringArmComponent）**（让摄像机与拍摄对象保持固定距离，并且在遇到遮挡时动态调整）的出现，使得游戏中的第三人称视角能够动态适应游戏场景。

##### 10.4.1摄像机组件

摄像机组件（CameraComponent）用于为 **Actor** 绑定一个摄像机视角子对象。如果 **视图目标（ViewTarget）** 是一个 **摄像机Actor（CameraActor）** ，或是一个包含摄像机组件且将 **查看目标时查找摄像机组件（Find Camera Component When ViewTarget）** 选项设置为"真"的Actor，则摄像机组件将提供摄像机属性的信息。

例如在游戏中，你可以用摄像机组件[在多个摄像机之间切换](https://docs.unrealengine.com/5.0/zh-CN/quick-start-guide-to-implementing-automatic-camera-control-in-unreal-engine-cpp)。（多个不同的Actor的摄像机，不是同一个Actor的摄像机）借助 **使用混合设置视图目标（Set View Target With Blend）** 和摄像机Actor，你可以在你的各个摄像机之间切换，并调整每个摄像机Actor中的属性（包括视场、角度、后期处理效果等）。

**拥有时取得摄像机控制权（Take Camera Control When Possessed）** 是一项可以为任何 **Pawn** 设置的属性，此属性使Pawn在被PlayerController拥有时，自动成为视图目标。所以打个比方，如果你有多个角色（角色也是Pawn的一种），并且希望在这些角色之间切换，同时每个角色都分配有自己的摄像机组件以提供摄像机视角，那么你可以把每个角色的"拥有时取得摄像机控制权（Take Camera Control When Possessed）"都设置为"真"，每当你切换角色时，系统都将自动使用该Pawn的摄像机组件。

##### 10.4.2弹簧臂组件

弹簧臂组件（SpringArmComponent）能使它的子对象与父对象之间保持固定距离；**但是如果发生遮挡，将缩短这段距离**。遮挡消失后，距离又会恢复正常。通常，弹簧臂组件（SpringArmComponent）充当"摄像机升降臂"，能防止玩家的跟随摄像机与场景对象发生遮挡（假如没有无弹簧臂组件，摄像机组件仍将保持固定距离，无论当前是否有遮挡物），请参阅[使用弹簧臂组件](https://docs.unrealengine.com/5.0/zh-CN/camera-components-in-unreal-engine)。

你可以调整弹簧臂组件上的几个摄像机相关属性，例如：**目标臂长（TargetArmLength）**，即没有发生碰撞时弹簧臂的自然长度；**探头大小（Probe Size）**，即检查碰撞时探针球应该多大；以及诸如 **摄像机滞后时间（CameraLag）** ，用于让**摄像机的移动略微滞后于它跟随的对象**。<font color='red'>这个很有用</font>

#### 10.5光源组件

你可以根据需求，为Actor添加各类 **光源组件（Light Components）** 作为子对象。所有光源组件都提供了一些通用设置（光源颜色或强度）以及特有设置（参见各类光源的特定设置）。

##### 10.5.1定向光源组件

**定向光源组件（DirectionalLightComponent）** 模拟了从无穷远的光源发射出的光源。这也就是说，**首此光源投射出的所有阴影都是平行的，使其成为模拟<font color='red'>阳光</font>的最佳选择。**

请参见 [定向光源](https://docs.unrealengine.com/5.0/zh-CN/directional-lights-in-unreal-engine) 以了解更多信息。

##### 10.5.2点光源组件

**点光源组件（PointLightComponent）** 十分类似现实世界中的**灯泡**——以灯泡钨丝为中心，向周围发射光源。但是，为了提高运行性能，点光源组件被简化了。它只从空间中的一个点均匀地向所有方向发射光线。

请参见 [点光源](https://docs.unrealengine.com/5.0/zh-CN/point-lights-in-unreal-engine) 以了解更多信息。

##### 10.5.3天空光源组件(天光)

**天空光源组件（SkyLightComponents）** 会采集关卡中的远距离对象（距离大于 SkyDistanceThreshold 的所有对象），并将其作为光源应用到场景中。这也就是说，天空的外观及其光线/反射将能保持一致。

请参见 [天空光照](https://docs.unrealengine.com/5.0/zh-CN/sky-lights-in-unreal-engine) 以了解更多信息。

##### 10.5.4聚光源组件

**点光源组件（PointLightComponent）** 十分类似现实世界中的灯泡——以灯泡钨丝为中心，向周围发射光源。但是，为了提高运行性能，点光源组件被简化了。它只从空间中的一个点均匀地向所有方向发射光线。

请参见 [点光源](https://docs.unrealengine.com/5.0/zh-CN/point-lights-in-unreal-engine) 以了解更多信息。

#### 10.6移动组件（*）

可实现任何形式的移动，无论是角色、还是使用了移动组件的发射物。

**移动组件（Movement Components）** 能为所属的 Actor（或角色）提供移动功能。

##### 10.6.1人物移动组件

character类独有且自带的组件

**角色移动组件（CharacterMovementComponent）** 允许非物理刚体类的角色移动（走、跑、跳、飞、跌落和游泳）。 该组件专用于 **角色（Characters）**，无法由其他类实现。当你创建一个继承自 **Characters** 的 **蓝图（Blueprints）** 时，CharacterMovementComponent会自动添加，无需你手动添加。

组件包含一些可设置属性，包括角色跌落和行走时的摩擦力、角色飞行、游泳、以及行走时的速度、浮力、重力系数、以及人物可施加在物理对象上的力。 CharacterMovementComponent还包含动画自带的、且已经转换成世界空间的根骨骼运动参数，可供物理系统使用。请参见[根运动](https://docs.unrealengine.com/5.0/zh-CN/root-motion-in-unreal-engine) 以了解更多信息。

有关使用人物移动的信息，请参见[设置角色动作](https://docs.unrealengine.com/5.0/zh-CN/setting-up-character-movement)。

##### 10.6.2发射物移动组件

在更新（Tick）过程中，**发射物移动组件（ProjectileMovementComponent）** 会更新另一个组件的位置。该组件还支持碰撞后的跳弹以及跟踪等功能。 通常情况下，拥有该组件的Actor的根组件会发生移动；不过，可能会选择另一个组件（see [SetUpdatedComponent](https://docs.unrealengine.com/en-US/API/Runtime/Engine/GameFramework/UMovementComponent/SetUpdatedComponent)). 如果更新后的组件正在进行物理模拟，只有初始参数（当初始速度非零时）将影响子弹（Projectile），且物理模拟将从该位置开始。

以下是使用ProjectileMovementComponent的蓝图示例（点击查看大图）。

![image-20220717184229220](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717184229220.png)

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717184357825.png" alt="image-20220717184357825" style="zoom:50%;" />

##### 10.6.3旋转移动组件

**RotatingMovementComponent** 允许某个组件以指定的速率执持续旋转。（可选）你可以偏移旋转时参照的枢轴点。注意，**在移动过程中，无法进行碰撞检测**。

使用旋转移动组件的案例包括：飞机螺旋桨、风车，甚至是围绕恒星旋转的行星。加上就会整个转

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717184844792.png" alt="image-20220717184844792" style="zoom:50%;" />

#### 10.7寻路组件

概述如何使用寻路组件来修改或扩展寻路功能。

寻路组件是一种可以在虚幻引擎中修改或扩展 **NavMesh** 寻路系统的组件。

##### 寻路修改器组件

**寻路修改器组件（Nav Modifier Component）** 本身没有任何功能。不过，假如你有一个Actor，并且用一个基本形状作为它的根节点，根组件的体积会根据寻路修改器组件的 **Area Class** 属性来修改 NavMesh 的生成效果**。每个 Actor 只能带有一个寻路修改器组件**，因为多个修改器组件是无效的。此外，这些（多余的）组件将出现在"组件"选项卡的层级结构之外，不能作为其他组件的父组件，也不能包含任何子组件。

这些区域类（Area Class）可定义一些基本设置，例如进入某个区域的 **成本（Cost）**，或者一些高级设置，例如蹲伏角色可移动的区域。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717192900195.png" alt="image-20220717192900195" style="zoom:50%;" />

成本是 NavMesh 系统中的一个重要概念。简单来说，在 NavMesh 中，<font color='brown'>从一个点移到另一个点的总成本等于路径经过的所有的区域成本总和（单个区域的大小在项目的偏好设置中定义）。</font>解算器会始终寻找成本最低的路径，因此，你可通过增加通过该区域的成本来让它避免某些区域（比如湿滑或崎岖不平的区域）。不过要注意，假如某个区域成本很高，但只要属于成本最低的路径，解算器仍然会通过它。

例如，通过红色区域的成本非常高，但是 Pawn 没有其他选择，只能从中通过：

![image-20220717185827519](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717185827519.png)

但是，如果你移除了墙壁：

![image-20220717185857498](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717185857498.png)

Pawn 将避免经过红色区域，即使它要绕更长的路线。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717193040721.png" alt="image-20220717193040721" style="zoom:50%;" />

#### 10.8Paper 2D 组件

Paper 2D组件用于为Actor添加（或修改） 2D **Sprite** 或 **Flipbook** 子对象。

##### 10.8.1Paper Sprite组件

**PaperSpriteComponent** 负责处理 **UPaperSprite** 实例的渲染和碰撞。 当你将Sprite资产从 **内容浏览器** 拖到 **蓝图** 中，该组件就会自动创建；或者，当你将某些Actor拖入关卡中时，该组件会包含在其中。

这类组件的一种用途就是充当搭建关卡的Sprite，比如，岩架或平台、梯子和斜坡等。将这些Sprite资产放置在关卡中将创建一个 **PaperSpriteActor**，该Actor使用的实例化PaperSpriteComponent（或唯一副本）基于选定Sprite资产生成。

有关在虚幻引擎4中创建 Sprite 的详细信息，请参见

[](animating-characters-and-objects/Paper2D/Sprites\Creation)

 文档。

##### 10.8.2Paper Flipbook组件

简单来说，**PaperFlipbookComponent** 是对 **SourceFlipbook** （是一系列依次播放的 Sprite，以创建 2D 动画）的封装。 在创建 **Paper2DCharacter** 时，将自动添加此组件类型，从而允许你创建可播放的 2D 动画人物。

PaperFlipbookComponent可以在3D空间中任意放置、附加到其他组件上或自身附带其他组件。 你可以为每个实例指定一个自定义颜色；该颜色会被传递给Flipbook材质作为顶点颜色参数。你也可以为它们指定一个自定义材质，用于替代SourceFlipbook中定义的默认材质。

使用脚本，你可通过调用 **SetFlipbook** 函数更改当前 SourceFlipbook 资产，但是，请注意仅当 **Mobility** 属性设为可移动（或在 Actor 构建过程中调用该函数）时，上述操作才有效。 你也可使用组件上的各种其他方式来控制播放速度、播放方向以及循环等。

有关设置和使用Flipbook组件的详细信息，请参见[Flipbook 组件](https://docs.unrealengine.com/5.0/zh-CN/flipbook-components-in-unreal-engine)文档。

#### 10.9物理组件(*)

介绍物理相关的组件，其中包括可破坏组件、推进器组件以及力组件。

这些物理组件用于影响那些在你的场景中以不同方式应用物理效果的任意对象。

##### 10.9.1可破坏组件

**可破坏组件（DestructibleComponent）** 用于存放 可破坏Actor 的物理数据。在添加该组件时，你必须指定要使用的 **可破坏网格体** 资产。你还可以覆盖并指定 **破裂效果（Fracture Effects）** 而非使用资产本身的破裂效果。

这类组件的用途包括模拟窗框中的玻璃。窗框是一个 **StaticMeshComponent**，而窗户则是能被玩家击碎的 **DestructibleComponent**。

##### 10.9.2物理约束组件

**物理约束组件（PhysicsConstraintComponent）** 是一种能连接两个刚性物体的接合点。你可以借助该组件的各类参数来创建不同类型的接合点。

借助 **PhysicsConstraintComponent** 和两个 **StaticMeshComponents**，你可以创建悬摆型对象，如秋千、重沙袋或标牌。它们可以对世界中的物理作用做出响应，让玩家与之互动（请参见 **[物理约束组件的用户指南](https://docs.unrealengine.com/5.0/zh-CN/physics-constraint-component-user-guide-in-unreal-engine)** 了解基于 **Blueprints** 的相关示例）。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717195206828.png" alt="image-20220717195206828" style="zoom:50%;" />

##### 10.9.3物理抓柄组件

**物理抓柄组件（PhysicsHandleComponent）** 用于"抓取"和移动物理对象，同时允许抓取对象继续使用物理效果。案例包括"重力枪"——你可以拾取和掉落物理对象（参见 [**物理内容示例**](https://docs.unrealengine.com/5.0/zh-CN/physics-in-unreal-engine) 了解详细信息）。

##### 10.9.4物理推进器组件

**物理推进器组件（PhysicsThrusterComponent）** 可以沿着 X 轴的负方向施加特定作用力。推力组件属于连续作用力，而且能通过脚本来自动激活、一般激活或取消激活。

推力组件的用途包括火箭（见下图）。它将持续施加作用力，将火箭向上推（因为推力部分位于火箭下方）。你可以用 **阻挡体积（Blocking Volumes）**，限制受推力影响的组件的动作。

##### 10.9.5径向力组件

**径向力组件（RadialForceComponent）** 用于发出径向力或脉冲来影响物理对象或可摧毁对象。与 **PhysicsThrusterComponent** 不同，这类组件只**施加"发射后不管"类型的作用力，而且并不持续。**

你可以使用这类组件来推动被摧毁对象（如爆炸物）的碎片。使用 **RadialForceComponent** 指定作用力和方向，当对象被摧毁时，你可以像下面的图示那样，沿着特定方向将碎片向外"推"（参见 [**Destructibles Content Examples**](https://docs.unrealengine.com/5.0/zh-CN/content-examples-sample-project-for-unreal-engine) 了解详细信息）。

#### 10.10渲染组件（*）

##### 10.10.1大气雾组件

**大气雾组件（AtmosphericFogComponents）** 用于创建雾效，如场景中的云或大气雾。该组件有多项设置，可影响雾在关卡中的生成方式。

下面通过示例介绍了 **衰减高度（Decay Height）** 设置（用于控制雾的密度衰减高度，例如，较低的值可以让雾变浓，较高的值会让雾变稀，产生更少的散布）。有关详细信息，请参见 **[大气雾用户指南](https://docs.unrealengine.com/5.0/zh-CN/environmental-light-with-fog-clouds-sky-and-atmosphere-in-unreal-engine)** 页面。

![image-20220717200701210](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717200701210.png)

##### 10.10.2指数高度雾组件

**指数高度雾组件（ExponentialHeightFogComponent）** 也用于创建雾效果，但其密度与雾的高度相关。

指数高度雾在地图中较低的地方产生密度较大的雾，在较高的地方产生密度较小的雾。雾会进行平滑过渡，因此随着你增加高度，不会看到明显的切换效果。指数级高度雾还提供了两种雾颜色，一种颜色用于面向主定向光源的半球体（如果不存在光源，则径直向上），另一种颜色用于相反方向的半球体。

![image-20220717200944488](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717200944488.png)

请参见**[指数级高度雾用户指南](https://docs.unrealengine.com/5.0/zh-CN/exponential-height-fog-in-unreal-engine)** 了解更多详情及可调整的设置。

##### 10.10.3广告牌组件

**广告牌组件（BillboardComponent）** 是一种2D纹理，始终朝向摄像机，其功能与 **箭头组件（ArrowComponent）** 类似，可用于某种放置方法且能够轻松选取。例如，在下面的雾气薄片中，唯一添加的组件就是 **BillboardComponent**（实际的雾气效果是一个通过脚本动态创建的材质）。

![billboard1.png](https://docs.unrealengine.com/5.0/Images/understanding-the-basics/components/rendering-components/billboard1.jpg)

在场景中，你可以通过选择 **BillboardComponent** 图标（一个可以指定的贴图）来控制雾气薄片。

![image-20220717201322454](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717201322454.png)

有关如何创建此雾效的示例，请参见 **[雾气薄片和光束](https://docs.unrealengine.com/5.0/zh-CN/samples-and-tutorials-for-unreal-engine)** 文档。

##### 10.10.4材质广告牌组件

**材质广告牌组件（MaterialBillboardComponent）** 是一种2D材质，始终朝向摄像机。这类组件可用于表现 2D 草丛或树叶。我们并未使用静态网格体来表现草丛甚至单片草叶，而是用一个带有材质的 **材质广告牌组件** ；这样就无需使用3D模型了，因为广告牌会始终朝向玩家，从而模拟草的三维效果。

##### 10.10.5线缆组件

**线缆组件（CableComponent）** 可以将两个组件绑定在一起，同时在它们之间渲染一条线缆。在这条线缆上，你可以指定一个材质并定义参数来影响连线的显示方式。想要了解更多有关线缆组件的信息，请点击下方的链接。

- [**线缆组件用户指南**](https://docs.unrealengine.com/5.0/zh-CN/cable-components-in-unreal-engine)

##### 10.10.6自定义网格体组件

**自定义网格体组件（CustomMeshComponent）** 能让你指定自定义三角形网格几何体。

##### 10.10.7姿态网格体组件

**姿态网格体组件（PoseableMeshComponent）** 允许你通过 **蓝图** 来控制骨骼变换。

##### 10.10.8贴花组件（*）

**贴花组件（DecalComponent）** 是一种可被渲染到网格体表面上的材质（类似于模型的"保险杠标贴"）。贴花组件可用于任何目的，例如墙上的弹孔，车辆的胎印，地上的血迹等（参见下文示例）。（一个框，在里面的网格体的贴图会被渲染成指定的材质）

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717202847825.png" alt="image-20220717202847825" style="zoom: 67%;" />

你可以参考以下页面，了解贴花组件详情。

- 参见 [贴花Actor用户指南](https://docs.unrealengine.com/5.0/zh-CN/decal-actors-in-unreal-engine)。
- 参见 [1.1 - 基本贴花](https://docs.unrealengine.com/5.0/zh-CN/content-examples-sample-project-for-unreal-engine)。
- 参见 [贴花内容范例](https://docs.unrealengine.com/5.0/zh-CN/content-examples-sample-project-for-unreal-engine)。

##### 10.10.9实例化静态网格体组件

**实例化静态网格体组件（InstancedStaticMeshComponent）** 可以高效渲染同一静态网格体的**多个实例**。 这类组件尤其适用于程序化创建的场景或房间，因为你不必在场景中放置成百上千个 **静态网格体Actor**，而是只需放置一个 **实例化静态网格体**，就能添加该静态网格体（如地板或墙）的多个实例，性能开销也大大降低。

请在我们的 Wiki 页面上参阅 [**Procedural Room Generation**](https://wiki.unrealengine.com/Videos/Player?series=PLZlv_N0_O1ga0aV9jVqJgog0VWz1cLL5f&video=mI7eYXMJ5eI) 教程，了解 **实例化静态网格体** 组件以及程序化生成随机房间的示例。

##### 10.10.10粒子系统组件

**粒子系统组件（ParticleSystemComponent）** 可以让你添加一个粒子发射器作为其他对象的子对象。**粒子系统组件** 用多种作用，例如，添加爆炸效果或燃烧效果。添加这类组件后，你可以借助脚本访问和设置粒子的效果参数（例如打开或关闭效果）。

请参见 [**Cascade粒子系统**](https://docs.unrealengine.com/5.0/zh-CN/creating-visual-effects-in-niagara-for-unreal-engine)了解详细信息。

##### 10.10.11后期处理组件

**后期处理组件（PostProcessComponets）** 可以为 **蓝图** 启用后处理控制。它通过 `UShapeComponent` 这个父类来提供体积数据（如可用）。这类组件可以变换场景的色调（前提是场景应用了后处理设置）。例如，假设你定义了一个默认的后处理设置并将其用于游戏，那么在玩家受到伤害（或丧命）时，你可以通过脚本将 **Scene Color Tint** 的设置改为黑色/白色色调。

有关详细信息，请参见 [**后期处理内容示例**](https://docs.unrealengine.com/5.0/zh-CN/content-examples-sample-project-for-unreal-engine)或 [**后期处理效果**](https://docs.unrealengine.com/5.0/zh-CN/post-process-effects-in-unreal-engine)文档。

##### 10.10.12场景捕获2D组件

**场景捕获2D组件（SceneCapture2DComponent）** 用于从单一平面捕获场景"快照"，并将其发送给渲染目标。它的参数包括：控制 **视场**，指定 **渲染目标** 纹理等等。它的用途包括创建镜子、模拟监视器画面（参见 [**监控摄像机切换按钮**](https://docs.unrealengine.com/5.0/zh-CN/samples-and-tutorials-for-unreal-engine)。）

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717204855939.png" alt="image-20220717204855939" style="zoom:50%;" />

在上面的示例中，我们创建了一个带有 **场景捕获2D组件（SceneCapture2DComponent）** 的 **蓝图** 并指定了一张 **渲染目标** 纹理，它随后被用作 **材质** 指定给场景中的几何体。有关详细信息，请参见 [**场景捕获2D的内容示例**](https://docs.unrealengine.com/5.0/zh-CN/content-examples-sample-project-for-unreal-engine)。

##### 10.10.13场景捕获立方体组件

**场景捕获立方体组件（SceneCaptureCubeComponent）** 可以用 6 个平面捕获场景的"快照"，并将其发送给渲染目标。

在多数情况下，**场景捕获2D组件（SceneCapture2DComponent）** 就能满足大部分场景捕获需求，但在需要对场景进行 3D 捕获时，你可以使用这个组件。但要注意，它会产生很大的性能消耗，只应在绝对必要时使用。请参见 [**反射**](https://docs.unrealengine.com/5.0/zh-CN/samples-and-tutorials-for-unreal-engine) 了解场景中反射的不同创建方法。

##### 10.10.14样条线网格体组件

**样条线网格体组件（SplineMeshComponents）** 可用于拉伸和弯曲静态网格体。使用 **样条线网格体组件** 时，你必须提供位置向量，并提供样条曲线起点和终点的切线。在下面的示例中，蓝图包含一个样条线网格体组件，以及一根受到组件影响的 **静态网格体** 管道。

当你指定方位向量以及组件的切线时，你可以使用脚本将它们设置为变量并改为公用（public），以便其可以在编辑器视区中编辑（如下面所示）。

![image-20220717205633881](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717205633881.png)

在上述示例中，我们可以单独移动 **Start Transform** 和 **End Transform** 而无需移动整个 Actor，从而随意拉伸或旋转它。该示例及其相关设置可以在 [**蓝图样条内容示例**](https://docs.unrealengine.com/5.0/zh-CN/content-examples-sample-project-for-unreal-engine) 内容示例图中找到。

##### 10.10.15文本渲染组件

**文本渲染组件（TextRenderComponent）** 可以用指定的字体渲染场景中的文本。其中包含与常用字体有关的属性，如缩放比例（Scale）、对齐（Alignment）、颜色（Color） 等。你可以使用该组件提示玩家场景中存在一个可交互对象。

例如，假设你的场景中有一把椅子，玩家在靠近时按一个按键就能坐下。你可以添加一个包含提示文本的 **文本渲染组件** 来执行就坐命令（此时关闭可见性），同时添加一个 **盒体组件（BoxComponent）** 用作触发器，用于在玩家进入时将文本的可见性设为 true（如下所示）。

![image-20220717205844080](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220717205844080.png)

在玩家进入触发器范围时，游戏将显示 **文本渲染组件** 文本以提示玩家如何坐下。

##### 10.10.16向量场组件

**向量场组件（VectorFieldComponent）** 用于引用

[**Vector Field**](creating-visual-effects/VectorFields)

 ——一种带有速度向量网格的 3D 容器，可用于确定 GPU Sprite的速度或加速度。 向量场可用于场景中的小规模效果（如强气流粒子效果）和大规模效果（如暴雪）。另请参见 [**局部向量场（Local Vector Fields）**](https://docs.unrealengine.com/5.0/zh-CN/content-examples-sample-project-for-unreal-engine) 和 [**全局向量场（Global Vector Fields）**](https://docs.unrealengine.com/5.0/zh-CN/content-examples-sample-project-for-unreal-engine) 了解详细信息。

#### 10.10.17形状组件

形状组件（Shape Component）可用于创建碰撞体积、触发器、方向提示工具，以及路径点。

##### 10.10.17.1·箭头组件

**箭头组件（ArrowComponent）** 是一种由直线和箭头组成的形状，表示对象的朝向。在下面这个门的示例中，箭头指示了门在场景中的朝向（门可能只能朝一个方向开启，也就是箭头的朝向）。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718100506778.png" alt="image-20220718100506778" style="zoom:50%;" />

箭头不会显示在实际游戏中（除非取消选中 *Hidden in Game* 选项），而颜色和大小可以随意调整。该组件没有任何碰撞设置，可以被用作脚本"标记"（例如，在 **人物蓝图** 中，将一个 **箭头组件** 添加在人物肩部，然后在玩家按下按钮时，将 **摄像机组件** 移动到 **箭头组件** 所在的位置）。

##### 10.10.17.2盒体组件

**盒体组件（BoxComponent）** 是一个盒体，通常被用于简单碰撞（也可用作下面示例中的触发器）。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718100618520.png" alt="image-20220718100618520" style="zoom:50%;" />

**盒体组件** 包住了火焰特效，并且其碰撞设置被指定为 **生成重叠事件（Generate Overlap Events）**。当其他对象与盒体重叠时，会触发一个事件，对重叠的 Actor 施加伤害。你也可以将 [碰撞响应（Collision Response）](https://docs.unrealengine.com/5.0/zh-CN/collision-in-unreal-engine) 设置为 **全部阻挡（BlockAll）** 来避免所有 Actor 进入这个盒体（如果你希望防止其他东西进入火焰范围内）。

##### 10.10.17.3胶囊形组件

**胶囊体组件（CapsuleComponent）** 是一种胶囊形状的简易碰撞体（如下所示），也可充当触发器。

![image-20220718100837795](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718100837795.png)

所有新建的 **角色蓝图** 都会自动包含 **胶囊体组件** （如上图所示）。胶囊体的碰撞设置能避免**角色与场景对象重叠**。**胶囊体组件** 还能 **生成重叠事件（Generate Overlap Events）** 或 **Generate Hit Events**，以便你提供脚本来指定何时发生这些事件。

##### 10.10.17.4球体组件

**球体组件（SphereComponents）** 是一种可用于碰撞的球体组件（放置在发射物周围，如下图）。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718102514806.png" alt="image-20220718102514806" style="zoom:50%;" />

与盒体、胶囊体一样，你可以设置 [**碰撞响应（Collision Responses）**](https://docs.unrealengine.com/5.0/zh-CN/collision-in-unreal-engine) 来生成所需的碰撞功能类型。

##### 10.10.17.5样条线组件

**样条线组件（SplineComponent）** 可用于生成街道或复杂路径（作为其他组件的运动路线）。

**样条线（Spline Component）** 允许用户在关卡内绘制样条线。它适合为可移动Actor（例如火车、背景人物、摄像机等）指定路径。在关卡中添加样条线Actor后，组件位置会显示一个点，该点连着旁边的第二个点。你可以右键单击样条线，在点击位置添加新节点。也可以单击某个点，删除（除非只有一个点）或复制它，或者更改两点之间的曲线轨迹。复制时，会在当前位置新建一个节点，但在线中被认为是"下一个"点。你可以使用移动和旋转工具，在关卡中操纵样条点。在组件的 **细节（Details）** 面板中，有一个名为 `持续（Duration）` 的字段，可以设置样条查询范围的值。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718103126661.png" alt="image-20220718103126661" style="zoom:67%;" />

在编辑器视口内，你可以 **右键点击** **样条线组件** 所在的 Actor 来编辑样条线曲线。这样会打开一个窗口，允许你为样条线添加点，或是定义要使用的样条线的点的类型。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718103615516.png" alt="image-20220718103615516" style="zoom:50%;" />

样条线主要有三种查询方法：

- 按节点索引查询。查询样条上节点的信息，包括点的位置或旋转度。
- 按距离查询。查询沿着样条线移动一段距离后，样条的旋转前向向量。
- 按 "时间" 查询。使用这种方法时，样条的长度被替换为用户输入的`时长（Duration）`值。可以查询给定"时间"下，样条的距离、位置和旋转信息。第三种方法尤其适用于固定物体（例如沿着样条轨迹移动的摄像机），因为设计师和程序员可以直观地根据摄像机沿样条移动的距离来计算摄像机的位置和旋转信息。 通过将"时长"设置为1.0，然后用一个"alpha"值插值，你可以很容易地进行样条插值或混合。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718111713368.png" alt="image-20220718111713368" style="zoom: 80%;" />

使用样条线创建的样条永远不会分叉成多个路径，也不会包含成环形回路或孤立的点。样条始终为单一路径，每个样条点仅通过一次。

#### 10.10.18骨骼网格体组件

骨骼网格体组件用于处理所有带有骨架的复杂动画数据。

##### 10.10.18.1骨骼网格体组件

**骨骼网格体组件（SkeletalMeshComponents）** 用于创建 `USkeletalMesh` 实例。骨骼网格体（指外观）的内部包含一具骨架（用于连接骨骼）；骨架负责移动骨骼网格体的各个顶点，以便匹配当前播放的动画。 因此，骨骼网格体组件适合用来表现人物、生物、复杂机械或任何据具备复杂运动的对象。请参见 **[骨架网格体](https://docs.unrealengine.com/5.0/zh-CN/skeletal-meshes)** 和 **[Skeletal Mesh Actors](https://docs.unrealengine.com/5.0/zh-CN/skeletal-mesh-actors-in-unreal-engine)** 了解骨骼网格体的详情。

除了可以指定骨骼网格体资产，你还可以定义网格体的动画模式（是使用 **[动画蓝图](https://docs.unrealengine.com/5.0/zh-CN/animation-blueprints-in-unreal-engine)** 还是 **[动画资产](https://docs.unrealengine.com/5.0/zh-CN/animation-sequences-in-unreal-engine)**）。

#### 10.10.19静态网格体组件

静态网格体组件可用作Actor的子对象，用于表示Actor的几何形状。

**静态网格体组件（StaticMeshComponent）** 可创建 `UStaticMesh` 的实例。**静态网格体（Static Mesh）** 是一个由多个静态多边形构成的几何体，是虚幻引擎 4 的基本场景构建单位。除用于构建场景，静态网格体还能用于创建运动对象（如门或电梯）、刚体物理对象、植物和地形装饰物、程序化创建的建筑、游戏目标和许多其他的视觉元素。

下面的示例使用了一个 静态网格体组件 用于表示灯具，并用聚光源组件和 点光源组件 创建警报效果。两种光源会不断循环播放。

#### 10.10.20工具组件

##### 10.10.20.1应用程序生命周期组件

**应用程序生命周期组件（ApplicationLifecycleComponent）** 负责接收来自OS（操作系统）的应用状态（开始、停止、结束等）通知。

##### 10.10.20.2平台事件组件

**平台事件组件（PlatformEventComponent）** 能根据平台的操作系统信息和事件来轮询和接收事件。目前，该组件支持可翻转式笔记本电脑，提供关于平台是否为可翻转式笔记本电脑及其状态（笔记本电脑模式或平板电脑模式）的信息。为了无需不断轮询这些函数就能知道平台状态何时改变，你可以绑定委托。当状态改变时，委托会通知代码（或蓝图）。

##### 10.10.20.3子Actor组件

**子Actor组件（ChildActorComponent）** 是一个能在注册时生成Actor并在注销时销毁Actor的组件。

子Actor组件通过提供C++类或蓝图资产，来指定要生成的Actor。选择类或资产后，将显示 **子Actor模板（Child Actor Template）** 字段。你可以展开此字段，编辑子Actor或其组件的所有公开的可编辑字段，包括蓝图中声明的公共变量。

##### 10.10.20.4场景组件

**场景组件（SceneComponent）** 包含变换属性，并支持绑定子对象，但它没有渲染或碰撞功能。你可以在层级中把它用作**"虚拟"组件**，用来偏移其他组件，或者用来统一**管理**它的所有子组件。例如，整体移动或旋转它的子组件，显示或禁止显示它们，或者将它们全部断开。注意，不要过度使用场景组件，因为场景组件越多，它们所属的Actor的内存占用和生成时间就越大。

##### 10.10.20.5箭头组件

**箭头组件（ArrowComponent）** 是一种**场景组件**，由一条线以及表示朝向的箭头构成。箭头组件适用于**调试**，比如表示插槽的位置或朝向。它可用于调试游戏中服饰、手持物或绑定物的位置，或调试枪口的位置。默认情况下，这些组件仅显示在编辑器中。要使其在游戏中可见，请取消选中 `在游戏中隐藏（Hidden in Game）` 复选框（显示在箭头组件的 **细节(Details)** 面板中）。

#### 10.10.20.6控件组件

介绍什么是控件组件——一种可用于在关卡中添加可交互控件的组件。

控件组件（Widget Component）允许你在游戏场景中用[虚幻示意图形（UMG）](https://docs.unrealengine.com/5.0/zh-CN/specialized-blueprint-visual-scripting-node-groups-in-unreal-engine)来创建3D UI元素。

**控件（Widget）** 组件是控件蓝图的3D实例，可以在游戏场景中与之交互。

本例使用控件蓝图来显示游戏中的交互式菜单。

你可以通过更改 **绘制大小（Draw Size）** 或使用 **按要求大小绘制（Draw at Desired Size）** 来更改控件组件的大小。

将包含控件组件的Actor放置到关卡中，控件类蓝图就会显示在游戏中。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718112558425.png" alt="image-20220718112558425" style="zoom:50%;" />

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718112750405.png" alt="image-20220718112750405" style="zoom:67%;" />

| **选项**                                   | **说明**                                                     |
| ------------------------------------------ | ------------------------------------------------------------ |
| **空间（Space）**                          | 用于渲染控件（世界场景（World）或屏幕（Screen））的坐标空间。使用世界场景（World）时，控件以网格体的形式在世界场景中进行渲染，并且可被遮挡，而屏幕（Screen）将在完全处于世界场景之外的屏幕上渲染控件，并且控件永远不会被遮挡。 |
| **控件类（Widget Class）**                 | 用于创建和显示用户控件实例的用户控件类。                     |
| **绘制大小（Draw Size）**                  | 显示的四边形的大小。                                         |
| **手动重绘（Manually Redraw）**            | 控件是否应等待被告知重绘方可实际绘制。                       |
| **重绘时间（Redraw Time）**                | 绘制时间间隔，如果为0，则重绘每一帧。如果为1，我们将每秒重绘。这也可以与手动重绘（Manually Redraw）配合使用。你可以说，手动重绘，但只能以这个最大速率重绘。 |
| **窗口可聚焦（Window Focusable）**         | 创建用于托管控件的虚拟窗口是否可聚焦。此窗口是否应得到用户的关注。 |
| **按要求大小绘制（Draw at Desired Size）** | 使渲染目标自动匹配控件类指定的所需大小。如果每一帧都绘制，那么成本会很高昂。 |
| **枢轴（Pivot）**                          | 控件相对于该位置放置的对齐点/枢轴点。                        |

##  11.运行和模拟

可以随时在虚幻编辑器中预览游戏，无需将其构建为独立的应用程序。这样，你就能快速调整游戏玩法和资产，并了解相应调整带来的后果。

在 **虚幻引擎** 中预览游戏的两种方法：

- **在编辑器中运行（Play In Editor）** (PIE)，你可以通过 **主工具栏（Main Toolbar）** 上的 **运行（Play）** 按钮访问它。
- **在编辑器中模拟（Simulate In Editor）** (SIE)，你可以从 **运行（Play）** 下拉菜单或使用Windows键盘快捷键上的 **Alt + S**（若是macOS，则使用 **Option + S** 快捷键）访问它。

在编辑器中，运行和模拟之间的主要区别在于 **运行** 将始终在玩家出生点（Player Start）位置开始游戏，并让你控制玩家角色。**模拟**<font color='brown'>不会移动摄像机，也不会产生玩家角色。</font>

你可以根据需要在在编辑器中运行（Play In Editor）和在编辑器中模拟（Simulate In Editor）会话之间切换。

### 11.1PIE(运行)ALT+P



![image-20220718132227348](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718132227348.png)

你将在关卡编辑器视口的左上角看到文本"按Shift+F1使用鼠标光标（Shift+F1 for Mouse Cursor）"。如果你要将鼠标控制交还给编辑器本身，请按 **Shift + F1** （Windows）或 **Shift + fn + F1** （macOS）。

#### 11.1.1在运行时更改Actor

1.游戏仍在PIE模式下运行时，按 **Shift + F1** 可以从关卡视口解锁鼠标光标，然后点击 **暂停（Pause）** 暂停游戏。

2.游戏仍然暂停时，**点击** **主工具栏（Main Toolbar）** 上的 **弹出（Eject）** 按钮。

3.对Actor进行更改

4.在 **主工具栏（Main Toolbar）** 中，点击 **占用（Possess）** 按钮（1），然后点击 **恢复（Resume）** 按钮（2）。<font color='red'>默认情况下，你使用此方法对关卡中的Actor所做的更改 **不** 保存。</font>

5.游戏仍在关卡视口中运行，**右键点击**Actor。然后，从弹出菜单中，选择 **保留模拟更改（Keep Simulation Changes）** 。或者，**左键点击** 文本Actor将其选中，然后按键盘上的 **K** 。

#### 11.1.2模拟及其他运行模式

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718132732355.png" alt="image-20220718132732355" style="zoom: 50%;" />

### 11.2PIE控制台

**PIE 控制台（PIE Console）** 是游戏内控制台，你可以在其中输入命令，以便显示性能数据，启用和禁用虚幻引擎功能等等。

要打开PIE控制台，请在PIE模式下玩游戏时按 **波浪号** （~）键。

再次按 **波浪号** 键展开控制台，第三次按 **波浪号** 会关闭它。

PIE控制台的行为与虚幻编辑器的主控制台相同。当你开始输入时，它会自动尝试完成你尝试输入的控制台命令。

分析项目性能是PIE控制台的不错用例。要了解更多信息，请参阅[统计命令](https://docs.unrealengine.com/5.0/zh-CN/stat-commands-in-unreal-engine)页面。

![image-20220718140556545](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718140556545.png)

#### 11.2.1运行虚幻引擎

本文所述方法只针对使用 **开发（Development）** 模式构建的项目。假如你采用了其他构建模式，请视情况将指令替换成 `UEEditor-_.exe` 或 `UE-_.exe`。有关二进制版本的命名规范，请参阅[编译虚幻引擎](https://docs.unrealengine.com/5.0/zh-CN/building-unreal-engine-from-source)。

##### 11.2.1.1运行编辑器

如需运行编辑器，需要将项目名称作为参数传递给可执行文件。

你可以添加 `-debug` 参数，让可执行文件强制加载项目模块的调试版本，该版本包含所有调试符号。由于主可执行文件固定使用 **开发** 配置进行编译，因此即便你在Visual Studio中将配置设为 **调试**，你仍需传递debug参数。当然，你必须首先用调试配置编译模块，以便可执行文件进行加载。

##### 11.2.1.2通过命令行运行编辑器

1. 打开命令行窗口，导航至 `[LauncherInstall][VersionNumber]\Engine\Binaries\Win64` 目录。

2. 运行 `UEEditor.exe`，并传递项目路径：

   ```
   UEEditor.exe "[ProjectPath][ProjectName].uproject"
   ```

##### 11.2.1.3通过可执行文件运行编辑器

1. 导航至 `[LauncherInstall][VersionNumber]\Engine\Binaries\Win64` 目录。
2. 右键单击 `UEEditor.exe` 可执行文件并选择 **创建快捷方式**。
3. 将快捷方式重命名为例如 **MyProject - Editor.exe** 这类命名，以表明该快捷方式运行MyProject游戏编辑器。 
4. 右键单击新建快捷方式并选择 **属性**。
5. 在 **目标（Target）** 属性末尾添加作为参数运行的游戏名称：

```
[LauncherInstall][VersionNumber]\Engine\Binaries\Win64\UEEditor.exe "[ProjectPath][ProjectName].uproject"   
```

​    6.按下 **OK** 保存更改。

​    7.**双击** 快捷方式以启动编辑器。

##### 11.2.1.4不使用参数(Stand-alone)运行编辑器

假如你未设置编辑器在启动时打开最近的项目，以无参模式运行编辑器可执行文件会直接打开项目浏览器。在此可 [新建项目](https://docs.unrealengine.com/5.0/zh-CN/creating-a-new-project-in-unreal-engine)、[打开现有项目](https://docs.unrealengine.com/5.0/zh-CN/creating-a-new-project-in-unreal-engine) ，或者打开[内容范例和和示例游戏](https://docs.unrealengine.com/5.0/zh-CN/samples-and-tutorials-for-unreal-engine)。

##### 11.2.1.5运行未烘焙游戏

在虚幻编辑器中加载项目后，便可使用 **运行方式（Play In）** 菜单在未烘焙游戏（Uncooked Game）模式下[测试gameplay](https://docs.unrealengine.com/5.0/zh-CN/in-editor-testing-play-and-simulate-in-unreal-engine#模式)。要在未烘焙游戏自带窗口中运行游戏，请使用关卡编辑器工具栏中的[在下拉菜单中运行](https://docs.unrealengine.com/5.0/zh-CN/in-editor-testing-play-and-simulate-in-unreal-engine#工具栏)以选择[新窗口位于（New Window At）> 玩家默认起始模式（Default Player Start Mode）](https://docs.unrealengine.com/5.0/zh-CN/in-editor-testing-play-and-simulate-in-unreal-engine#模式)。

##### 11.2.1.6使用命令行运行未烘焙游戏

使用命令行运行时，必须将要运行的项目名称及 `-game` 作为参数传递。

1. 使用命令行，导航至 `[LauncherInstall][VersionNumber]\Engine\Binaries\Win64` 目录。
2. 运行 **UEEditor.exe**，并向其传递要运行的项目路径及 `-game` 参数。

##### 11.2.1.7通过可执行文件运行未烘焙游戏

通过可执行文件运行时，你必须在快捷方式的 **目标（Target）** 属性中，指定要运行的项目路径，并传递 `-game` 作为参数。

1. 导航至 `[LauncherInstall][VersionNumber]\Engine\Binaries\Win64` 目录。

2. **右键单击** **UEEditor.exe** 可执行文件并选择 **创建快捷方式**。

3. 重命名快捷方式以表明反映其将运行的游戏，即 **MyProject.exe**。

4. **右键单击** 新建快捷方式，选择 **属性** 以显示快捷方式的属性。

5. 在 **目标** 属性末尾添加要作为参数运行的项目完整路径，并指定要作为游戏运行的 `-game` 参数：

   ```
   [LauncherInstall][VersionNumber]\Engine\Binaries\Win64\UEEditor.exe "[ProjectPath][ProjectName].uproject" -game
   ```

6. 按下 **OK** 保存更改。

7. **双击** 快捷方式运行游戏。

##### 11.2.1.8运行烘焙游戏

欲了解打包和运行烘焙游戏版本的相关方法，参见[打包项目](https://docs.unrealengine.com/5.0/zh-CN/packaging-unreal-engine-projects)

##### 11.2.1.9实用游戏命令

运行游戏时，有众多 **主机命令** 可用于游戏控制台中 。按 **~（波浪符）** 或 **Tab** 键可打开控制台。下方列出了部分实用命令。

- `EXIT/QUIT`
- `DISCONNECT`
- `OPEN [MapURL]`
- `TRAVEL [MapURL]`
- `VIEWMODE [Mode]`

##### 11.2.1.10加载地图

运行引擎或编辑器时，可以指定要加载的地图，或者指定加载新地图。这样你就能直接打开要测试的地图，避免了许多麻烦。

##### 11.2.1.11启动时加载地图

引擎每次运行时都会尝试打开一张默认地图。你可以在 `DefaultEngine.ini` 配置文件中指定该地图，此文件位于游戏项目的"Config"文件夹中。默认情况下，你可以通过 ini. 文件 `[URL]` 部分中的 **地图** 属性来设置要运行的地图。例如，Vehicle Game的 `DefaultEngine.ini` 文件进行下列设置：

```
GameDefaultMap=/Game/Maps/MyMap
```

此设置将确保 `MyMap.umap`（位于 `(UE4Directory)/(YourProjectName)/Content/Maps`）在启动时自动加载，除非其被覆盖。通常，你可以将指定要加载的地图或主菜单地图作为默认加载地图。

要覆盖默认地图，可将地图命名（无需文件扩展名）作为命令行参数传入。在上文中提到过，你必须在命令行中指定项目名称。你还可以指定地图名称，以强制引擎加载其他地图。例如，以下命令行能让引擎加载 `ExampleMap` 地图：

```
UEEditor.exe "[ProjectPath][ProjectName].uproject" ExampleMap -game
```

上述方法对于编辑器运行也适用。编辑器打开时，你可以指定要加载的地图名称，以便加载此地图，而非默认或空白地图。如需运行编辑器并加载 `ExampleMap` 地图，可使用以下命令行：

```
UE4Editor.exe "[ProjectPath][ProjectName].uproject" ExampleMap -game
```

地图名称也可以是完整的地图URL，其中可包含GameMode等额外设置。设置能以键值对的形式添加在地图名称后面，以"?"分割。例如：

``` 
DM-Deck?Game=CaptureTheFlag
```

##### 11.2.1.12加载新地图

若要在游戏进程中加载新地图，以便开发测试或在游戏期间切换地图，可在控制台输入 `OPEN` 或 `TRAVEL` 命令，后跟要加载的**地图命名**（无需文件扩展名）。

关于 `OPEN` 命令和 `TRAVEL` 命令的区别，参见上文[实用游戏命令](https://docs.unrealengine.com/5.0/zh-CN/running-unreal-engine)章节。

## 12.打包项目（*）

你必须先对虚幻项目进行正确打包，之后才能将其发布给用户。打包能确保所有代码和内容都为最新且使用正确格式，以便在预期的目标平台上运行。

打包过程会涉及以下几个步骤。首先，所有项目特定的源代码会被编译。代码编译完成后，所有所需的内容都会被转化（即所谓的"烘培"）成目标平台可以使用的格式。然后，编译后的代码和经过烘焙的内容将被打包成一组可发布的文件，例如安装程序。

在菜单栏的 **文件（File）** 菜单中，有一个名为 **打包项目（Package Project）** 的选项。该选项包含一个子菜单，其中列出了所有引擎能支持得平台，你可以为这些平台打包项目。

假如选择为Android设备打包，你就会看到多个选项。请参阅[Android参考页面](https://docs.unrealengine.com/5.0/zh-CN/android-development-basics-for-unreal-engine)页面获取更多信息。

在打包前，你还可以设置一些 **高级（Advanced）** 选项。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718144739874.png" alt="image-20220718144739874" style="zoom:50%;" />

### 12.1设置游戏默认地图

你懂的

### 12.2创建打包文件

日志

假如这些窗口没有显示，你只需点击 **窗口（Window）** > **开发者工具（Developer Tool）** > **输出日志（Output Log）** / **消息日志（Message Log）** 来启用它们。

### 12.3发布

假如你想将iOS或Android游戏发布到App Store或Google Play Store上，你就要用发布（Distribution）模式创建打包文件。为此，请点击 **打包（Packaging）** 菜单中的 **打包设置（Packaging Settings）** 选项，然后勾选 **发布（Distribution）** 复选框。

假如是iOS，你需要在Apple的开发人员网站上创建发布证书（Distribution Certificate）和移动设备配置（MobileProvision）。请以安装开发证书的方式安装发布证书，并以"Distro_"为前缀命名发布配置，紧接着命名另一个配置（因此你将同时拥有 `Distro_MyProject.mobileprovision` 和 `MyProject.mobileprovision`）。

假如是Android，你需要创建一个密钥来签署 `.apk` 文件，并使用名为 `SigningConfig.xml` 的文件向编译工具传递一些信息。该文件位于引擎的安装目录（`Engine/Build/Android/Java/`）中。假如你编辑了该文件，它就会影响你的所有项目。然而，你可以将该文件复制到项目的 `Build/Android/` 目录（无 `Java/` 子目录），这样它就只会影响该项目。你可以在该文件的内部找到关于如何生成密钥和填写文件的说明。

### 12.4高级设置

在主菜单栏中点击 **文件（File）> 打包项目（Package Project）> 打包设置...（Packaging Settings...）**，或者点击 **编辑（Edit）> 项目设置（Project Settings）> 打包（Packaging）**，编辑器会显示一些和打包有关的高级配置选项。

![image-20220718145352443](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718145352443.png)

包括

| **选项**                                                     | **说明**                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **编译配置（Build Configuration）**                          | 编译代码类项目时使用的编译配置。若要调试代码项目，请选择"调试游戏（DebugGame）"。对于大多数具有最低限度调试支持但性能更佳的其他开发，请选择开发（Development）。对于不含调试信息且不含调试导向性功能（例如绘制调试形状或打印屏幕上的调试消息）的最终发布版本，请选择发布（Shipping）。注意，纯蓝图项目没有用于创建DebugGame编译的选项。 |
| **暂存目录（Staging Directory）**                            | 包含游戏打包后的版本的目录。当你在目标目录选择中选择另一个目录时，它将自动更新。 |
| **完整重编译（Full Rebuild）**                               | 是否应编译所有代码。如果禁用，则只编译修改过的代码。这可以加快打包过程。对于发布版本，你应该始终执行完整重编译，以确保没有任何内容丢失或过时。此选项默认为启用。 |
| **使用Pak文件（Use Pak File）**                              | 是否将项目的资产打包为单个文件或单个包。如果启用，所有资产将被放入单个.pak文件，而非复制所有单个文件（默认为启用）。如果你的项目使用大量资产文件，则使用Pak文件可以使发布变得更简单，因为它减少了需要传输的文件数量。此选项默认为禁用。 |
| **生成文件块（Generate Chunks）**                            | 是否生成可用于流送安装的.pak文件块。                         |
| **编译HTTP文件块安装数据（Build Http Chunk Install Data）**  | 是否为HTTP块安装文件生成数据。此配置允许在运行时安装将在Web服务器上托管的该数据。 |
| **Http数据块安装数据目录（Http Chunk Install Data Directory）** | 它表示数据在编译后的目标安装目录。                           |
| **Http数据块安装数据版本（Http Chunk Install Data Version）** | 它表示HTTP数据块安装数据的版本名称。                         |
| **包含先决条件安装文件（Include Prerequisites Installer）**  | 它指定打包游戏是否包含先决条件的安装文件，例如可重新发布的OS组件。 |

### 12.5签名和加密

随着虚幻引擎4.22的发布，我们为桌面平台（Windows、Mac和Linux）集成了行业标准[OpenSSL](https://www.openssl.org/)库。

当以发货产品的形式分发时，`.Pak` 文件可以签名或加密，通常是为了防止数据提取或篡改。要激活、停用或调整项目上的密码设置，请转到 **项目设置（Project Settings）** 菜单，找到 **加密（Crypto）** 部分。

![image-20220718164319053](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718164319053.png)

| **加密Pak INI文件（Encrypt Pak INI Files** | 对项目的 `.pak` 文件中所有 `.ini` 文件进行加密。这将以极小的运行时成本防止轻松挖掘或篡改产品的配置数据。 |
| ------------------------------------------ | ------------------------------------------------------------ |
| **加密Pak索引（Encrypt Pak Index）**       | 加密 `.pak` 文件索引，以极小的运行时成本防止UnrealPak打开、查看和解压缩你产品的 `.pak` 文件。 |
| **加密UAsset文件（Encrypt UAsset Files）** | 加密 `.pak` 文件中的 `.uasset` 文件。这些文件包含关于内部资产的标头信息，但不包含实际资产数据本身。加密这些数据为你的数据提供了额外的安全性，但是增加了少量运行时成本和数据熵，这会增加补丁的大小。 |
| **加密资产（Encrypt Assets）**             | 完全加密 `.pak` 中的所有资产。                               |
| **启用Pak签名（Enable Pak Signing）**      | 激活或禁用 .pak 文件签名。                                   |

<font size=4>加密资产</font>

此设置会对运行时文件I/O性能产生可度量的影响，并增加最终打包数据中的熵，从而降低分发补丁系统的效率。

此外，可以设置或清除用于签名或加密的密钥。

### 12.6内容烘焙

作为一名开发人员，在迭代新的或修改过的游戏内容时，你可能并不总是希望经历将所有内容都打包到Staging Directory并从此处运行的漫长过程。因此，只需点击 **文件（File）> 烘焙内容（Cook Content）> [平台名称（PlatformName）]**，即可为特定目标平台烘焙内容，而无需打包。

注意，该功能将更新项目本地开发人员工作区中的内容，并且不会将任何资产复制到暂存目录。你可以直接从本地开发人员工作区运行游戏，以实现快速迭代。

![image-20220718164853252](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718164853252.png)

### 12.7优化加载时间

较短的加载时间对于开放世界场景游戏而言至关重要，而且对于任何类型的游戏来说也非常重要。虚幻引擎提供了几种方法来优化项目在打包过程中的加载时间。以下是缩短游戏加载时间的一些推荐做法。有关如何打包项目的信息，请参阅[打包和烘焙游戏](https://docs.unrealengine.com/5.0/zh-CN/packaging-and-cooking-games-in-unreal-engine)部分。

### 12.8使用事件驱动加载器（Event Driven Loader，EDL）和异步加载线程（Asynchronous Loading Thread，ALT）

- 默认情况下，**异步加载线程（Asynchronous Loading Thread）**(ALT)是关闭的，但是可以在引擎（Engine）>流送（Streaming）部分下的项目设置（Project Settings）菜单中打开。对于修改过的引擎，可能需要进行一些调整，但一般来说，ALT应该会将加载速度提高一倍，包括具有"预先"加载时间的游戏和持续流送数据的游戏。ALT的工作方式是在两个独立的线程上同时运行序列化和后加载代码，因此，它增加了一项要求，即游戏代码中的"UObject"类构造函数、"PostInitProperties"函数和"Serialize"函数必须具有线程安全性。激活ALT后，ALT会将加载速度提高一倍。有关使用异步加载方法（在C++中）的更多信息，请参阅[异步资源加载](https://docs.unrealengine.com/5.0/zh-CN/asynchronous-asset-loading)页面。

![image-20220718165356554](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718165356554.png)

- 默认情况下激活 **事件驱动加载程序（Event-Driven Loader）**，但是可以在"引擎（Engine）>流送（Streaming）"部分下的"项目设置（Project Settings）"菜单中禁用。对于大多数项目，EDL会将加载时间减半。EDL是稳定的，可以向后移植到旧版本的虚幻引擎，也可以针对修改过或自定义的引擎版本进行调整。

### 12.9压缩.pak文件

- 要在项目中使用 `.pak` 文件压缩，打开"项目设置（Project Settings）"并找到"打包（Packaging）"部分。在此部分，打开"打包（Packaging）"标题的高级部分，并选中出现的标有"创建压缩烘焙包（Create compressed cooked packages）"的复选框。
- 大多数平台不提供自动压缩，且压缩你的 `.pak` 文件将减少加载时间，但有一些特殊情况需要考虑：

| **Steam**  | Steam在用户下载文件时压缩文件，所以初始下载时间不会受到游戏正在压缩的.pak文件的影响。然而，Steam的差分补丁系统在未压缩文件时会运行得更好。压缩的.pak文件可以节省在客户系统上的空间，但是在打补丁时需要更长的时间来下载。 |
| ---------- | ------------------------------------------------------------ |
| **Oculus** | 不启用 `.pak` 文件的压缩。Oculus补丁系统无法正确处理压缩后的 `.pak` 文件。此外，压缩 `.pak` 文件也不会减小文件大小。 |

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718170012941.png" alt="image-20220718170012941" style="zoom: 50%;" />

![image-20220718170349453](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718170349453.png)

### 12.10对pak文件排序

井井有条的 `.pak` 文件对于减少加载时间至关重要。为了帮助你以最佳方式对 `.pak` 文件进行排序，UE4提供了一组工具来发现所需数据资产的顺序，并编译更快加载的包。从概念上讲，此过程类似于基于配置文件的优化。按照该方法对我们的 `.pak` 文件进行排序：

1. 使用命令行选项 `-fileopenlog` 编译并运行打包游戏，这将导致引擎记录打开文件的顺序。
2. 测试游戏的所有主要方面。加载每个关卡、每个可操作角色、每个武器、每个载具等等。加载所有内容后，退出游戏。
3. 在部署的文件中，将有一个名为"GameOpenOrder.log"的文件，其中包含优化.pak文件顺序所需的信息。例如，在Windows各版本上，你将在"WindowsNoEditor/(YourGame)/Build/WindowsNoEditor/FileOpenOrder/"中找到该文件。将该文件复制到开发目录"/Build/WindowsNoEditor/FileOpenOrder/"路径。
4. 日志文件就绪后，重新编译 `.pak` 文件。该 `.pak` 和未来生成的所有 `.pak` 文件都将使用日志文件中规定的文件顺序。

在生产环境中，日志文件应该检入源码控制，并定期更新新的 `-fileopenlog` 运行结果，包括游戏准备发行时的最后一次运行。

## 13.管理内容

并不是所有的游戏内容都是在编辑器中创建的。大部分的美术素材都应该在外部的工具中制作，比如 3ds Max，Maya，Photoshop，ZBrush 等。下表中罗列了一些**典型**的应该在编辑器内部制作的素材，以及哪些应该在外部工具中创建。

| **由虚幻编辑器中创建**                         | **由外部应用程序中创建**        |
| ---------------------------------------------- | ------------------------------- |
| 游戏关卡                                       | 静态网格物体（Static Meshes）   |
| 材质                                           | 骨架网格物体（Skeletal Meshes） |
| 粒子系统                                       | 骨架动画（Skeletal Animation）  |
| 过场动画序列                                   | 材质（Textures）                |
| 蓝图脚本                                       | 声音（WAVs）                    |
| 给人工智能用的导航网格（AI Navigation Meshes） | IES 灯光信息                    |
| 预计算光照信息（Light Maps）                   | Nvidia APEX 文件（APB 及 APX）  |
| 场景（光卡）光照                               |                                 |

### 13.1FBX内容管线(*)

有关将FBX内容导入通道用于网格体、动画、材质和纹理的信息。

虚幻引擎支持以多种文件格式将内容导入项目。

FBX是一种灵活的文件格式，归Autodesk所有，可以提供数字内容创建（DCC）应用程序之间的互操作性。某些应用程序（例如Autodesk Motionbuilder）本身支持该格式。而Autodesk Maya、Autodesk 3ds Max和Blender等其他软件使用FBX插件支持该格式。

与其他导入方法相比，虚幻FBX导入通道的优点是：

- 对静态网格体、骨骼网格体、动画和变形目标使用单一文件格式。
- 在一次导入操作中导入多个LOD和Morph/Blendshape。
- 导入材质和纹理资产，并自动将它们应用到静态网格体。

<font color='red'>虚幻引擎FBX导入通道使用 **FBX 2020.2** 。在导出期间使用不同的版本可能会导致不兼容。</font>

#### 13.1.1FBX动画流程

使用FBX内容通道设置、导出和导入骨架网格体的动画。

FBX导入通道支持动画，使用者可通过简单工作流从3D软件将 *骨架网格体* 动画导入虚幻引擎以便在游戏中使用，当前单个文件中只能导入/导出每个 *骨架网格体* 的单一动画。

此页面是使用FBX内容通道将动画导入虚幻引擎的技术概览。

##### 13.1.1.1命名

使用FBX格式将动画导入虚幻引擎时，AnimationSequence将被设为与文件相同的命名。随骨架网格体导入动画时，创建的AnimationSequence将从动画序列中的根骨骼获取命名。导入进程完毕后，可通过 **内容浏览器** 进行重命名。

##### 13.1.1.2创建动画

动画可以特定于一个 *骨架网格体*，也可以重复用于多个骨架网格体（前提是每个 *骨架网格体* 使用的骨架相同）。使用FBX内容通道创建动画并将其导入虚幻引擎实际上只需要一个**带动画的骨架**。是否将网格体绑定到骨架则完全取决于使用者，绑定后创建动画将更为简单，因为使用者可以清楚地看到网格体在动画中的变形。而在导出时则只需要骨架。

##### 13.1.1.3从3D应用程序导出动画

**动画必须单个导出**；单个文件包含每个 *骨架网格体* 的一个动画。下方的步骤将说明单个动画如何将其自身导出到一个文件。绑定到此骨架的网格体已隐藏，因为动画自行导出时并不一定需要它们。

1.在视口中选择要导出的关节。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718184242119.png" alt="image-20220718184242119" style="zoom:67%;" />

2.在 **文件（File）** 菜单中选择 **导出选项（Export Selection）**（如果需要无视选择导出场景中的所有内容，则选择 **导出所有（Export All）**）

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718184310567.png" alt="image-20220718184310567" style="zoom:67%;" />

3.选择动画导出的FBX文件的所在路径和命名，并在 **FBX导出（FBX Export）** 对话中设置正确选项。为便于导出动画，必须启用 **动画（Animations）** 勾选框。

![image-20220718184401458](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718184401458.png)

4.点击 ![maya_export_button.jpg](https://docs.unrealengine.com/5.0/Images/working-with-content/fbx-content-pipeline/fbx-animation-pipeline/maya_export_button.jpg) 按钮创建包含网格体的FBX文件。

##### 13.1.1.4导入动画

FBX动画导入通道可一次性导入 *骨架网格体* 和动画，或单独导入网格体/动画。

**含动画的骨架网格体**

1. 点击 **内容浏览器** 中的 ![import_button.png](https://docs.unrealengine.com/5.0/Images/working-with-content/fbx-content-pipeline/fbx-animation-pipeline/import_button.jpg) 按钮。在打开的文件浏览器中找到并选中需要导入的FBX文件。**注意：**可能需要在下拉菜单中选择 ![import_fbxformat.jpg](https://docs.unrealengine.com/5.0/Images/working-with-content/fbx-content-pipeline/fbx-animation-pipeline/import_fbxformat.jpg) ，过滤掉不需要的文件。

<font color='brown'>导入资源的导入路径取决于导入时 **内容浏览器** 的当前位置。在执行导入前必须导航至正确的文件夹。导入完成后也可将导入的资源拖入一个新文件夹。</font>
     2.在 **FBX导入选项** 对话中选择正确的设置。导入网格体的命名将遵循默认的命名规则。请参见[**FBX导入对话**](https://docs.unrealengine.com/5.0/zh-CN/fbx-import-options-reference-in-unreal-engine)，了解到所有设置的完整详情。

​     3.点击 ![button_import.png](https://docs.unrealengine.com/5.0/Images/working-with-content/fbx-content-pipeline/fbx-animation-pipeline/button_import.jpg) 按钮来导入网格体和LOD。如进程成功，结果网格体、动画（AnimationSequence)、材质和纹理将显示在 **内容浏览器** 中。现在即可看到用于保存动画的AnimationSequence，其命名默认为骨架根骨骼的命名。

**单个动画**

要导入动画，首先需要一个可以将动画导入的AnimationSequence。可通过 **内容浏览器** 进行创建，或直接在AnimationSequence编辑器中创建。

1.点击 **内容浏览器** 中的 ![import_button.png](https://docs.unrealengine.com/5.0/Images/working-with-content/fbx-content-pipeline/fbx-animation-pipeline/import_button.jpg) 按钮。在打开的文件浏览器中找到并选中需要导入的FBX文件。**注意：** 可能需要在下拉菜单中选择 ![import_fbxformat.jpg](https://docs.unrealengine.com/5.0/Images/working-with-content/fbx-content-pipeline/fbx-animation-pipeline/import_fbxformat.jpg) ，过滤掉不需要的文件。

2.在 **FBX导入选项** 对话中选择正确的设置。导入网格体的命名将遵循默认的命名规则。请参见[**FBX导入对话**](https://docs.unrealengine.com/5.0/zh-CN/fbx-import-options-reference-in-unreal-engine)，了解到所有设置的完整详情。

<font color='red'>导入动画时，必须指定一个现有的骨架！</font>

3.点击 ![button_import.png](https://docs.unrealengine.com/5.0/Images/working-with-content/fbx-content-pipeline/fbx-animation-pipeline/button_import.jpg) 按钮来导入网格体和LOD。如进程成功，结果网格体、动画（AnimationSequence)、材质和纹理将显示在 **内容浏览器** 中。现在即可看到用于保存动画的AnimationSequence，其命名默认为骨架根骨骼的命名。

虚幻编辑器支持非等分缩放动画。导入动画时，如果存在缩放，其无需额外设置便可直接导入。出于内存原因，引擎不会保存所有动画的缩放，只会保存不为1的缩放。

请参见[**骨架网格体动画（Skeletal Mesh Animation）**](https://docs.unrealengine.com/5.0/zh-CN/skeletal-mesh-animation-system-in-unreal-engine)页面，了解更多内容和视频参考。

#### 13.1.2FBX 资产元数据管道

[虚幻引擎中的FBX资源元数据管线|虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/fbx-asset-metadata-pipeline-in-unreal-engine/)

#### 13.1.3FBX 导入错误(*)

Description：导入 FBX 文件时出现错误的说明。 Parent: working-with-content/fbx-content-pipeline Version: 4.9 Order:1000

此列表包含导入 FBX 文件时可能出现的错误或警告信息。

##### [FBX 导入错误 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/fbx-import-errors-in-unreal-engine/)

#### 13.1.4FBX导入选项参考

FBX导入选项对话框中可用选项的说明。

虽然将FBX文件导入到虚幻引擎4是一个相对简单的过程，但是有相当多的选项可以调整导入的资产。本文档将介绍这些选项。

当你使用FBX管道通过 **内容浏览器** 导入内容时，将出现 **FBX导入选项（FBX Import Options）** 对话框。导入器将自动检测你要导入的文件类型，并相应地调整其接口。

##### 13.1.4.1静态网格体选项

使用FBX导入 *StaticMesh* 时可用的选项如下所示。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718191056573.png" alt="image-20220718191056573" style="zoom:50%;" />

选项的详情见

[虚幻引擎FBX导入选项参考 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/fbx-import-options-reference-in-unreal-engine/)

##### 13.1.4.2骨架网格体选项

使用FBX导入 *SkeletalMesh* 时可用的选项如下所示。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718191259669.png" alt="image-20220718191259669" style="zoom:50%;" />

选项的详情见

[虚幻引擎FBX导入选项参考 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/fbx-import-options-reference-in-unreal-engine/)

##### 13.1.4.3动画选项

使用FBX导入动画时可用的选项如下所示

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718191511722.png" alt="image-20220718191511722" style="zoom:50%;" />

选项的详情见

[虚幻引擎FBX导入选项参考 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/fbx-import-options-reference-in-unreal-engine/)

##### 13.1.4.4变换

下面将解释使用FBX导入任何静态或骨架网格体资产时可用的选项。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718191616616.png" alt="image-20220718191616616" style="zoom:50%;" />

选项的详情见

[虚幻引擎FBX导入选项参考 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/fbx-import-options-reference-in-unreal-engine/)

##### 13.1.4.5杂项

下面将解释使用FBX导入任何静态或骨架网格体资产时可用的其他各种选项。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718191904321.png" alt="image-20220718191904321" style="zoom:67%;" />

选项的详情见

[虚幻引擎FBX导入选项参考 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/fbx-import-options-reference-in-unreal-engine/)

##### 13.1.4.6材质选项

使用FBX导入材质时可用的选项如下所示。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718191957072.png" alt="image-20220718191957072" style="zoom: 67%;" />

##### 13.1.4.7命名规范

下表显示了在启用 **覆盖全名（Override FullName）** 时如何命名各种内容类型。

该表假定以下条件：

- **%1** 是要导入的资产的名称，即导入路径的最后一部分。
- **%2** 是FBX文件中的网格体节点名称。对于骨架网格体，如果它由多个FBX网格提组成，则使用第一个FBX网格体名称作为FBX节点名称的一部分。

表见

[虚幻引擎FBX导入选项参考 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/fbx-import-options-reference-in-unreal-engine/)

#### 13.1.5FBX材质管道

有关使用FBX内容管道传输基本材质和纹理与网格体的指南。

FBX管道将应用于网格体（静态网格体和骨架网格体）的材质和纹理从3D应用程序传输到虚幻。 要转换简单材质，可以导入源材质中使用的纹理，这样会在虚幻中创建已经将纹理连接到相应通道的材质， 最后将材质应用于导入的网格体。FBX管道简化了网格体导入流程， 自动完成过去需要人工完成的复杂流程。

##### 13.1.5.1材质支持

FBX管道仅支持导入基本材质。可以传输的材质类型包括：

- Surface

- Anisotropic

- Blinn

- Lambert

- Phong

- Phone E

  目前，支持随着网格体导入的贴图（纹理）将添加到材质，某些常见类型将连接到材质的默认输入，但某些则需要手动连接。 此外，一些不太常见的贴图类型可能无法导入，例如Maya中用于环境光遮蔽的漫反射通道。

除了这些材质类型之外，还可以传输这些材质的仅特定功能。FBX材质管道不传输单独的设置， 但支持传输材质使用的特定贴图或纹理。

##### 13.1.5.2多种材质

网格体自身可以应用若干材质，每个材质覆盖网格体的特定表面，而FBX能够处理包含多个材质的网格体的导入（假设它们已经在3D应用程序中正确设置）。

就网格体上使用多种材质而言，**Maya非常简单明了。您只需选择想要对其应用材质的网格体表面，然后应用材质即可**。

针对Maya中应用于网格体的每个材质，都将在虚幻编辑器中创建一个材质，导入的网格体对于其中每种材质都有对应的材质插槽。应用于网格体后，材质仅影响网格体的对应多边形，就像Maya中一样。

##### 13.1.5.3材质命名

虚幻编辑器在导入过程中创建的材质将根据3D应用程序中的源材质命名。具体从哪里抽取名称，则取决于是从哪个应用程序导出网格体的。

如果来自于Maya，则虚幻编辑器中的材质名称取自Maya中应用于网格体的着色引擎名称。

##### 13.1.5.4材质顺序

当材质最初从FBX导入时，材质名称将分配到**材质插槽**，这样重新导入FBX时，可以使用 **原始导入材质名称** 将材质 与正确的元素索引相匹配。这种方法比使用`Skin##`命名约定来确定材质顺序更加一致，可以保证导入流程直接查找FBX文件中的名称， 以确定哪个分段应该与列表中已经填充的现有材质相匹配。这里的"插槽名称（Slot Name）"将匹配网格体" 细节层次（Level of Detail，LOD）"部分中的"材质名称（Material Name）"下拉选择。

//材质插槽取代了skin##

如果您将鼠标悬停于 **插槽名称（Slot Name）** 字样上方，工具提示将列出已经导入的**材质名称**。4.14之前导入的任何静态网格体或骨架网格体将在工具提示中显示`None`材质名称。

##### 13.1.5.5添加或移除材质插槽

要添加或移除任何材质插槽，请使用"材质（Materials）"列表顶部的 **添加**（**+**）按钮和"插槽名称（Slot Name）"旁边的 **移除**（**x**）按钮。添加的插槽可以用来覆盖较低LOD分段，但不能覆盖基本LOD。

##### 13.1.5.6在蓝图和C++中设置材质插槽

在运行时，调用 **按名称设置材质**，使用您为材质指定的**插槽名称**来设置组件上的材质。这样您就不必对材质元素索引号进行硬编码 来检索您所寻找的材质插槽。还可以确保Gameplay代码在网格体上的材质顺序一旦发生变化（因为它引用的是插槽名称，因此不太容易变化）时也能正常工作。

##### 13.1.5.7纹理导入

如果材质在3D应用程序中分配了纹理作为漫反射或法线贴图，只要在[FBX导入属性（FBX Import Properties）](https://docs.unrealengine.com/5.0/zh-CN/fbx-import-options-reference-in-unreal-engine)中启用了 **导入纹理（Import Textures）**，就可以导入这些纹理。

将在虚幻编辑器中新创建的材质中将构建纹理取样表达式，导入的纹理将分配到该纹理取样。系统还会向材质添加纹理坐标表达式，并将它连接到纹理取样的 **UV** 输入。但是，您需要将某些纹理连接到它们的材质插槽。

如果在3D应用程序中应用于材质的纹理格式与虚幻不兼容，或者连接到了未知材质属性（例如，Maya中的漫反射），则它们不会导入。在此情况下，以及材质中不存在纹理的情况下，虚幻编辑器中的材质将通过随机着色的矢量参数进行填充。

#### 13.1.6FBX变换目标管线

使用FBX内容通道为骨架网格体创建和导入变换目标。

**变换目标（Morph Target）** 是特定网格体的顶点位置的快照，该网格体在某种程度上已经变形。例如，您可以使用一个角色模型，对其面部进行重塑以创建一个面部表情，然后将编辑后的版本保存为变换目标。在Unreal中，您可以混合变换目标以使角色面部做出该表情。变换目标可以通过FBX导入到Unreal中，并封装在动画序列中。

##### 13.1.6.1命名

当使用FBX格式将变换目标导入Unreal时，各变换目标将根据三维应用程序中的混合形状或变换的名称命名。

- 该名称将是添加到blendshape节点名称中的blendshape的名称，即"[BlendShapeNode]_[BlendShape]"。

##### 13.1.6.2设置变换目标

在Maya中设置要导出到FBX的变换目标需要使用混合形状。下面的步骤简要说明了为导出设置变换目标所需的步骤。有关更详细的信息，请参阅应用程序的帮助文件。

[虚幻引擎中的FBX变换目标管线 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/fbx-morph-target-pipeline-in-unreal-engine/)

#### 13.1.7FBX 场景导入

[FBX Scene Import in Unreal Engine | 虚幻引擎5.0文档](https://docs.unrealengine.com/5.0/zh-CN/fbx-scene-import-in-unreal-engine/)

#### 13.1.8骨架网格体管道

使用FBX内容通道设置、导出和导入骨架网格体。

FBX 导入管道支持 *骨架网格体（Skeletal Mesh）*。这提供了一种简化的处理流程来将有动画的网格体从 3D应用程序中导入到虚幻引擎内，以便在游戏中使用。除了导入网格体外，如果需要，动画和变形目标都可以使用FBX格式 在同一文件中传输。同时，还可以 导入3D应用程序中给这些网格体应用的材质所使用的纹理（仅限漫射和法线贴图）， 并且自动创建材质，将其应用于导入的网格体。

以下是使用FBX导入骨架网格体所支持的功能：

- [材质（包括纹理）](https://docs.unrealengine.com/5.0/zh-CN/fbx-skeletal-mesh-pipeline-in-unreal-engine#材质)
- [动画](https://docs.unrealengine.com/5.0/zh-CN/fbx-skeletal-mesh-pipeline-in-unreal-engine#动画)
- [变形目标](https://docs.unrealengine.com/5.0/zh-CN/fbx-skeletal-mesh-pipeline-in-unreal-engine#morphtargets)
- 多个UV集合
- 平滑组
- [顶点颜色](https://docs.unrealengine.com/5.0/zh-CN/fbx-skeletal-mesh-pipeline-in-unreal-engine#vertexcolors)
- [LOD](https://docs.unrealengine.com/5.0/zh-CN/fbx-skeletal-mesh-pipeline-in-unreal-engine#meshlods)

目前，对于每个 *骨架网格体*，只能将单个动画导入单个文件。但是，在一个文件中可以传输 一个 *骨架网格体* 的多个变形目标。

##### 13.1.8.1一般设置

###### 单一网格体和由多部分构成的网格体

*骨架网格体* 可以由一个连续网格体构成，也可以由几个独立的网格物体构成， 所有网格体都对同一个骨架进行皮肤处理。

使用多个网格体时，每个构成部分的LOD可以不同，并且每个部分可以单独导出， 以便在模块化的角色系统中使用。这种创建 *骨架网格体* 的方式不会使性能降低。 每个构成部分导入到虚幻编辑器之后，它们会组合到一起。

###### 13.1.8.1.1绑定

绑定是指将网格体绑定到骨骼/关节的骨架层级。这使得底下骨架的骨骼/关节可以影响网格体的顶点，当骨骼或关节移动时会使得**网格体发生变形**。

Maya使用 *平滑绑定（Smooth Bind）* 命令将网格体绑定到骨架。无论 *骨架网格体* 是由一个完整网格体还是由多个网格体部分构成，过程都是相同的。

[虚幻引擎骨架网格体管道 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/fbx-skeletal-mesh-pipeline-in-unreal-engine/)

###### 13.1.8.1.2骨架

在Maya中，一般使用 *关节工具* 为 *骨架网格体* 创建骨架。同样，也有 无数关于在Maya中如何使用这个工具及创建绑定的教程。Maya帮助文档也是获得关于这个主题信息 的很好资源。

#### 13.1.9FBX静态网格体管线

FBX导入流程中加入 *静态网格体* 支持后，将网格体从3D软件加入虚幻引擎4的操作便极为简便。网格体导入后，应用到3D软件中网格体的材质纹理（仅限漫反射和法线贴图）也将被导入，并用于生成应用到虚幻引擎4中网格体的材质。

[虚幻引擎中的FBX静态网格体管线 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/fbx-static-mesh-pipeline-in-unreal-engine/)

### 13.2毛发渲染与模拟

关于在虚幻引擎中渲染、模拟、创建和编辑毛发造型的信息。

虚幻引擎的毛发渲染与模拟系统利用基于**发束的工作流来渲染每束毛发**，让毛发在移动时准确地遵循物理原则。借助此系统，美术师可以在DCC中先创建出毛发造型（groom），然后实时模拟并渲染出数以千万计（甚至更多）的照片级毛发。

[虚幻引擎毛发渲染与模拟 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/hair-rendering-and-simulation-in-unreal-engine/)

<font size=5>Groom资产编辑器用户指南</font>

[虚幻引擎Groom资产编辑器用户指南 | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/groom-asset-editor-user-guide-in-unreal-engine/)

### 13.3骨骼网格体

绑定到层次化骨骼骨架上的网格物体，它可以产生动画，以便使网格物体发生变形。

骨架网格体由两部分构成：构成骨架网格体表面的一组多边形，用于是使多边形顶点产生动画的一组层次化的关联骨骼。

在虚幻引擎 4 中，通常使用骨架网格体代表**角色或其他带动画的对象**。3D 模型、绑定及动画都是在外部建模和动画应用程序（3DSMax、Maya、Softimage 等）中创建的，然后通过虚幻编辑器的内容浏览器把这些资源导入到虚幻引擎 4，并将其保存到包中。

#### 13.3.1逐平台LOD

虽然拥有多个细节层级(LOD)骨架网格体可以有助于降低距离对象的渲染成本，但在内存等资源有限的平台上，存储这些信息所需的额外内存可能是个问题。在下面的教程中，我们将介绍如何限制平台可以使用的骨架网格体LOD数量。

[逐平台LOD | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/per-platform-lod/)

#### 13.3.2骨架网格体到静态网格体的转换（*）

如能够设置项目角色的动作并将这些动作保存到之后使用，采集截图制作市场或宣传材料时便会十分轻松。在以下文档中，我们将说明如何设置虚幻引擎 4（UE4）项目角色的动作，并将这些动作保存为静态网格体，以便之后使用。

##### 13.3.2.1设置骨架网格体的动作

1.首先在 **Content Browser** 中找到并双击打开 **SK_Mannequin** 骨架网格体资源。

2.自由发挥

##### 13.3.2.2转换多个放置的骨架和静态网格体

选中所有网格体后，右键点击其中一个网格体。在出现的快捷菜单中选择 **Convert Actors To Static Mesh** 选项。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220718204316911.png" alt="image-20220718204316911" style="zoom:50%;" />

### 13.4静态网格体（*）

关于在虚幻引擎中导入和操作静态网格体的信息。

静态网格体资产是虚幻引擎中的一种**基本单位**，用于在关卡中创建场景几何体。这些3D模型在外部建模程序（如3dsMax、Maya、Blender等）中创建，并通过内容浏览器**导入**到虚幻编辑器中。使用虚幻引擎制作的关卡中，绝大部分内容都是由静态网格体组成；这些内容通常以静态网格体Actor的形式存在。

由于静态网格体被缓存在**显存**中，所以它们可以平移、旋转和缩放，并且可以比其他类型的几何体更加复杂。

#### 13.4.1创建并使用LOD

当玩家靠近您在场景中放置的静态网格模型时，您想要让网格模型看起来非常细致。但是，一旦玩家远离网格模型，您就不需要让网格模型那么细致和复杂了。如果网格模型在屏幕上只占了几个像素的位置，并且玩家几乎看不到它，就没必要让它看起来非常复杂和细致。但是，当玩家靠近网格模型并且能够很清楚地看到它，网格模型就需要细致点了。在 UE4 中，您可在场景中放置一个网格模型，当玩家远离它的时候，可让该网格模型切换为不复杂的网格模型，以便让场景运行得更流畅。您可通过使用 **Level of Details** 或 **LODs** 来达到上述效果。本使用说明将向您介绍如何将不太细致版本的网格模型导入 UE4，**然后让网格模型随着玩家的靠近与远离，从一个网格模型无缝切换为另一个网格模型**。

##### 13.4.1创建LOD

找到该资源后，在静态网格模型编辑器中打开它，打开方式为 **double-clicking** 该资源或 **right-clicking** 该资源并从出现的关联菜单中选择 **Edit**。现在，您将可以看到类似以下画面。

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220719082323105.png" alt="image-20220719082323105" style="zoom:67%;" />

[在虚幻引擎中创建并使用LOD | 虚幻引擎5.0文档 (unrealengine.com)](https://docs.unrealengine.com/5.0/zh-CN/creating-and-using-lods-in-unreal-engine/)

## 14.UI

再border下面加button会让鼠标左键点击失效



## 15.RootMotion

<font size=6>作用</font>

让角色动画呈现更加丝滑，并且对于带有位移的动画(瞬移，跳劈这样的)，可以让整个角色根据动画移动，而不是只有网格体(模型)在动



步骤:

<font size=5>Maya</font>

1.在maya里面给骨骼添加根骨骼(root)

2.选中root在动画编辑器里面把原来的根骨骼(一般叫Hips)的X和Z轴复制给Root,在把Hips里面的归0

<font size=5>UE4</font>

动画序列,启用根运动勾选

![image-20220803150415132](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220803150415132.png)



动画蓝图

![image-20220803150538520](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220803150538520.png)

角色蓝图(让AI根据rootmotion平滑地移动)

characMovementComponent

![image-20220803150620334](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220803150620334.png)

class default

![image-20220803150728740](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220803150728740.png)







引擎操作注意事项

使用游戏已暂停节点会导致玩家输入失效(因为playercontroller也被暂停了)
