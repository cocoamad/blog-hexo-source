title: objc kvo的秘密
date: 2014-03-09 09:47:04
tags: objc的秘密
---
KVO(Key Value Observing)，是`观察者模式`在`Foundation`中的实现


##KVO的原理
简而言之就是：
 1. 当一个object有观察者时，动态创建这个object的类的子类
 2. 对于每个被观察的property，重写其`set`和`get`方法
 3. 在重写的`set`方法中调用`- willChangeValueForKey:`和`- didChangeValueForKey:`通知观察者
 4. 当一个property没有观察者时，删除重写的`set`和`get`
 5. 当没有observer观察任何一个property时，删除动态创建的子类

空说无凭，简单验证下。

```
@interface Sark : NSObject
@property (nonatomic, copy) NSString *name;
@end

@implementation Sark
@end
```

```
Sark *sark = [Sark new];
// breakpoint 1
[sark addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew context:nil];
// breakpoint 2
sark.name = @"萨萨萨";
[sark removeObserver:self forKeyPath:@"name"];
// breakpoint 3
```
断点使用调试器分别使用`- class`和`object_getClass()`打出`sark`对象声称的Class和真实的Class
```
// breakpoint 1
(lldb) po sark.class
Sark
(lldb) po object_getClass(sark)
Sark

// breakpoint 2
(lldb) po sark.class
Sark
(lldb) po object_getClass(sark)
NSKVONotifying_Sark

// breakpoint 3
(lldb) po sark.class
Sark
(lldb) po object_getClass(sark)
Sark
```
上面的结果说明，在sark对象被观察时，framework使用`runtime`动态创建了一个Sark类的子类`NSKVONotifying_Sark`  
而且为了隐藏这个行为，NSKVONotifying_Sark重写了`- class`方法返回之前的类，就好像什么也没发生过一样  
但是使用`object_getClass()`时就暴露了，因为这个方法返回的是这个对象的`isa`指针，**这个指针指向的一定是个这个对象的类对象**  

接下来验证一下KVO重写set方法后是否调用了`- willChangeValueForKey:`和`- didChangeValueForKey:`  
最直接的验证方法就是在Sark类中重写这两个方法：


```
@implementation Sark

- (void)willChangeValueForKey:(NSString *)key
{
    NSLog(@"%@", NSStringFromSelector(_cmd));
    [super willChangeValueForKey:key];
}

- (void)didChangeValueForKey:(NSString *)key
{
    NSLog(@"%@", NSStringFromSelector(_cmd));
    [super didChangeValueForKey:key];
}

@end
```
// TODO:

##KVO的硬伤
###KVO坑爹的API
// TODO:
到现在还没有block版本的API？
###KVO难于调试
// TODO:
###KVO的死循环
// TODO:
