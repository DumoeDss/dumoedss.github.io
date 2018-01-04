---
title: StrangeIoc框架介绍
date: 2018-01-04 23:58:53
tags: Unity,StrangeIoc
categories: Unity
---

![](http://strangeioc.github.io/strangeioc/class-flow.png)

```
Strange attractors createpredictable patterns, often in chaotic systems.
```

<!-- More-->

StrangeIoc文档的翻译，转自腾讯GAD，不过GAD上的代码与正文不匹配，按照原文重新排版了一下。

GAD翻译[Unity StrangeIoc 框架介绍](http://gad.qq.com/article/detail/19392)

原文[The Big, Strange How-To](http://strangeioc.github.io/strangeioc/TheBigStrangeHowTo.html)

## 介绍

　　Strange 是一个轻量的高扩展性的控制反转框架，专为C#和Unity而设计，它拥有如下特点，大部分特点是可选的：
　　一个核心的绑定系统，可以支持各种绑定（bindone or more of anything to one or more of anything else.）
依赖注入
1、映射为单例，值或者工厂（每次需要时创建一个新的实例）
2、命名注入
3、 构造函数注入或者setter注入（可以理解为属性注入）
4、标记指定的构造函数
5、标记指定函数在构造函数之后触发
6、注入到Monobehiavours
7、多态绑定：可以绑定接口或者实体类
　　两种风格的共享事件桥（Two styles of shared event bus）
1、都可以发送信号到程序的任何地方
2、都会为本地通信映射本地的事件桥
3、都会映射事件到相应的Command类来分离逻辑
4、使用新的信号实现增加了类型安全保证
　　MonoBehaviour 中介
1、帮助分离view与逻辑
2、隔离unity特有的代码与其它逻辑代码
　　可选的MVCS（Model/View/Controller/Service）结构
　　多个Context
1、允许子控件（子场景）单独运行，或者运行在主app之中
2、允许Context之间通信
　　扩展简单，可以自建新的绑定器：

## 目录介绍：

　　在项目的StrangeIoC > scripts > strange 目录下面有三个子目录，如果在unity里面只能看到两个：
　　Framework 包含构成Strange的主要类
　　Extensions 库函数
　　tests 单元测试

## 1、绑定

　　Strange 的核心内容是对于绑定的一个简单包装。也就是说，基本上我们可以建立各种绑定（bind，connect）。可以绑定一个接口到它的实体类上面。或者绑定一个事件到一个处理者（handler）上。或者绑定两个类：当一个被实例化时，自动创建另一个的实例。为何我们要这么做？很高兴你问了这个问题！当我们编程的时候，大部分的工作最后其实都是绑定。如果你曾触发过一个事件（或者使用Unity3D的SendMessage），如果你遇到过一个类要调用另一个类，如果你写了太多的“if…else”语句，你就是遇到了某种形式的绑定，也就是说，你曾把某些东西与另一些东西绑定。
　　但是直接把一些东西绑定到一起会产生问题，因为这么做会造成代码很难去改变，而且容易出bug。例如，我肯定你编写过表达这类结构的代码：“一个东西，包含了另一些东西作为它的组成部分”。比如一个飞船的类，包含一个炮台和一个键盘控制，你实现了它，一切都看起来很好，直到你的老板说他不想要键盘控制，他想要鼠标控制。所以你重写了飞船类，但是等等，你的飞船类其实并没有改变，只是控制改变了，所以为何你要重写飞船类？
　　你可以创建一个鼠标控制类并使用它，而不是把控制写到飞船类里面。但是如果飞船包含了鼠标控制类的一个引用，你仍然是直接绑定，为了从键盘控制变为鼠标控制（并且可以在老板改变主意的时候再改回来），你不得不改变飞船类内的引用。
　　利用Strange 可以创建间接绑定，减轻你的代码对程序其它部分的依赖性。这是面向对象编程的一个基本原则。只有对象本身可以在不依赖其它实体类的情况下起作用，你的代码才可以称为真正的面向对象。使用绑定器可以让你的代码更加自由灵活。

### 绑定的结构

　　我们快速的看一下一个绑定的结构，这个结构在整个Strange 跟它的所有扩展中都会出现，所以你应该想要了解一下它的运作模式。
　　一个Strange的绑定由两个必须的部分和一个可选的部分组成，这两个必须的部分是一个键值对（key and value），使用键来触发值；因此一个事件可以作为一个键来触发一个回调，或者一个类的实例化可以作为键来引发另一个类的实例化。可选的这部分是一个名称（name）。在某些状况下，我们需要描述使用同样键（key）的两个绑定，这时，使用名称（name）就可以加以区分。
　　这三个部分有两种方法组合到一起，要么使用一个值，要么使用泛型（C# generics）
　　使用泛型会是下面这种结构：

```
Bind<Spaceship>().To<Liberator>();
```

　　”Bind“ 后面是一个键，“To”后面是值。 或者我们也可以使用值来表示绑定：

```
Bind(“MeaningOfLife”).To(42);
```

　　输入“MeaningOfLife” 将得到42。
　　有时候这两种方法会混合使用：

```
Bind<Spaceship>().To(“Enterprise”);
```

　　当我们使用 Spaceship输入将会得到一个字符串“Enterprise”。
　　使用名称（name）时，绑定的形式也不会有大的变化：

```
Bind<IComputer>().To<SuperComputer>().ToName(“DeepThought”);
```

　　最后来说，以下这几种绑定其实是一样的：

```
Bind<IDrive>().To<WarpDrive>();

Bind(typeof(IDrive)).To(typeof(WarpDrive));

IBinding binding = Bind<IDrive>();
binding.To<WarpDrive>();
```

　　区别仅仅是语法糖而已。
　　绑定的形式无穷无尽，Strange提供给你几个非常有用的绑定形式。不仅如此，整个绑定框架是很简单的，所以你可以自己去扩展，创建新的绑定器组件。下面的章节我们会一一介绍这些形式。

## 2、扩展

　　或许你曾听说过Strange是一个依赖注入框架。我对这个描述感到些许不适。是的，Strange提供依赖注入（DI）并且它很有用，但是这个框架的核心，正如我提过的，是绑定（binding）。安装框架时会附带几个有用绑定器扩展，这个部分我们会详细讲解这几个扩展。记住，这并不阻碍你自己去实现你的自定义扩展。
　　注意：下面的章节我提到的是MVCSContext版的Strange，是推荐版本，这个版本包含了下面提到的所有扩展。这是开始学习Strange的最佳方法。

### 注入扩展

　　与控制反转（IoC）关系最大的绑定器扩展是注入器包（injector package）。我们在前面章节提到过注入，在这里将要特别说明一下。
　　你也许对于编写接口很熟悉，一个接口内部并不包含任何实现，它仅仅对与类的输入输出做了一些规范定义。在C#里接口的形式：

```
interface ISpaceship
{
	void input(float angle, float velocity);
	IWeapon weapon{get;set;}
}
```

　　而此接口的实现类例如下面表示：

```
class Spaceship : ISpaceship
{
	public void input(float angle, float velocity)
	{
    		//do stuff here
	}
    
	public IWeapon weapon{get;set;}
}
```

　　使用接口编程，我们减轻了一些类包含另一些类的问题（Thing-Contains-SubThingsproblem）。我们的Spaceship类不再需要包含一个键盘监听器，它只需要一个对于输入的响应方法即可。它也不需要一个炮台，只需要一个实体类（’concrete’class）来实现IWeapon接口。这是一个很大进步。
　　但是这里有个问题：谁来告诉Spaceship使用那种Iweapon（武器）？我们假设Spaceship在一个GameField类（战场）中，或许GameField可以告诉Spaceship使用哪种武器，但是，GameField仍然需要了解实体类，仅仅是依赖的位置变化了，这样是不够的。　
　　GameField 需要有个接口来推送它自己的所有依赖（包括所有Spaceship的依赖），推送到应用的上层。

```
TopOfApp > GameModule > GameField > Spaceship

Phaser --------------------------------------------------->
```

　　这样就不需要实体类了，但是这意味着一个很长的依赖链，贯穿了整个类层次结构。这种结构是脆弱的，只要一个地方发生改变可能破坏很多其它地方，而这些可能被破坏地方又是很难定位的。而且，GameField（或者依赖链上的其它类）需要包含IWeapon。但是GameField 很可能不需要知道IWeapon，为何要为根本不需要的类创建一个依赖？
　　那使用工厂模式呢？ 如果我们创建一个 SpaceshipFactory, 用来创建Spaceship类并且实现了IFactory接口，这样的话GameField只需要这一个依赖，我们就快要成功了。

```
GameField ---------> SpaceshipFactory : IFactory

ISpaceship <---------      (creates concrete Spaceship)
```

　　虽然我仍需要关心ISpaceship现在又多了IFactory，但是不需要关心IWeapon了。然而我可能又要关心IEnemy，仔细想想，我需要把所有工厂都联系起来，并且搞清楚怎么安置它们。貌似这种做法还可以，但是其实这种收到普遍好评的模式有一个致命缺陷。
　　所以我们来考虑一个完全不同的模式，在这个模式中，不会有一个类与另一个类显示的产生依赖。这个模式叫做依赖注入（Dependency Injection， DI）。在DI模式中，一个叫做Injector的类会提供某个类需要的依赖（通常用接口表示）。这种做法通常是利用反射机制来实现。
　　使用依赖注入（DI），如果GameField 需要一个ISpaceship，它将会建立一个依赖如图：

```
ISpaceship <---------      (as if by magic)
```

　　这里并不需要依赖链，或者工厂。除了你的类实际需要的，并没有其它多余的依赖。而且，你再也不需要显示的声明依赖了。
　　那么这一切是怎么做到的?
　　C# 的 System.Reflection （反射）允许一个类在运行时被解构和分析，值得注意的是它的速度并不快，所以我们在Strange里尽量少用。当我们反射一个类时，我们可以检测它的方法和属性，我们可以看到它的构造方法，还有它需要哪些参数。通过检查这些线索，我们可以推断出这个类的依赖是什么，而后给它提供这些依赖。
　　在Strange中设置一个依赖的代码通常会是如下这种形式

```
[Inject]
public IInterface myInstance {get;set;}
```

　　但是Strange是如何知道要提供哪个实体类给接口呢？你可以通过在一个叫做Context的核心文件中绑定依赖来达成。正如我说过的，“标准的”Context是MVCSContext，你可以扩展这个类来实现Strange的各种优点，当你扩展MVCSContext时，你可以使用如下的形式：

```
injectionBinder.Bind<IWeapon>().To<PhaserGun>();
```

　　现在，当一个类需要IWeapon（武器）接口时，它就获得了PhaserGun（激光枪）这个实体类。如果你想要把PhaserGun换成SquirtCannon（散弹炮），你不需要更改其它任何地方，只需要在Context里把代码改成如下：

```
injectionBinder.Bind<IWeapon>().To<SquirtCannon>();
```

　　瞧！飞船（Spaceship）现在用的是散弹炮（SquirtCannon）了。所有这一切仅仅需要一个关键字来表示这里需要依赖注入：

```
class Spaceship : ISpaceship
{
    public void input(float angle, float velocity)
    {
    	//do stuff here
    }
    
    [Inject] //<----- The magic word!
    public IWeapon weapon{get;set;}
}
```

　　值得一提的是，即使你不使用依赖注入，这个[inject]标签也是完全无害的，所以你可以把它加到你的类里，如果有一天你不想使用依赖注入了，这个标签一点也不会影响现有的代码，去掉标签，“weapon”就成了一个普通的属性。

实例化可注入的对象（instances）
　　这里有一个地方要注意。如果你想得到依赖注入带来的好处，你需要做如下两件事：
1、在Context中绑定相关类，这一点在上面已经讨论过。
2、使用InjectionBinder实例化所需对象
　　一开始你可能觉得第二点有些不同寻常，但是它是很直观的。就像工厂模式一样，唯一不同的是我们在injector中做了所有事情，而不会去为每种类型去建一个工厂。并且大部分时候InjectionBinder是完全不可见的。我们已经习惯了使用如下形式来构造一个实例：

```
IClass myInstance = new MyClass();
```

　　…所以要好好整理一下思路，让我再一次重申，大多数情况下，你们不需要使用这个方法，因为实例是通过注入得来的。只要你听了我马上要告诉你的，你就不会总想着瞎写new myclass( )了

```
IClass myInstance = injectionBinder.GetInstance<IClass>() as IClass;
```

　　正如你看到的，我们还是摆脱了实体类的专制。而且你拿到的实例附带了它本身所需的全部依赖。我们要做的跟以前的老方法比仅仅是略有不同。

注入映射（Injection mapping）的种类
　　我们可以用多种方法来绑定注入，这些方法都很有用，其中一个很有用的绑定是ToSingleton（单例）。它的形式如下：

```
injectionBinder.Bind<ISocialService>().To<TwitterService>().ToSingleton();
```

　　单例模式可能你已经了解了。它是指在整个应用中只有一个实例会被创建。如果你使用了这种模式，你可能会见到如下代码：

```
ISocialService socialService = TwitterService.Get();
```

　　单例模式也有一些问题，最突出的就是有时它们并不是唯一的。例如，上面的代码，可能会造成本来只有一个ISocialService（Twitter），而由于设计上的改变，某一天可能会变成3个（Twitter，Facebook，G+）。写出TwitterService.Get()代码的人不仅具体的依赖了TwitterService，他还显式的认为这是一个单例。如果这一点变了，他不得不重构代码。
　　与上面使用单例来Get的代码相比：

```
[Inject]
public ISocialService {get;set;}
```

　　等等，这个不可能是对的。它看起来跟之前我们看到的injection 标记一模一样。是的，这就是我要说的点，你的类并不需要TwitterService，它需要的是ISocialService。而且它并不会关心这个接口的实例是不是单例。
　　由于Strange的依赖只是一个映射，所以更改单例的映射成了很繁琐的事情，客户端不仅不知道ISocialService用的是哪一个实例，而且不知道这些实例是不是单例。应该是这样的，一旦你使用了依赖注入，你就不需要再写单例了，而是映射单例（map Singletons）。
　　但是在如下例子中，我们不仅仅是改变services，我们会增加多个services。那我们怎么分辨它们呢？这就引出了第二种映射方法：名字注入（named injections）

```
injectionBinder.Bind<ISocialService>()
	.To<TwitterService>().ToSingleton()
	.ToName(ServiceTypes.PRIMARY);
    
injectionBinder.Bind<ISocialService>()
	.To<TwitterService>().ToSingleton()
	.ToName(ServiceTypes.SECONDARY);

injectionBinder.Bind<ISocialService>()
	.To<TwitterService>().ToSingleton()
	.ToName(ServiceTypes.TERTIARY);
```

　　名字注入与其他的注入方法稍有区别，注入器（injector）可以通过名称来分辨满足同一接口的不同类。使用这种方法，你可以在不同的地方注入ISocialService并且拿到满意的版本。使用类（client class）需要在标签中添加一个名字来匹配（`[Inject (Name)]`）。

```
[Inject (ServiceTypes.TERTIARY)] //We mapped TwitterService to TERTIARY
public ISocialService socialService{get;set;}
```

　　名字可以是任意类型，但是实践中使用枚举是一个不错的选择。注意，类中的这个带名字的标签创建了另一种依赖（我们毕竟声明了实用类需要一个比某个接口更加具体的东西：不仅满足接口，还要满足名字），所以我们要小心使用这个功能。
　　有时，你清楚的知道你想要注入的是什么。或许你加载了一个配置表，它会用在应用的各处。这个可以用值映射(value mapping)来实现.

```
Configuration myConfig = loadConfiguration();
injectionBinder.Bind<IConfig>().ToValue(myConfig);
```

　　这个例子里，myConfig是某个配置文件的加载结果，现在，不管你在哪里需要Iconfig，你将会得到myConfig实例。再次申明，使用类并不知道它是否是一个单例，值或者其它类型。它只需使用Iconfig即可，不需要关心它的值从哪来。
　　有时你也可能遇到你没有某个类的修改权限。也许这个类来自外部库，已经被写成了单例。这时你仍可以使用值映射来实现注入，只需要获取实例，映射实例即可：

```
TouchCommander instance = TouchCommander.Get();
injectionBinder.Bind<TouchCommander>().ToValue(instance);
```

　　如果TouchCommander实现了某个接口，绑定到接口肯定是个更好的选择。或者（我自己经常这么做）你可以创建一个接口将TouchCommander包裹在facade（外观类）里面。毕竟，某天你可能将TouchCommander换成另外一个触摸事件处理系统。如果你真的这么做了，而你的应用中遍布TouchCommander的引用，那么你不得不面对大量的重构。一个实现了你所选接口的facade类可以帮你避免这个问题，并且可以牢牢的控制对于TouchCommander实例的引用。
　　那现在如果你每次都想要一个新的实例呢？我们利用工厂映射（factory mapping）来实现这个要求：

```
injectionBinder.Bind<IEnemy>().To<Borg>();
```

　　这与单例映射（ToSingleton mapping）基本相同，只是没有了ToSingleton指令。当这个注入发生时，你将得到一个新的IEnemy，在这个例子中，它映射到了一个实体类Borg上。注意我们可以结合这些映射，比如工厂映射也可以使用名字：

```
injectionBinder.Bind<IEnemy>().To<Borg>().ToName(EnemyType.ADVANCED);
injectionBinder.Bind<IEnemy>().To<Romulan>().ToName(EnemyType.BASIC); 
```

　　你还可以多重绑定，这样允许绑定的多态性，这是对于一个类可以实现多个接口的另类说法：

```
injectionBinder.Bind<IHittable>().Bind<IUpdateable>().To<Romulan>();
```

　　这样不管[inject]标签标记了IHittable还是IUpdateable都允许你获得一个enemy（敌人）。注意，多个“Bind”是有意义的，但是多个“To”却毫无意义。你可以绑定多个接口，但是只有绑定到单个实体类型（concrete type），或者值的时候才会生效。·

可注入类中的一些操作（Some thing you can do with Injectable Classes）
　　在前文我层提过如何在类中声明setter注入。在这里我再重申一下，如果想要一个属性是可注入的，使用[Inject]标签：

```
[Inject]
public ICompensator compensator{get;set;}
```

　　或者给它加上名字：

```
[Inject(CompensatorTypes.HEISENBERG)]
public ICompensator compensator{get;set;}
```

　　或者用一个maker类来标记它：

```
[Inject(typeof(HeisenbergMarker))]
public ICompensator compensator{get;set;}
```

　　这些都是setter注入的例子，它只是Strange两种注入方式的一种。还有一种方式是构造器注入（constructor Injection），这种注入方式是在类的构造方法中完成的。Setter注入有两个显著的缺点，首先可注入的属性需要是public类型才能被注入。这可能并不是你最初的选择（如果不使用注入你可能希望它是private）。利用构造器注入你可以保持private类型是private的。其次如果你使用setter注入，你必须小心的编写你的构造函数。定义上来看，构造函数会在setter被赋值前运行。所以，注入的属性在构造函数中是不能使用的。而由于构造器注入是以构造函数参数的形式来提供所需依赖，所以注入值可以立即使用。

| Type of Injection | Advantages                               | Disadvantages                            |
| ----------------- | ---------------------------------------- | ---------------------------------------- |
| Setter            | 1. 允许名字注入                                                                                                     2. 更少的代码                                                     3. 更加灵活 | 1. 注入的依赖无法在构造函数中使用                                                                           2. 一些private的属性不得不变为public |
| Constructor       | 1. private属性可以保持private                       2. 注入的依赖可以在构造函数中使用 | 1. 不允许名字注入                                                     2. 更多的代码                                                         3. 灵活性较差 |

　　除了[Inject]标签，你应该了解更多的状态标签（attributes）。
　　如果你的类有多个构造函数，[Construct]标签可以用来标记哪一个会被Strange用到。如果没有构造函数被[Construct]标记，Strange会用参数最少的那个构造函数。当然，如果你只有一个构造函数，你完全不需要使用[Construct]标签。

```
public Spaceship()
{
	//This constructor gets called by default...
}
  
[Construct]
public Spaceship(IWeapon weapon)
{
	//...but this one is marked, so Strange will call it instead
} 

```

　　如果你使用setter注入，[PostConstruct]会是很有用的一个标签。任何标记了[PostConstruct]的方法，会在注入发生以后立即被调用。在这个方法中使用注入依赖不会产生空指针。

```
[PostConstruct]
public void PostConstruct()
{
	//Do stuff you’d normally do in a constructor
}

```

　　可以声明多个带有[PostConstruct]标签的函数，而且它们可以被排序。

```
[PostConstruct(1)]
public void PostConstructOne()
{
	//This fires first
}

[PostConstruct(2)]
public void PostConstructTwo()
{
	//This fires second
}

```

　　到底要用setter注入还是构造器注入呢？详细请看[这里](http://shaun.boyblack.co.za/blog/2009/05/01/constructor-injection-vs-setter-injection/)。

注意：
　　下面几个点在使用注入的时候要格外注意：
1、 注意依赖回路。如果类之间相互注入，这将会导致一个依赖的死循环。Strange 会抛出一个InjectionException 异常来警告你，但是你应该第一时间避免这类情况。
2、依赖注入使用了反射，前文说过，它的执行比较慢。Strange使用ReflectionBinder来最小化这个问题（并且效果显著），但是要考虑这个方法对于性能要求很高的代码是否可行，比如说你游戏的主循环。
3、注意，如果你使用了注入，你必须要进行映射。创建依赖但是忘记填充它会造成空指针错误。幸运的是Strange会替你关注这些问题，并且帮你找出缺少的映射和需要映射的地方。

### 反射器扩展（The reflector extension）

　　事实上，你并不需要十分了解这个扩展，但它确实存在，并且处理注入过程中的反射。反射是在运行时对类的分析过程。Strange使用这个技术来决定将要注入什么。
　　值得注意的是反射器扩展是在开发后期才编写的，它是为了优化反射执行缓慢的问题。当时我觉得如果我把反射结果做缓存应该可以提高反射的性能，于是我写了ReflectionBinder来做这件事。在这之前，每个类在每次实例化的时候都会进行反射。现在每个类只进行一次反射。使用1000个中等复杂的实例来测试，结果有5倍的性能提升。这是一个极好的，扩展核心绑定器来解决问题的例子。）
　　有一个值得你注意的特性是它可以“预反射”类。也就是说，你可以在当前性能压力比较小的时候来触发消耗比较大的反射过程（比如，当玩家在看一些静态UI的时候）。这个可以通过injectionBinder实现。
　　第一个例子展示了怎样反射一个类的链表：

```
List<Type> list = new List<Type> ();
list.Add (typeof(Borg));
list.Add (typeof(DeathStar));
list.Add (typeof(Galactus));
list.Add (typeof(Berserker));
//count should equal 4, verifying that all four classes were reflected.
int count = injectionBinder.Reflect (list);

```

　　第二个例子简单的反射了所有映射到injectionBInder的所有东西；

```
injectionBinder.ReflectAll();

```

### 分发器扩展（The dispatcher extension）

　　注：EventDispatcher是Strange默认的分发系统。现在我们有了Signal扩展，它在分发中加入了类型安全。我们推荐这个新的系统，但是计划对这两种系统都支持。使用哪一个由你来决定。
　　原理上来说，分发器的功能类似经典观察者模式中的观察对象。也就是说，它允许被某些类监听，并且在特定事件发生时，通知这些类。在Strange中，我们实现了EventDispatcher，它会将触发器（可以是任何类型，但是一般用String或者枚举）与有单个参数或者无参函数绑定，当触发器触发时会调用所绑定函数。结果参数（如果需要）会以IEvent的形式提供，IEvent是一个包含了事件相关数据的值对象（你可以自己实现IEvent接口，Strange标准的实现叫做TmEvent）。
　　如果你在使用MVCSContext版的Strange，默认将会有个全局的事件分发器（叫做‘contextDispatcher’），它自动被注入到了应用的各个地方，你可以用它来发送消息。还有一个crossContextDispatcher可以用来在不同的Context间发送消息。
　　使用EventDispatcher可以实现两个基本功能：发送事件和监听事件。也就是说，只有少数几种方法可以配置事件怎样来被发送和接收。我们先从最简单的监听形式来开始。

```
dispatcher.AddListener("FIRE_MISSILE", onMissileFire);

```

　　这将监听分发器，当“FIRE_MISSILE”事件被触发时，onMissileFire函数将会被调用。
　　让我们先入为主的认为，简单的东西不一定好。使用String会使得代码不好维护。String可以在其它相关代码不可知的情况下被修改，这个可能造成灾难性后果。更好的形式应该是常量。。。或者是枚举：

```
dispatcher.AddListener(AttackEvent.FIRE_MISSILE, onMissileFire);

```

　　你可以像下面这样移除监听：

```
dispatcher.RemoveListener(AttackEvent.FIRE_MISSILE, onMissileFire);

```

　　在底层，AddListener和RemoveListener其实跟Bind与Unbind是同义词。AddListener/RemoveListener对只是提供大多数人熟悉的语法糖。还有一个比较方便的方法可以利用boolean来更新监听器：

```
dispatcher.UpdateListener(true, AttackEvent.FIRE_MISSILE, onMissileFire);

```

　　被调用的方法可以有一个参数或者没有参数，取决于你是否关心事件所带的信息：

```
private void onMissileFire()
{
	//this works...
}

private void onMissileFire(IEvent evt)
{
	//...and so does this.
	Vector3 direction = evt.data as Vector3;
}

```

　　你也会想要发送事件。这是一种表达“看这里！我在做一件很酷的事情！”的方法，这里也有几中方法来做这件事。同样，我们从简单的开始：

```
dispatcher.Dispatch(AttackEvent.FIRE_MISSILE);

```

　　这种形式的发送会生成一个新的TmEvent并且调用所有的监听者，但是由于你并没有提供数据，TmEvent的数据将会是null。你也可以发送事件并提供数据：

```
Vector3 orientation = gameObject.transform.localRotation.eulerAngles;
dispatcher.Dispatch(AttackEvent.FIRE_MISSILE, orientation);

```

　　现在生成的TmEvent将会携带与orientation匹配的Vector3数据。
　　最后，你还可以显式的构建TmEvent并将它发送:

```
TmEvent evt = new TmEvent(AttackEvent.FIRE_MISSILE, dispatcher, this.orientation);
dispatcher.Dispatch(evt);

```

　　使用哪种方法取决于编码风格。每种方法对于监听器来说其实是一样的。

 

### 命令扩展（The command extension）

　　除了将事件与方法绑定，你也可以绑定事件与命令（Command）。Command是经典Mode-View-Controller-Service架构中的Controller（控制器）。在MVCSContext版本的Strange中，CommandBinder监听着分发器所有的事件（当然在你自己的Context中你可以改变这一点）。下文中将要讲到的信号（Signals），也可以与命令绑定。当触发一个信号或者事件时，CommandBinder 会查看是否有Command与信号或事件绑定。如果找到绑定，一个新的Command实例会被创建。这个Command会被注入，执行，然后销毁。我们先来看一个简单的Command：

```
using strange.extensions.command.impl;
using com.example.spacebattle.utils;

namespace com.example.spacebattle.controller
{
	class StartGameCommand : EventCommand
	{
		[Inject]
		public ITimer gameTimer{get;set;}

		override public void Execute()
		{
			gameTimer.start();
			dispatcher.dispatch(GameEvent.STARTED);
		}
	}
}

```

　　这个例子中有几个值得注意的地方。首先，注意我们继承了EventCommand，而且我们使用了strange.extensions.command.impl命名空间。继承EventCommand甚至是Command并不是必须的，但是你必须实现ICommand接口。其次，注意你可以在Command中使用注入。这一点非常有用，因为它意味着在Command中可以与model或者service进行交互。最后要注意通过继承EventCommand我们可以使用dispatcher（EventDispatcher被注入到了Context的各个地方 ），所以contextDispatcher的所有监听者，无论在应用的哪个地方，都可以接收到我们发出的GameEvent.STARTED事件。由于这是一个同步命令，我们只需触发事件即可。一旦Execute()执行完毕，这个命令将被清除。
　　但是，对于异步命令，比如调用网络服务该怎样处理呢？我们可以使用Retain()和Release()这一对方法来处理，请看这个例子：

```
using strange.extensions.command.impl;
using com.example.spacebattle.service;

namespace com.example.spacebattle.controller
{
	class PostScoreCommand : EventCommand
	{
		[Inject]
		IServer gameServer{get;set;}
        
		override public void Execute()
		{
			Retain();
			int score = (int)evt.data;
			gameServer.dispatcher.AddListener(ServerEvent.SUCCESS, onSuccess);
			gameServer.dispatcher.AddListener(ServerEvent.FAILURE, onFailure);
			gameServer.send(score);
		}

		private void onSuccess()
		{
			gameServer.dispatcher.RemoveListener(ServerEvent.SUCCESS, onSuccess);
			gameServer.dispatcher.RemoveListener(ServerEvent.FAILURE, onFailure);
			//...do something to report success...
			Release();
		}

		private void onFailure(object payload)
		{
			gameServer.dispatcher.RemoveListener(ServerEvent.SUCCESS, onSuccess);
			gameServer.dispatcher.RemoveListener(
			ServerEvent.FAILURE, onFailure);
			//...do something to report failure...
			Release();
		}
	}
}

```

　　你应该差不多会理解上面的例子了。我们给gameServer发送了SendScore请求，这个请求需要处理一段时间。这个命令需要等待服务器返回结果。通过在Execute方法中第一行调用Retain()，我们将这个命令保存在了内存中。当你调用了Retain()，无论回调函数的结果是怎样，记得还要调用ReLease()，否则会造成内存泄漏。

#### 映射命令（Mapping commands）

　　虽然我们可以在任何地方映射命令，但是我们通常在Context中做这件事。这样当你需要时可以更方便的找出映射关系。命令映射看起来像一个注入映射：

```
commandBinder.Bind(ServerEvent.POST_SCORE).To<PostScoreCommand>();

```

　　你可以绑定多个命令到单个信号：

```
commandBinder.Bind(GameEvent.HIT).To<DestroyEnemyCommand>().To<UpdateScoreCommand>();

```

　　你也可以随时取消绑定：

```
commandBinder.Unbind(ServerEvent.POST_SCORE);

```

　　如果你想要一个命令只触发一次，有一个表示“一次性”的指令可以很方便的满足这个需求：

```
commandBinder.Bind(GameEvent.HIT).To<DestroyEnemyCommand>().Once();

```

　　通过Once()方法，你可以确保命令触发一次以后绑定关系会自动销毁。
　　序列（Sequences）是指一系列的命令按照顺序执行。命令一个接一个的被触发，直到队列结束，或者其中一个命令执行失败。命令可以调用Fail()函数来打破序列。如果要创建几个链形依赖的事件，比如建立有序动画，或者设立一个守卫类来判断命令是否需要执行。
　　映射一个序列只需要加一个InSequence()指令：

```
commandBinder.Bind(GameEvent.HIT).InSequence()
	.To<CheckLevelClearedCommand>()
	.To<EndLevelCommand>()
	.To<GameOverCommand>();

```

　　上面这个序列代表的意义是，一次点击可能表示已经通过了一个关卡。所以我们运行CheckLevelClearedCommand（检查是否已通关）。如果通关，我们运行EndLevelCommand（结束关卡）。如果这这个命令表示我们已经打到最后一关，那么我们运行GameOverCommand。在这个过程中，我们可以调用Fail()来停止这个流程。
　　如同常规的命令一样，序列中的命令可能是异步执行的。如果是这种情况（假设Fail()没有被调用），后面的命令将会在Release()调用后被执行。

### 信号扩展（The signal extension）

　　信号是一个分发机制，它可以代替EventDispatcher，在Strange v.0.6.0中被引入。EventDispatcher创建并且发送 IEvent对象并使用其中的一个data属性来传递信息，而信号通过注册到回调，可以传递0到4个强类型的参数。这样做相比EventDispatcher有两个主要优势。第一，信号的发送不会产生新的对象，所以不需要对很多新的实例进行GC（垃圾回收）。更重要的是第二点，信号发送是类型安全的，如果信号与它注册的回调不匹配，在编译时就会报错。
　　另外一个重要的不同点是每个context里有一个“全局“的EventDispatcher（还有“更加全局”的CrossContextDispatcher）触发事件，信号使用了一个不同的模式。每个”事件“都是一个针对某一任务的信号个体产生的结果。所以EventDispatcher是完全统一的，而信号可能会有多种。让我们来看一些例子。
　　这里是两个信号，每个有一个参数：

```
Signal<int> signalDispatchesInt = new Signal<int>();
Signal<string> signalDispatchesString = new Signal<string>();

```

　　注意看每个信号的发送类型是怎样在实例化中确立的。我们用几个回调来延伸一下上面的例子：

```
Signal<int> signalDispatchesInt = new Signal<int>();
Signal<string> signalDispatchesString = new Signal<string>();

signalDispatchesInt.AddListener(callbackInt);		//Add a callback with an int parameter
signalDispatchesString.AddListener(callbackString);	//Add a callback with a string parameter

signalDispatchesInt.Dispatch(42);			//dispatch an int
signalDispatchesString.Dispatch("Ender Wiggin");	//dispatch a string

void callbackInt(int value)
{
	//Do something with this int
}

void callback(string value)
{
	//Do something with this string
}

```

　　值得注意的是信号是强类型的，信号与侦听者类型符合是编译要求。也就是说应用会编译失败，如果你写了下面的代码：

```
Signal<int> signalDispatchesInt = new Signal<int>();
Signal<string> signalDispatchesString = new Signal<string>();

signalDispatchesInt.AddListener(callbackString); //Oops! I attached the wrong callback to my Signal!
signalDispatchesString.AddListener(callbackInt); //Oops! I did it again! (Am I klutzy or what?!)

```

　　这样，很难有一些运行时的类型错误。（作者原话：This makes screwingup your listeners pretty darned difficult）信号的参数是类型安全并且向下兼容的。这表示对于参数类型可赋值即可构成合法的

```
//You can do this...
Signal<SuperClass> signal = new Signal<SuperClass>();
signal.Dispatch(instanceOfASubclass);

//...but never this
Signal<SubClass> signal = new Signal<SubClass>();
signal.Dispatch(instanceOfASuperclass);

```

　　你可以创建带有0-4个参数的信号，信号使用Action类作为底层机制来实现类型安全。Unity的C#实现每个Action允许最大4个参数，所以我们只提供这么多。如果你想要4个以上参数，考虑创建一个值对象，并且发送这个对象。

```
//works
Signal signal0 = new Signal();

//works
Signal<SomeValueObject> signal1 = new Signal<SomeValueObject>();

//works
Signal<int, string> signal2 = new Signal<int, string>();

//works
Signal<int, int, int> signal3 = new Signal<int, int, int>();

//works
Signal<SomeValueObject, int, string, MonoBehaviour> signal4 = new Signal<SomeValueObject, int, string, MonoBehaviour>();

//FAILS!!!! Too many params.
Signal<int, string, float, Vector2, Rect> signal5 = new Signal<int, string, float, Vector2, Rect>(); 

```

　　当然，如果不想使用上述例子中的声明方法你还可以自建信号的子类。这个在Strange中很有用，特别是当你想要一些简单的，易读的跨Context信号的名称时。这是一个信号子类的例子;

```
using System;
    using UnityEngine;
    using strange.extensions.signal.impl;

    namespace mynamespace
    {
        //We're typing this Signal's payloads to MonoBehaviour and int
        public class ShipDestroyedSignal : Signal<MonoBehaviour, int>
        {
        }
    }
     

```

将信号映射到命令 
　　如果你想要在你的Context中将Signal绑定到Commands（很好的想法）你需要做一些小变动。要是想要全部的Signals体验，加入如下代码到你的Context中：

```
protected override void addCoreComponents()
{
	base.addCoreComponents();
	injectionBinder.Unbind<ICommandBinder>();
	injectionBinder.Bind<ICommandBinder>().To<SignalCommandBinder>().ToSingleton();
}

```

　　这段代码告诉Strange我们不再使用默认的CommandBinder，而是用SignalCommandBinder来代替它。这样，Signals会触发Commands而不是Events。注意Strange目前支持Events或者Signals映射到Commands但是二者不可以混用。
　　加入上面代码后，Signals也可以像Events一样被映射到Commands。基本语法如下：

```
commandBinder.Bind<SomeSignal>().To<SomeCommand>();

```

　　注意，这仍然是commandBinder。我们只是去掉了原来处理EventDispatcher的映射，然后加入了处理Signals的映射。当然所有Command-mapping的一些特性都支持，包括多命令（multipl ecommands），序列和Once()映射。
　　将一个信号映射到命令会自动生成一个注入映射，你可以使用[Inject]标签来获取它，示例如下：

```
[Inject]
public ShipDestroyedSignal shipDestroyedSignal{get; set;}

```

　　你可以在任何需要的地方使用这个注入（当然最好要符合一定的规范），包括在Commands（命令）中或者是Mediators中。
　　为了弄清楚Signal/Command映射是如何工作的，我们简单的过一个例子，例子中我们将上述ShipDestorySignal 信号映射到一个Command，我们将从在Context中绑定信号开始：

```
commandBinder.Bind<ShipDestroyedSignal>().To<ShipDestroyedCommand>();

```

　　在ShipMediator类中，我们注入这个信号，然后将它发送：

```
[Inject]
public ShipDestroyedSignal shipDestroyedSignal{get; set;}

private int basePointValue; //imagining that the Mediator holds a value for this ship

//Something happened that resulted in destruction
private void OnShipDestroyed()
{
	shipDestroyedSignal.Dispatch(view, basePointValue);
}

```

　　发送一个通过SignalCommandBinder映射的信号会产生一个ShipDestoryedCommand实例：

```
using System;
using strange.extensions.command.impl;
using UnityEngine;

namespace mynamespace
{
	//Note how we extend Command, not EventCommand
	public class ShipDestroyedCommand : Command
	{
		[Inject]
		public MonoBehaviour view{ get; set;}

		[Inject]
		public int basePointValue{ get; set;}

		public override void Execute ()
		{
			//Do unspeakable things to the destroyed ship
		}
	}
}　

```

　　你可以看到，映射信号到命令的方法与之前使用事件的方法比较相似。
　　有两个重要警告：第一，Signals支持相同类型的多个参数，但是Injections（注入）不支持。所以映射到Command(命令)的信号不允许有多个相同类型的参数。

```
//This works
Signal<int, int> twoIntSignal = new Signal<int, int>();
twoIntSignal.AddListener(twoIntCallback);

//This fails
Signal<int, int> twoIntSignal = new Signal<int, int>();
commandBinder.Bind(twoIntSignal).To<SomeCommand>();

```

　　再次提醒，你可以使用值对象来突破这个限制
　　第二个警告：Strange有一个方便的，内置的START事件来驱动程序开始。解绑EventDispatcher会把这个事件关闭。因此推荐的解决方法是使用自定义的StartSignal来重写你Context中的Launch方法，像如下一样：

```
override public void Launch()
{
	base.Launch();
	//Make sure you've mapped this to a StartCommand!
	StartSignal startSignal= (StartSignal)injectionBinder.GetInstance<StartSignal>();
	startSignal.Dispatch();
}

```

不需要Command来映射信号
　　上面提到，将Signal映射到Command会自动创建一个映射，通过注入的方式，它可以在任何地方获得，但是要是你只想要一个没有绑定Command的Signal呢？ 这种情况下，只需像如下一样用injectionBinder来创建映射，跟其他注入的类一样：

```
injectionBinder.Bind<ShipDestroyedSignal>().ToSingleton();

```

### 中继扩展(The mediation extension)

　　MediationContext 是Strange专为Unity3D编写的部分。这是因为mediation目的是为了细致的控制你的View(视图)与程序的其它部分交互。在开发过程中View 本身就是常常变动的，把这种天然的杂乱留在view类里面处理是极其明智的选择。因此，我们建议你的view包含两个不同的MonoBehaviours：View和Mediator。

View
　　View 类代表着MVCS结构这种的V，一个View就是一个MonoBehaviour，你可以编写它来控制视觉（和听觉）的输入和输出。这个类可以在Unity编辑器中挂在相关的GameObject上。如果它有公有组建，它们可以像其它正常脚本一样在编辑器中调整（Inspector上）。想要绿色的按钮吗？在View中实现吧。想要绿色按钮上面有数字吗？在View中实现吧。想要注入一个model或者service吗？等等，不要这么做！为什么呢？
　　你的View可以被注入，但是将你的View直接与models和services绑定是非常不好的习惯。正如我说过的，你的View代码可能会变得混乱，保护其它类不受这些混乱影响是值得的。下一章我们将会带出我们认为的使用Strange开发程序的最佳架构，但是现在我们先来考虑一下你的View应该只负责做如下的事情：
1、编写可视组建。
2、当用户与这些组建发生交互时发送事件。
3、暴露接口，允许其它角色修改这些组建的视觉状态。
　　通过用这三条来限制你自己，通过把所有的逻辑或者状态都放在View之外，通过拒绝将models和services注入到View里，你包装了View并且让开发更加简单，在长远来看好处会越来越突出。在这个上面请相信我。
　　现在，在上面的第三条中我提到要暴露接口给其它角色。谁将是这个角色呢？

Mediator
　　Mediator类是一个单独的MonoBehaviour，它的工作是既要了解View又要了解整个程序。它是一个thin class（瘦类），也就是说它的职责会很专精。它对View有亲密的了解，而且它可以注入，并且拥有发送和接收信号的权限。所以想想绿色按钮上面的数字。你本来打算将一个service注入到View中来展示，比如说，在线好友的数量。现在你可以将service注入到mediator，但是既然Mediator应该是瘦的，更好的答案是发送一个请求，让Command来处理Service调用，然后发送一个响应。这样做代价是绕了很多弯路，但是得到是清楚的代码结构。
　　这个是一个Mediator的示例：

```
using Strange.extensions.mediation.impl;
using com.example.spacebattle.events;
using com.example.spacebattle.model;
namespace com.example.spacebattle.view
{
	class DashboardMediator : EventMediator
	{
		[Inject]
		public DashboardView view{get;set;}

		override public void OnRegister()
		{
			view.init();
			dispatcher.AddListener
				(ServiceEvent.FULFILL_ONLINE_PLAYERS, onPlayers);
			dispatcher.Dispatch
				(ServiceEvent.REQUEST_ONLINE_PLAYERS);
		}
		
		override public void OnRemove()
		{
			dispatcher.RemoveListener
				(ServiceEvent.FULFILL_ONLINE_PLAYERS, onPlayers);
		}

		private void onPlayers(IEvent evt)
		{
			IPlayers[] playerList = evt.data as IPlayers[];
			view.updatePlayerCount(playerList.Length);
		}
	}
}

```

　　需要注意的点：
1、DashBoardView的注入说明了Mediator获得它的View的途径。
2、OnRegister() 是注入之后立即调用的方法。它就像构造函数一样，你可以用它初始化view和其它一些的初始进程，包括——上面的例子中使用的——请求重要数据。
3、ContextDispatcher会在每个继承EventMediator的Mediator中注入，所以你可以向整个context中发送事件。
4、OnRemove()是为了清除和收尾的；它会在View被销毁的时候调用。记得在这里去除添加过的listener。
5、这个例子中的View暴露了两个方法作为接口。在实际情况中，你很可能需要更大的接口，但是原则是不变的：Mediator只被用来在View和程序的其它模块传递信息。
　　将View绑定到Mediator对你来说应该很熟悉了：

```
mediationBinder.Bind<DashboardView>().To<DashboardMediator>();

```

另外两点备注：
　　并不是所有的MonoBehaviour都有资格作为View。这里有幕后的魔法来让Stange知道某个View的存在。所以要么是继承View类，要么是复制那个魔法在你的代码中（只有几行），或者自己建立一个view继承于View类，然后你的其它类继承于自建的view。最后的那个模式很有用，因为你可能想要插入一些debugging代码可以被所有的View调用。
　　Mediation绑定是实例对实例的。一个新的View被创建时一个新的Mediator也会被创建来支持它。所以很多View意味着很多Mediator。

### 环境扩展（The context extension）

　　Context包把所有的各种不同的Binder(绑定器)包装在一起。比如说，MVCSContext包含一个EventDispatcher，一个InjectionBinder，一个MediationBinder和一个CommandBinder。你也可以，像我们讨论过的，把CommandBinder重映射到SignalCommandBinder。(Signal)CommandBinder 监听着EventDispatcher（或者Signal）。Commands和Mediators依靠Injection。Context就是我们将这些依赖联系起来的地方。建立项目时，你将要重写Context或者MVCSContext，所有使得程序正常运行的绑定将被写在在它们的子类中。
　　多个Context也是可行的。这将是你的程序更加模块化。独立的模块可以自己运行，仅仅在需要时才与别的模块交互。因此核心游戏可以写成一个app，社交模块可以分开编写，聊天app可以作为另一个app，这个三个app在后面的开发中可以被绑定在一起，每个只需共享其它两个需要的部分。

## 3、MVCSContext：整体情况

　　这一章节基本上是对于如何使用Strange和它的MVCSContext创建一个app。前面几个章节，我描述了所有的组件，在这里，我将解释如何将它们结合到一起。
　　如果要写一个游戏，使用Strange的话，应当从哪里入手呢？ 让我们从最基本的地方开始。

### 概念：

　　Strange 使用MVCSContext 将整个微框架（micro-architecture）包装成一个方便，易用的包。正如它的名字所体现的，它被设计成使用MVCS模式运行的应用。（S 是Service，代表所有你的应用之外的东西，比如web service）。
　　以下是所有将要被组装的组件：
1、你的应用的入口是一个叫做ContextView的类，它只是一个简单的MonoBehaviour 用来初始化MVCSContext。
2、MVCSContext（技术上来说，这里应该是一个MVCSContext的子类）是设置所有绑定的地方。
3、The dispatcher（分发器）是通信总线，允许你在整个app中发送信息。MVCSContext中dispatcher发送的对象叫做TmEvent。你还可以重写Context来使用信号。
4、Commands 是可以被IEvent 或者 Signal触发的类。当一个Command被执行时，它承担了部分应用逻辑。
5、Models 存储状态。
6、Service 与外部程序通信。
7、View是附加在Gameobject 上面的MonoBehaviour组件：这部分是玩家实际看到和与之交互的部分。
8、Mediator 也是MonoBehaviour，但是它的功能是分离View和程序的其它部分。
　　下图表示了这些部分是怎样结合在一起的：

![](http://strangeioc.github.io/strangeioc/class-flow.png)

### 建立你自己的项目

　　下载Stange的目录。在里面你将找到一个完整的Unity项目实例（我建议你看看这个）。打开项目中的TestView.unity文件。
　　所有使用Strange所需的东西都在以下目录：

```
Assets/scripts/strange

```

　　其中有两个重要的子目录分别是 framework 和 extensions

### 场景设置：

　　当你在Unity中打开场景时，你会找到一个叫做“ViewGO”的Gameobject它的下面还包含一个camera。ContextView最好是游戏层级的最上层，其它内容都在它的下面。虽然Unity并不要求一个单独的顶层GameObject，但是使用Strange的话这样是最好的，尤其是有多个context的时候，因为Strange用展示的层级来决定某个View是属于哪个Context的。GameObject上还有一个叫做“MyFistProjectRoot”的MonoBehaviour脚本。
　　运行一下场景，当你点击旋转的文字时候观察会发生什么。没什么特别的，我们现在只是展示一下结构。
　　在Inspector视图中，双击一下MyfirstProjectRoot，打开脚本编辑器。

### 开始了解ContextView…

　　ContextView 是一个MonoBehaviour脚本用来实例化你的Context。MyFirstProjectRoot 继承了 ContextView类，并且是我们程序的起点。

```
using System;
using UnityEngine;
using strange.extensions.context.impl;
 
namespace strange.examples.myfirstproject
{
    public class MyFirstProjectRoot : ContextView
    {
        void Awake()
        {
            context = new MyFirstContext(this);
        }
    }
}

```

　　注意，我们使用了strange.extensions.context.impl 命名空间。所有Strange使用的命名空间都是这样的结构，所以你只引入你需要的。
　　其余的代码很简单。ContextView定义了一个叫做context的属性来引用我们的内容（context）。我们只需要将它定义就好。我们已经写了MyFirstContext类。This参数是MyFirstProjectRoot的引用。它告诉了Context哪一个GameObject被认为是ContextView。

### Context bind

　　正如我在上面解释过的，Context是绑定操作进行的地方。如果没有绑定，Strange 应用只不过是一堆杂乱的组件。Context是让混乱恢复秩序的胶水。由于我们继承了MVCSContext，我们也拥有了一系列绑定的优点。MVCSContext被设计用来使得IoC-style的程序有着清晰的结构。IoC-style的程序包含一个注入器，一个消息总线，命令模式，model和service的支持，还有UI的中继。下面是一个简单的Context的代码。

```
using System;
using UnityEngine;
using strange.extensions.context.api;
using strange.extensions.context.impl;
using strange.extensions.dispatcher.eventdispatcher.api;
using strange.extensions.dispatcher.eventdispatcher.impl;
 
namespace strange.examples.myfirstproject
{
    public class MyFirstContext : MVCSContext
    {
 
        public MyFirstContext () : base()
        {
        }
        
        public MyFirstContext (MonoBehaviour view, bool autoStartup) : base(view, autoStartup)
        {
        }
        
        protected override void mapBindings()
        {
            injectionBinder.Bind<IExampleModel>()
                .To<ExampleModel>()
                .ToSingleton();
            injectionBinder.Bind<IExampleService>()
                .To<ExampleService>()
                .ToSingleton();
 
            mediationBinder.Bind<ExampleView>()
                .To<ExampleMediator>();
 
            commandBinder.Bind(ExampleEvent.REQUEST_WEB_SERVICE)
                .To<CallWebServiceCommand>();
            commandBinder.Bind(ContextEvent.START)
                .To<StartCommand>().Once ();
 
        }
    }
}

```

　　如你所见，我们继承了MVCSContext,也就是说我们继承了它所有的映射关系（你会发现对它更深层次的探索会很有趣）。所以我们已经有了injectionBinder和commandBinder以及dispatcher。注意dispatcher可以在应用的任意地方取得，而且与commandBinder是耦合的，所以任何一个发出的事件既可以触发回调，也可以触发commands和sequences。
　　若你已经看过了各个组件的介绍，上面的映射关系正是你所预期的。对注入来说，我们映射了一个model和一个service，它们都是单例。在这个例子中我们将会只有一个view(ExampleView)，我们将它绑定到一个Mediator(ExampleMediator)。最后我们映射了两个commands。其中更重要一个的是StartCommand。它与一个特殊的事件：ContextEvent.START相绑定。这个事件会触发程序的开始。你可以绑定一些command或者sequence在它后面，然后将它们当作你的应用的初始化。而且可以看到我们用了Once()来约束它，这个方法会在事件触发一次之后解绑该事件。
　　注意还有一个postBinding()方法。如果你想在绑定之后而在程序启动（Launch()方法调用）之前运行一些代码，它是非常有用的。MVCSContext用它来处理那些较早(在mapBindings()运行之前)注册的View。另一个明显并且有用的例子是当你切换场景时在postBinding()中调用DontDestoryOnLoad(contextView)，来保留contextView(和Context)。

### 当Command被触发

　　当ContextEvent.START事件触发时，因为它与StartCommand绑定，一个新的StartCommand实例将会被实创建并且执行

```
using System; 
using UnityEngine; 
using strange.extensions.context.api; 
using strange.extensions.command.impl; 
using strange.extensions.dispatcher.eventdispatcher.impl; 

namespace strange.examples.myfirstproject
{
	public class StartCommand : EventCommand
	{
		[Inject(ContextKeys.CONTEXT_VIEW)]       
		public GameObject contextView{get;set;}             
		
		public override void Execute()       
		{          
			GameObject go = new GameObject();          
			go.name = "ExampleView";          
			go.AddComponent<ExampleView>();          
			go.transform.parent = contextView.transform;       
		}    
	}
}

```

　　StartCommand继承自EventCommand，这说明了它是一个合法的Command可以被commandBinder使用。它继承了Command和EventCommand中的所有东西。特别要说明的是，继承EventCommand意味着你已经获得了一个IEvent来注入事件，并且你可以获得dispatcher。
　　如果你仅仅继承自Command，你将没法自动获得那些对象，但是你可以手动注入它们通过以下方法：

```
[Inject(ContextKeys.CONTEXT_DISPATCHER)]
IEventDispatcher dispatcher{get;set;}

[Inject]
IEvent evt{get;set;}

```

　　上面使用了两种不同的注入。IEventDispatcher和GameObject都用了被命名的实例。是因为我们想要参考这些对象的版本。我们不是仅仅想要任何一个GameObject，我们只想要被标记为ContextView的。我们也不会满足于任何一个旧的IEventDispatcher.仅有的一个用来在整个Context中通信的EventDispatcher被标记为ContextKeys.CONTEXT_DISPATCHER. 另一方面IEvent为了被这个特殊的command使用而被简单的映射，所以不需要名字。
　　在当前场景中我们所用的依赖是ContextView。我们将会给它增加一个子view。
　　Execute()方法是由commandBinder自动调用的。在大多数情况下，执行的顺序如下：
1、实例化与事件绑定的Command
2、注入依赖，包括IEvent本身
3、调用Execute()
4、删除Command
　　Commond并不需要立刻被清除，稍后我们会再做说明。如果你看了Excute()中的代码，你会发现它是纯Unity代码。创建一个实例，将一个MonoBehaviour赋上。然后将它的父节点设置为ContextView。我们所用的这个特别的MonoBehaviour正好是一个Strange IView。由于我们在context中映射了这个view:

```
mediationBinder.Bind<ExampleView>().To<ExampleMediator>();

```

　　这个view自动被中继，也就是说一个新的ExampleMediator刚刚被创建！

### A View is mediated

　　如果你之前写过unity代码，当你创建了一个View，你会认为它是一个MonoBehaviour, 但是重点是View是你可以看得到或者与之交互的。我将不会花时间来过ExampleView的代码。如果你懂得C#和Unity, 可以在示例文件中自己去看，并不需要过多解释。在此我只想让你注意两点，第一：

```
public class ExampleView : View 

```

　　通过继承View，你可以将每个View与Context建立联系。要使用Strange，你要么继承自View要么自己写相关的功能。但是如果你不继承View，你仍然需要实现IView接口。这个要求是为了确保Context可以操作你的MonoBehaviour。(在未来的版本里我或许会寻找其它方法来实现，允许你将一个Mediator映射到任意MonBehaviour)。
　　第二点：

```
[Inject]
public IEventDispatcher dispatcher{get; set;}

```

　　注意我们正在注入IEventDispatcher。但是这个跟与StartCommand里的那个不一样。仔细看代码，在上文中提到的EventCommand中写的是下面这个

```
[Inject(ContextKeys.CONTEXT_DISPATCHER)]
public IEventDispatcher dispatcher{get; set;}

```

　　通过对注入命名，Command指明了它使用共同的context dispatcher。View不应该注入那个dispatcher。中继的用途是将view与你的逻辑隔离。然而Strange仍然允许你注入到View,但是最好对这个功能有所限制。注入一个本地的dispatcher来与Mediator通信是可以的。注入一个布局文件(当你要发布到多个平台时会很有用)或者配置也是可以的。但是听我的建议，永远不要将model或者service或者其它View和Mediator之外的实例注入。
　　现在我将告诉你：这是整个框架中大部分开发者最难理解的概念。View应该只做展示和输入。当一些输入发生时View应当通知Mediator。Mediator(它被允许注入context dispatcher)抽象出View中与应用中其它地方有交互的部分。这将保护你的应用不受混乱的view代码影响，并且保护你的view不受相反的情况影响。
　　所以，别说我什么都没告诉你。
　　最后针对view，注意View基类使用了标准的MonoBehaviour回调 Awake(),Start(),和OnDestroy().所以如果你重写了这些回调的话，请记得要调用base.Awake()等，保证Strange运行正确。
　　现在我们看看Mediator

```
using System;
using UnityEngine;
using strange.extensions.dispatcher.eventdispatcher.api;
using strange.extensions.mediation.impl;

namespace strange.examples.myfirstproject
{
    public class ExampleMediator : EventMediator
    {
        [Inject]
        public ExampleView view{ get; set;}
        
        public override void OnRegister()
        {
            view.dispatcher.AddListener
				(ExampleView.CLICK_EVENT, onViewClicked);
            dispatcher.AddListener
				(ExampleEvent.SCORE_CHANGE, onScoreChange);
            view.init ();
        }
        
        public override void OnRemove()
        {
            view.dispatcher.RemoveListener
				(ExampleView.CLICK_EVENT, onViewClicked);
            dispatcher.RemoveListener
				(ExampleEvent.SCORE_CHANGE, onScoreChange);
            Debug.Log("Mediator OnRemove");
        }
        
        private void onViewClicked()
        {
            Debug.Log("View click detected");
            dispatcher.Dispatch(ExampleEvent.REQUEST_WEB_SERVICE,
				"http://www.thirdmotion.com/");
        }
        
        private void onScoreChange(IEvent evt)
        {
            string score = (string)evt.data;
            view.updateScore(score);
        }
    }
}

```

　　在最上面，看看我们在哪注入了ExampleView。这是Mediator获得他所中继的View的方法。Mediator可以获得View的绝大部分信息。(Mediator经常被认为是抛弃型代码”throw-away-code”，因为它十分针对View和程序其它部分的特性)。当然Mediator被允许知道View有个dispatcher并且这个dispatcher有一个叫做ExampleView.CLICK_EVENT的事件。通过监听该事件，Mediator使用了一个处理函数（onViewClicked()）来告知程序其它部分，这个事件代表什么。
　　我再次强调我之前的观点：View不应该发送REQUEST_WEB_SERVICE事件，View仅仅是视图作用。它仅仅触发一些事件类似 帮助按钮被点击，碰撞，向右滑动。将这些事件变得对程序其它部分有意义是Mediator的工作，比如 请求帮助，导弹击中敌方，玩家重新装弹。后面的事件会绑定到Command上，然后Command会调用支持系统，计算比分增加（增加Score Model中的分数）或者决定玩家是否被允许重新装填。
　　OnRegister()和OnRemove()方法就像mediator的构造函数和析构函数。OnRegister()在注入后会立刻调用，所以我通常用它设置监听器或者调用view的初始化函数。OnRemove()在MonoBehaviour的OnDestroy()后调用。他正好可以用来清除操作。确保你移除了所有的监听器，否则Mediator可能不会被正确的回收。
　　最后请注意，我们通过继承EventMediator可以获得common dispatcher。通过它Mediator类监听了SCORE_CHANGE事件，这单将在后面讲到。当View中有点击事件时，Mediator会发送REQUEST_WEB_SERVICE事件 ，这将会导致：

### 另一个Command被触发

　　回顾之前的Context代码，其中有一行我们没有讲清楚

```
commandBinder.Bind(ExampleEvent.REQUEST_WEB_SERVICE).To<CallWebServiceCommand>();

```

　　这个表示每当收到这个事件，就会启动CallWebServiceCommand。目前，我们并不是真正的去调用一个web服务，因为它跟教程无关，但是它会让你的注意力集中到一个稍微不同的使用Command的方法。

```
using System;
using System.Collections;
using UnityEngine;
using strange.extensions.context.api;
using strange.extensions.command.impl;
using strange.extensions.dispatcher.eventdispatcher.api;

namespace strange.examples.myfirstproject
{
    public class CallWebServiceCommand : EventCommand
    {
        [Inject]
        public IExampleModel model{get;set;}
        
        [Inject]
        public IExampleService service{get;set;}
 
        public override void Execute()
        {
            Retain ();
            service.dispatcher.AddListener
                (ExampleEvent.FULFILL_SERVICE_REQUEST, onComplete);
            string url = evt.data as string
            service.Request(url);
        }
        
        private void onComplete(IEvent result)
        {
            service.dispatcher.RemoveListener
			    (ExampleEvent.FULFILL_SERVICE_REQUEST, onComplete);
            model.data = result.data as string;
            dispatcher.Dispatch(ExampleEvent.SCORE_CHANGE, evt.data);
            Release ();
        }
    }
}

```

　　现在，其中的大部分代码你都应该理解了。我们在这个command中注入了ExampleModel和ExampleService。我们监听这个服务，并且触发时调用函数。我们使用了mediator触发的事件上所携带的数据（url）。当服务完成时（可能很快，也可能是5ms或者30秒！）就触发了onComplete()。我们取消监听器的映射，在model中设定一个值，最后触发SCORE_CHANGE事件，它会被mediator监听到。
　　一切都很顺利，但是如果你认真看就会想起我之前说过Command在执行Execute()后会立即释放。所以为何这个Command没有被垃圾回收？答案是在command的最顶部调用了Retain()函数。它会防止这个Command被释放。直到Release()被调用。显然调用Release()非常重要，除非你想要有内存泄漏。

### And we’re served…

　　你可以看看ExampleModel和ExampleService来弄明白它们是怎么运行的，但是它们其实很简单。Models仅仅是存数据的地方，Services是参与调用网络的。他们只有一个角色，比较简单。

`不要让Model和Service监听事件。`
　　Model和Service都是被Command使用的，它们并不在交互链中，所以他们不应该那样使用。你可以注入它们，而且你也可以注入一个本地的dispatcher (像上面一样)用来让它给你的Commands回应。而且注入context dispatcher并且发送事件也是可以的。但是

`不要让Model和Service监听事件。`
跨Context的映射
　　通常来说，你应该接受Context之间存在界限的事实。毕竟存在界限是有原因的：它可以使你的一部分程序独立的运行，更加模块化。但是有时一些对象，可能是一个model或者service或者signal需要跨越多个Context。在strange 版本v.0.6.0中我们增加了一个机制使得实现跨越更容易。像下面一样容易：

```
injectionBinder.Bind<IStarship>().To<HeartOfGold>().ToSingleton().CrossContext();

```

　　加上CrossContext()表明这个绑定会跨Context实例化。它将在所有子Context中可用。注意你也可以重写一个CrossContext绑定。如果你做了本地绑定，本地的绑定会覆盖了跨Context的绑定。

## 4.结论

　　上面大概解释了Strange如何使用。现在你了解了很多关于Strange如何运行，它可以做一些事情使得这个世界更美好。我建议你试一试多context的例子，因为它将给你更多的可能。我还建议你用Strange写一些东西，实践出真知。如果你从未用过一个IoC框架，我保证它会优化你的开发方式。如果你用过，我希望它不会让你失望。
　　最后，Strange是可以扩展的。无穷无尽的绑定器可以被创造。当更多的人接触这种思考方式，我希望我们能够看到更多令人惊艳的插件。