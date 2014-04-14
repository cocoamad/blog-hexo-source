title: @interface的设计哲学
date: 2014-04-13 13:01:54
tags: objc刨根问底
---

# 我是前言
学习objc时，尤其是先学过其他编程语言再来看objc时，总会对objc的**类**声明的关键字`interface`感到有点奇怪，在其它面向对象的语言中通常由`class`关键字来表示，而`interface`在java中表示的却大约相当于objc的`protocol`，这个关键字的区别究竟代表了objc语言的设计者怎样的思想呢？接下来对这个问题进行一些思考和探究.   

-----

# interface?
什么是interface？先来段Wiki:  
> In object-oriented programming, a protocol or interface is a common means for unrelated objects to communicate with each other. These are definitions of methods and values which the objects agree upon in order to cooperate.

接口约定了对象间交互的属性和方法，使得对象间无需了解对方就可以协作。  
说的洋气点就是`解耦`嘛，细心点也能发现Wiki中`interface`和`protocol`表示了相近的语义。    
引用我和项目组架构师讨论有关interface的问题时他的说法:

> interface就是一个object定义的可以被外界影响的方式

说着他指了下旁边桌子上放着的一把伞，说，这把伞我可以打开它，打开这个动作就是它的一个interface，桌子旁边还放着一个盒子，虽然它和伞都放在这张桌子上，但是它们之间永远不会互相影响，所以：  

> interface只存在于能互相影响的两者间  

-----

# @interface生成了class？

学习objc时最早接触的就是怎么写一个类了，从`.h`中写`@interface`声明类，再从`.m`中写`@implementation`实现方法，所以，objc中写一个`@interface`就相当于c++中写一个`class`。但这是真的么？  

写个小test验证一下： 
有两个类，`Sark`和`Dark`，`Sark`类只有`.m`文件，其中只写`@implementation`；`Dark`类只有`.h`头文件，其中只写`@interface`，然后如下测试代码：  


```
Class sarkClass = NSClassFromString(@"Sark");
Class darkClass = NSClassFromString(@"Dark");
```

`NSClassFromString`方法调用了runtime方法，根据类名将加载进runtime的这个类找出来，没有这个类就回返回空(Nil)。   
结果是`sarkClass`存在，而`darkClass`为空，说明什么？是否说明其实`@implementation`才是真正的Class？  
进一步，不止能取到这个没有@interface的类，还可以正常调用方法（因为万能的runtime）  

如下面的测试代码： 
```
Sark *sark = [Sark new];
[sark speak];
```

要是没有`@interface`的声明，类名，方法名都会报错说找不到，但是可以像下面一样绕一下：  

```
Class cls = NSClassFromString(@"Sark");
id obj = [cls performSelector:NSSelectorFromString(@"new")];
[obj performSelector:NSSelectorFromString(@"speak")];
```

其实，从`rewrite`后的objc代码可以发现，对于消息的发送，恰恰就是会被处理成类似上面的代码，使用字符串mapping出`Class`，`selctor`等再使用`objc_msgSend()`进行函数调用，如下面所示： 

```
// 经过clang -rewrite-objc 命令重写后的代码
Sark *sark = ((id (*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("Sark"), sel_registerName("new"));
((void (*)(id, SEL))(void *)objc_msgSend)((id)sark, sel_registerName("speak"));
```


# 对比@interface和@implementation

`@interface` 我们干过的事：
1. 继承
2. 声明协议
3. 定义实例变量（@interface后面加大括号那种）
4. 定义@property
5. 声明方法 

`@implementation` 我们干过的和可以干的事：  
1. 继承
2. 定义实例变量
3. 合成属性（@synthesize和@dynamic）
3. 实现方法（包括协议方法）

在`@implementation`干一些事情用的相对较少，但是是完全合法的，如这样用：  

``` 
@implementation Sark : NSObject {
    NSString *_name;
}
```

通过对比可以发现，**@interface对objc类结构的合成并无决定性作用**，加上**无决定性**是因为如果没有`@interface`会丢失一些类自省的原始数据，如属性列表和协议列表，但对于纯粹的对象消息发送并无影响。

-----

# 类与接口的设计原则 - 电视和遥控器
我喜欢将`Class`和`interface`的关系比喻成`电视+遥控器`，那么objc中的消息机制就可以理解成：  
**用户（caller）通过遥控器（interface）上的按钮（methods）发送红外线（message）来操纵电视（object）**  
所以，有没有遥控器，电视都在那儿，也就是说，有没有interface，class都是存在的，只是这种存在并没有意义，就好像这个电视没人会打开，没人会用，没人能看，一堆废铁摆在那儿。  

![](http://ww4.sinaimg.cn/large/51530583tw1efdy7cw48wj20c108qjru.jpg)

对比简洁的遥控器，一个拥有很多按钮的老式电视遥控器，我们经常会用到的按钮能有几个呢？
![](http://ww4.sinaimg.cn/large/51530583tw1efe08u9hb7j208c0b4jrp.jpg)

所以，在设计一个类的interface的时候，如同在设计遥控器应该有怎样功能的按钮，要从调用者的角度出发，区分边界，应该时刻有以下几点考虑： 
1. 这个方法或属性真的属于这个类的职责么？（电视遥控器能遥控空调？）
2. 这个方法或属性真的必须放在`.h`中（而不是放在`.m`的类扩展中）么？
3. 调用者必须看文档才能知道这个类该如何使用么？（买个电视还非要看说明书才会用？）
4. 调用者是否可以发现类内部的变量和实现方式？（脑补下电视里面一块电路板漏在外面半截- -） 
5. ...  


-----

# objc的@interface设计技巧Tips  

看过不少代码，从@interface设计上多少就能看出作者的水平，分享下我对于这个问题的一些拙见。 

## .h的@interface中只写需要外部看到的

比如，有如下一个类（这个类无意义，主要关注写法）：  
```
// Sark.h
@interface SarkViewController : NSObject <NSXMLParserDelegate /*1*/, NSCopying> {
    NSString *_name; // 2
    IBOutlet UITextField *_nameTextField; // 2
}
@property (nonatomic, strong) NSXMLParser *parser; // 3
- (IBAction)nameChangedAction:(id)sender; // 4
@end
```

这个interface出现的问题：
1. 类内部自己使用的协议，如`<NSXMLParserDelegate>`不应该在头文件@interface中声明，而应该在类扩展中声明；公开由外部调用的协议，如`<NSCopying>`则写在这儿是正确的。  
2. `实例变量`和`IBOutlet`不应出现在这儿定义，这将类的内部实现暴露了出去，自从属性可以自动合成后，这里就更应该清净了。
3. 内部使用的属性对象不要暴露在外，应该移动到类扩展中。
4. 调用者对IBAction同样不需要关心，那么就不应该放在这儿。  

## 将类按子功能分组

- 将相同功能的一组属性或方法写在一起
- 在头文件中也可以使用类扩展将interface按功能分区  

如，RAC


----- 


# References
http://en.m.wikipedia.org/wiki/Interface_(object-oriented_programming)

-----
原创文章，转载请注明源地址，blog.sunnyxx.com
