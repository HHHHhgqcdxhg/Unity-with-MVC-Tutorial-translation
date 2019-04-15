# Unity with MVC: How to Level Up Your Game Development

> [Unity with MVC: How to Level Up Your Game Development](https://www.toptal.com/unity-unity3d/unity-with-mvc-how-to-level-up-your-game-development)的翻译  
没有找到好的国内的unity mvc的资料,国外的找到了这个.  
自己翻译下,功力尚浅望轻喷.(页面最下面有评论板块)  
原文地址: [https://www.toptal.com/unity-unity3d/unity-with-mvc-how-to-level-up-your-game-development](https://www.toptal.com/unity-unity3d/unity-with-mvc-how-to-level-up-your-game-development)  

<code>(以括号开头和结尾的代码块为译者的注释和吐槽)</code>
  
程序猿经常从经典的<code>Hello World</code>着手开始学习.从那开始,项目越做越大.每个新挑战都指向一个重要的课题:  

项目越大,就越乱. 

不久后,就可以看到大小团队不会随意的不顾一切的做项目.代码应该被维护且能长时间持续使用.你工作过的公司不会只要你联系方式而是每次需要修或改善代码的时候叫你(然而你不希望这样).  

这就是[软件设计模式](https://www.toptal.com/agile/ultimate-introduction-to-agile-project-management)存在的理由;他们利用简单的规则来操控一个软甲工程的整体架构.他们使一个或多个程序猿把一个大项目的部分分开成一些核心部分来写,并且用一个标准化的方法来组织,使得遇到不熟悉的代码的部分时能消除混乱  

如果所有人都遵守这些规则,就会使legacy代码能更好的维护和操纵,并且新的代码可以更灵活地添加进去.规划开发的方法论花费的时间更少.Since problems don’t come in one flavor, there isn’t a silver bullet design pattern.你必须仔细考虑每个模式的优点和缺点,然后找到最适合拿来解决手头的问题的.  

在这份教程中,我会结合我在[Unity游戏开发平台](http://unity3d.com/)和Model-View-Controller(MVC)模式上的经验.在我七年的开发中,我与我乱成一锅粥的游戏开发正面对决中,我利用设计模式得到了很好的代码结构和开发速度.  

我一开始会讲一些uninty的基础架构:Entity-Component模式.然后我会解释MVC如何适应它,并且用一些模拟的项目来解释它.  

## 动机
在软件相关的文献中,我们可以找到一大堆关于设计模式的.纵然他们有一系列规则,开发者仍然经常需要做一些小的改动来使设计模式能更适应他们独特的问题.  

这种"程序编制的自由性"证明了我们找不到一个单一的,确定的方法来设计软件.所以,这篇文章不应是解决你问题的终极方案,而是展示两种流行的设计模式:Entity-Component和Model-View-Controlle的效益和发展潜力.  

## Entity-Component模式
Entity-Component(EC)是一种设计模式,我们首先定义组成应用的元素们的层级(Entities),然后定义将要被包含的特征和数据(Components).在更多的"程序猿"看来,一个Entity可以是一个包含了有0个或多个Components的对象.让我们这样描述一个Entity:  

<code>
some-entity [component0, component1, ...]
</code>

这里是一个简单的EC树的例子:  

```
- app [Application]
    - game [Game]
        - player [KeyboardInput, Renderer]
        - enemies
            - spider [SpiderAI, Renderer]
            - ogre [OgreAI, Renderer]
        ui [UI]
            - hud [HUD, MouseInput, Renderer]
            - pause-menu [PauseMenu, MouseInput, Renderer]
            - victory-modal [VictoryModal, MouseInput, Renderer]
            - defeat-modal [DefeatModal, MouseInput, Renderer]
```

EC在缓解多重继承的问题方面是一个好的模式,当一个复杂的类结构引出问题时,例如[钻石问题](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem): 一个类D,继承自两个类,B和C,而B和C继承自同一个类A,这时就会引发冲突,因为B和C会不同地修改A的属性.  
![20190414221244.png](https://pic.ggemo.com/picgo/20190414221244.png)  

因为在游戏开发中继承应用很广泛,所以这种问题会很常见.  

通过分解小的Component的属性和数据操作器,可以使他们可以附加在不同的Entitiy上并重用,而不用依赖于复杂的继承关系(顺便,这不只是C#或JavaScript,Unity常用的语言的特征).  

### Entity-Component模式的不足
EC在OOP之上一个层面,EC有利于整理和更好的组织你的代码架构.然而,在大项目中,我们仍然"太过随意"然后我们会发现自己身处一个"属性海洋",可能会费一段艰苦的时光来找到对的Entity和Component,或搞清楚它们应该如何相互影响.一个给定的任务中会有无穷种组合Entity和Component的方式.  
![20190414222558.png](https://pic.ggemo.com/picgo/20190414222558.png)  

一个摆脱混乱的途径是在Entity-Component之上添加一些的额外的参考.比如,我喜欢的一种考虑软件的方式是将其差分成三种不同的类别:  
- 一些处理原数据,允许创建,读取,更新,删除和查找(也就是[CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete)).
- 另一些实现一些接口来和其他元素相互作用,检测他们范围内相关的事件并且当发生是触发通知.  
- 最后,一些元素负责接收这些通知,制定事务逻辑决策,并决定如何操作数据.  

幸运的是,我们已经有表现得符合这些需求的设计模式了.

## Model-View-Controller(MVC)模式
[ Model-View-Controller pattern](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)(MVC)模式将软件分成三个主要模块:Models (数据CRUD), Views (界面/事件检测) and Controllers (决策/处理).MVC实施起来足够灵活,即使是建立在ECS 或OOP之上.  

游戏和UI的开发有通常的工作流程: 等待用户输入,或其他的触发条件,向合适的地方发送这些通知,决定如何响应,并做出相应的数据操作.这些行为清楚地展现了MVC应用的兼容性.  

这种方法论引入了其他的抽象层,会对软件规划有帮助,并且允许新的程序猿应对一个就算很大的代码库.通过将思考过程分离成数据,接口和决策可以减少必须通过查找来添加或修改功能的源文件.

## Unity和EC
让我们仔细看看unity预先给我们了什么.  
Unity是一个建立在EC上的开发平台,所有的Entity都是<code>GameObject</code>的实例,它们的属性可以使他们"可见","可移动","可交互",等等,都是由继承<code>Component</code>的类提供的.  

Unity editor的**Hierarchy面板**和**Inspector面板**提供了装配你的应用的强大的方法,用比通常情况下更少的代码来附加组件,配置它们的初始化状态和引导你的游戏.  
![Hierarchy Panel with four GameObjects on the right](https://pic.ggemo.com/picgo/20190414232815.png)|![Inspector Panel with a GameObject’s components](https://pic.ggemo.com/picgo/20190414232850.png)
---|---

尽管如此,我们讨论过的,我们会遇到"太多属性"的问题并且发现我们身处一个庞大的层级中,属性分散在各个地方,使开发者更加难以存活.  

按MVC的方式思考,我们可以按事物的功能将它们分类,使我们的应用的结构像下面这个例子这样:  
![20190414233458.png](https://pic.ggemo.com/picgo/20190414233458.png)  

### 将MVC打造进游戏开发环境
现在我要介绍两种简单的 一般的MVC模式的变种,可以将其打造进我曾用MVC建立Unity项目时遇到的特殊的情况:  

1. MVC的类容易散布在代码里  
    - 在Unity中,开发者通常必须拖拽对象来获取它们,或者使用类似<code>GetComponent( ... )</code>的笨重的查找语句来获取对象
    - 如果Unity崩溃了或者一些bug使所有的拖拽来的引用丢失了,会使该死的引用丢失接连发生.  
    - 必须使用一个独特的根引用对象,通过它来使**Application**中的所有对象可获取和重新获得.  
2. 一些元素封装了一些 一般的,需要高度重用的,不能自然地分类到Model,View或Controller中的功能.我简单地叫它们**组件**.它们也是Entity-Component中的"Components"的概念但是在MVC框架中仅仅作为辅助者.  
    - 例如,一个<code>Rotator</code>组件,只会按照角速度旋转物体,但不会通知,存储,或决定任何事.  

为了帮助缓解两种问题,我提出了一种魔改的设计模式,我叫它**AMVCC**,或Application-Model-View-Controller-Component.  
![20190415001843.png](https://pic.ggemo.com/picgo/20190415001843.png)  

- **Application**
    你的应用的单独的入口,是所有决定性的实例和依赖应用的数据的容器
- **MVC**
    到目前为止你应该知道的. :)
- **Component**
    可复用的,小的,容易被包含的脚本

## 例子: 10 Bounces
作为一个简单的例子,让我们来看下一个简单的被叫做<strong>_10 Bounces_</strong>的游戏,在此我会使用AMVCC模式的核心要素  

这个游戏的结构很简单: 一个带有一个<code>SphereCollider</code>和一个<code>Rigidbody</code>的<code>Ball</code>(将会在"Play"之后开始下落),一个当做地面的<code>Cube</code>和5个搭建AMVCC的脚本  

### Hierarchy
在写代码之前,我通常会从hierarchy开始并创建我的类和物件的大纲.通常遵守这个新的AMVCC风格.  
![20190415003635.png](https://pic.ggemo.com/picgo/20190415003635.png)  

我们可以看到,<code>view</code> GameObject包含了所有的可见的元素,和其他的包含了<code>View</code> 脚本的物件<code>(???)</code>.  
<code>model</code>和<code>controller</code>组件在一些小项目中通常只包含他们各自的脚本.在大项目中,他们会包含 包含了各自的脚本的GameObjects.  

当其他人拿到你的项目希望看到:  
- **数据**:
    查看<code>application > model > ...</code>  
- **逻辑/工作流程**:
    查看<code>application > controller > ...</code>
- **渲染/接口/监听器**:
    查看<code>application > view > ...</code>  

如果所有的团队都遵从这些简单的规则,遗产项目将不再成为问题<code>(我谔谔)</code>  

需要注意的是,<code>Component</code>没有固定的容器,它们非常灵活,可以在开发者空闲的时候附加到不同的元素上.  

### 写脚本

> **注意**:下面的脚本都是真实世界的实现的抽象版本.一个详细的实现对渲染有太多好处.然而,如果你想浏览更多,[点击这里](https://bitbucket.org/eduardo_costa/thelab-unity-mvc/overview)查看我的个人的Unity MVC框架,*Unity MVC*.你可以找到大多数应用需要的实现AMVCC结构框架的核心classe.

让我们来看一下构成*10 Bounces*的脚本的结构.  

在开始之前,因为和常见的Unity的工作流程不同,让我们先简略地阐明脚本和GameObjects是如何共同工作的.在Unity中,"组件",在Entity-Component的概念中,由<code>MonoBehaviour</code>类表现.一个组件要在运行时存在,开发者需要将其拖拽入一个GameObject(在Entity-Component模式中的"Entity")或者用<code>AddComponent<YourMonobehaviour>()</code>命令.在此之后,脚本就会被实例化并且准备好在执行时使用.  

首先,我们定义<code>Application</code>类(AMVCC中的"A"),作为主要的类,包含所有实例化的游戏元素的引用.我们也可以创建一个辅助的基类叫<code>Element</code>,让我们能拿到Application实例和他的子元素的MVC实例.<code>(子元素的MVC实例是什么鬼啊晕了晕了)</code>  

考虑到这一点,让我们开始定义<code>Application</code>类(AMVCC中的"A"),将要包含一个独特的实例.在它里面,三个变量<code>model</code>,<code>view</code>,和<code>controller</code>,将在运行时给我么提供所有的MVC实例的接入点.这些变量需要是包含 想要的脚本的<code>public</code>引用的<code>MonoBehaviour</code>.  

之后,我们也要创建一个辅助基类叫做<code>Element</code>,来给我们提供Application的实例.  

注意这两个类都要继承<code>MonoBehaviour</code>,他们是将要被附加到GameObject "Entities"上的"组件".  

```C#
// BounceApplication.cs

// 所有元素的基础类都在这个应用中
public class BounceElement : MonoBehaviour
{
   // 应用和所有实例的入口.
   public BounceApplication app { get { return GameObject.FindObjectOfType<BounceApplication>(); }}
}

// 10 Bounces 入口.
public class BounceApplication : MonoBehaviour
{
   // MVC根实例的引用.
   public BounceModel model;
   public BounceView view;
   public BounceController controller;

   // Init things here
   void Start() { }
}
```

我们可以在<code>BounceElement</code>中创建MVC的核心类,<code>BounceModel</code>,<code>BounceView</code>和<code>BounceController</code>通常会是更多专门的实例的容器,不过我们这个例子比较简单,所以只有View有嵌套结构.Model和Controller可以各自用一个脚本来完成:  

```C#
// BounceModel.cs

// 包含这个应用的所有相关数据.
public class BounceModel : BounceElement
{
   // 数据
   public int bounces;  
   public int winCondition;
}
```

```C#
// BounceView .cs

// 包含这个应用的所有相关视图.
public class BounceView : BounceElement
{
   // ball的引用
   public BallView ball;
}
```

```C#
// BallView.cs

// 记录Ball的视图和属性.
public class BallView : BounceElement
{
   // 只有这个是必须的.物理部分在工作的其他部分实现.
   // 碰撞体的回调函数.
   void OnCollisionEnter() { app.controller.OnBallGroundHit(); }
}
```

```C#
// BounceController.cs

// 控制应用工作流程.
public class BounceController : BounceElement
{
   // 处理Ball的碰撞事件
   public void OnBallGroundHit()
   {
      app.model.bounces++;
      Debug.Log("Bounce "+app.model.bounces);
      if(app.model.bounces >= app.model.winCondition)
      {
         app.view.ball.enabled = false;
         app.view.ball.GetComponent<Rigidbody>().isKinematic=true; // 使ball停下
         OnGameComplete();
      } 
   }

   // 处理胜利的情况
   public void OnGameComplete() { Debug.Log("Victory!!"); }
}
```

创建好了所有的脚本之后,我们继续配置和附加它们.  

层级布局会类似这样:  

```
- application [BounceApplication]
    - model [BounceModel]
    - controller [BounceController]
    - view [BounceView]
        - ...
        - ball [BallView]
        - ...
```

拿<code>BounceModel</code>做例子,我们看下它在Unity里是什么样子的:  
![BounceModel with the bounces and winCondition fields.](https://pic.ggemo.com/picgo/20190415103626.png)  

所有的脚本都设定好并且运行之后,我们会在**Console Panel**中看到这样的输出:  
![20190415103754.png](https://pic.ggemo.com/picgo/20190415103754.png)  
<code>(别忘记给ball添加Rigidbody和给ball和ground添加碰撞体)</code>  

### 通知
就像上面这个例子,一个球碰到地板的时候会执行<code>app.controller.OnBallGroundHit()</code>函数.无论如何,在应用中的所有通知都用这种方法并不是"错误"的,但是,以我的经验,在AMVCC的Application类中用一个简单的通知系统能得到更好的结果.  

让我们修改<code>BounceApplication</code>结构来实现:  

```C#
// BounceApplication.cs

class BounceApplication 
{
   // 遍历所有Controller并作为通知数据的代表
   // 这个方法很容易找到,因为每个类都是"BounceElement",都有一个"app"实例
   public void Notify(string p_event_path, Object p_target, params object[] p_data)
   {
      BounceController[] controller_list = GetAllControllers();
      foreach(BounceController c in controller_list)
      {
         c.OnNotification(p_event_path,p_target,p_data);
      }
   }

   // Fetches all scene Controllers.
   public BounceController[] GetAllControllers() { /* ... */ }
}
```

然后,我们需要一个新的脚本让所有开发者添加通知的事件名,这些事件会在运行时进行调度.  

```C#
// BounceNotifications.cs

// 这个类将所有的事件名定义为static.
class BounceNotification
{
   static public string BallHitGround = “ball.hit.ground”;
   static public string GameComplete  = “game.complete”;
   /* ...  */
   static public string GameStart     = “game.start”;
   static public string SceneLoad     = “scene.load”;
   /* ... */
}
```

显而易见,通过这种方式,程序的可读性增加了,因为开发者不需要在所有的源码中找<code>controller.OnSomethingComplexName</code>方法来理解运行时会产生什么样的行为.只需查看一个文件,就可以理解这个应用的所有行为.  

现在,我们只需要改变<code>BallView</code>和<code>BounceController</code>来适应这种新系统:  

```C#
// BallView.cs

// 记录Ball的视图和属性.
public class BallView : BounceElement
{
   // 只有这个是必须的.物理部分在工作的其他部分实现.
   // 碰撞体的回调函数.
   void OnCollisionEnter() { app.Notify(BounceNotification.BallHitGround,this); }
}
```

```C#
// BounceController.cs

// 控制应用工作流程.
public class BounceController : BounceElement
{
   // 处理Ball的碰撞事件
   public void OnNotification(string p_event_path,Object p_target,params object[] p_data)
   {
      switch(p_event_path)
      {
         case BounceNotification.BallHitGround:
            app.model.bounces++;
            Debug.Log("Bounce " + app.model.bounces);
            if(app.model.bounces >= app.model.winCondition)
            {
               app.view.ball.enabled = false;
               app.view.ball.GetComponent<RigidBody>().isKinematic=true; // 使ball停下
               // 通知自身,说不定有其他其他controller响应
               app.Notify(BounceNotification.GameComplete,this);            
            }
         break;
         
         case BounceNotification.GameComplete:
            Debug.Log(“Victory!!”);
         break;
      } 
   }
}
```

大项目将会由很多通知.所以为了避免一个庞大的switch-case结构,可以创建不同的controller来处理不同范围的消息.  

## 实际项目中的AMVCC
上面的例子展示了一个AMVCC模式的应用场合.为了使你的思维方式能符合MVC的三个元素,而且按一个有序的层级展示entity应该明确这种技能.  

在大项目中,开发者需要面对更多的复杂的场景,并且不好的决定一些事物是该放到View层还是Controller层,或者遇到一个给定的class需要更加彻底地分散到更小的模块.  

### 翻阅的规则 (by Eduardo)
并不存在什么"普遍的MVC整理规则".但是有一些简单的规则,我遵守它们来帮我决定一些事物是Model,View,Controller,还是需要分解成更小的模块.

#### Class分类
##### Model
- 包含一个应用的核心数据或状态,比如player的<code>health</code>,或是抢的<code>ammo</code>(弹药)  
- 序列化的,并行的,和/或这两种的变种  
- 加载/保存数据(本地或者网络)  
- 在运行中通知Controller  
- 为游戏的[有限状态机](https://gamedevelopment.tutsplus.com/tutorials/finite-state-machines-theory-and-implementation--gamedev-11867)保存游戏状态  
- 不会接触到Views  

##### View
- 从Model层中获取数据来为用户展示实时的游戏状态.比如,一个View层的方法<code>player.Run()</code>会在内部调用<code>model.speed</code>显示player的行为  
- 不会改动Model层
- 严格的实现自身类的功能.比如:  
   - 一个<code>PlayerView</code>不应该实现输入检测或更改游戏状态  
   - 一个View应该表现得像是有通知重要事件的接口的黑盒子
   - 不存储核心数据(比如速度,血量,...)  

##### Controller
- 不存储核心数据
- 会过滤掉 不希望有的View 发来的通知  
- 更新和使用Model的数据  
- 管理Unity的场景的工作流

#### Class层级
既然这样,我不需要遵守太多的步骤.通常,当我需要给变量名加太多的"前缀"时,我就会意识到需要把类分解,或同个元素有太多的变种(如MMO游戏中的<code>Player</code>类或FPS游戏中的<code>Gun</code>类).  

比如,一个单个的包含了Player数据的<code>Model</code>会有很多<code>playerDataA, playerDataB,...</code>;  
一个处理Player通知的<code>Controller</code>会有很多<code>OnPlayerDidA,OnPlayerDidB,...</code>.  
我们想要减小代码体积并且去掉<code>player</code>和<code>OnPlayer</code>前缀.  

因为只用数据理解起来很简单,所以我们用一个<code>Model</code>类来做例子.  

在写代码时,我通常先用一个单个的<code>Model</code>类包含游戏中的所有数据.  

```C#
// Model.cs

class Model
{
   public float playerHealth;
   public int playerLives;

   public GameObject playerGunPrefabA;
   public int playerGunAmmoA;

   public GameObject playerGunPrefabB;
   public int playerGunAmmoB;

   // Ops Gun[C D E ...] will appear...
   /* ... */

   public float gameSpeed;
   public int gameLevel;
}
```

很容易看出,游戏越复杂,变量数会更多.当它够复杂时,我们可能会以一个庞大的包含了<code>model.playerABCDFoo</code>的类来告终.嵌套的元素会简化代码完成,并且使数据有变种的空间.  

```C#
// Model.cs

class Model
{
   public PlayerModel player;  // Player数据的容器
   public GameModel game;      // Game数据的容器
}
```

```C#
// GameModel.cs

class GameModel
{
   public float speed;         // 游戏运行速度 (影响游戏难度)
   public int level;           // 当前载入的游戏关卡/阶段
}
```

```C#
// PlayerModel.cs

class PlayerModel
{
   public float health;        // Player 血量 (在0.0 和 1.0之间)
   public int lives;           // Player 死后"retry"的次数.
   public GunModel[] guns;     // 当前Player在游戏中可切换的枪的数组
}
```

```C#
// GunModel.cs

class GunModel
{
   public GunType type;        // 列举枪的类型
   public GameObject prefab;   // 枪的3D Asset模板
   public int ammo;            // 当前子弹数
   public int clips;           // 可能的再装填弹药的次数
}
```

有了这些类的配置,开发者可以在某时直观地操纵源码中的一个概念.让我们假设一个武器和它们的配置非常多的FPS游戏.<code>GunModel</code>实际上被包含在一个类中,这个类也可以从一系列Prefabs中为各个种类创建实例并将其存储起来方便以后用到.  

相比之下,如果所有的枪的信息都包含在一个单独的<code>GunModel</code>中,如<code>gun0Ammo</code>,<code>gun1Ammo</code>,<code>gun0Clips</code>等等,然后当用户需要存储<code>Gun</code>数据的时候,将要需要存储整个<code>Model</code>,即便它还包含了不需要的<code>Player</code>数据.在这种情况下,很明显用一个新的<code>GunModel</code>类会比较好.  
![Improving the class hierarchy.](https://pic.ggemo.com/picgo/20190415172440.png)  

就像所有事物都会有优缺点一样.有时人会不必要地过分划分并且增加代码复杂度.只有经验能磨炼你的为你的项目寻找MVC分类的技能.  

> 解锁新的游戏开发特殊能力: 在Unity游戏中使用MVC模式

## 结论
除此之外还有大量的软件设计模式.在这篇文章中,我尝试着展示了在过去的项目中对我帮助最大的一个.开发者应该一直在吸收新的知识的同时一直保持怀疑.我希望这篇教程能帮助你学习新的东西,并与此同时,成为一个你形成自己的开发风格的跳板.  

同时,我真的希望你去多多查找其他的模式来找到最适合你的那个.一个好的出发点是[这篇wikipedia文章](https://en.wikipedia.org/wiki/Software_design_pattern),里面有出色的模式列表和他们的特征.  

如果你喜欢**AMVCC**模式并且想要尝试它,不要忘了尝试下我的库,[<strong>*Unity MVC*</strong>](https://bitbucket.org/eduardo_costa/thelab-unity-mvc/overview),这里包含了所有的,创建AMVCC应用的核心类.  

## 关于作者
[https://www.toptal.com/resume/eduardo-dias-da-costa](https://www.toptal.com/resume/eduardo-dias-da-costa)