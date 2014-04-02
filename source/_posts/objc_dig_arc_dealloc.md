title: ARC下dealloc过程及.cxx_destruct的探究
date: 2014-04-02 16:39:00
tags: objc刨根问底
---

## 我是前言  
这次探索源自于自己一直以来对`ARC`的一个疑问，在`MRC`时代，经常写下面的代码：  

```
- (void)dealloc
{
    self.array = nil;
    self.string = nil;
    // ... //
    // 非Objc对象内存的释放，如CFRelease(...)
    // ... //
    [super dealloc];
}
```

对象析构时将内部其他对象`release`掉，申请的非Objc对象的内存当然也一并处理掉，最后调用`super`，继续将父类对象做析构。而现如今到了`ARC`时代，只剩下了下面的代码：

```
- (void)dealloc
{
    // ... //
    // 非Objc对象内存的释放，如CFRelease(...)
    // ... //
}
```

**问题来了：**  

  1. 这个对象实例变量（Ivars）的释放去哪儿了？ 
  2. 没有显示的调用`[super dealloc]`，上层的析构去哪儿了？ 


------

<!--more-->
## ARC文档中对dealloc过程的解释
 
[llvm官方的ARC文档](http://clang.llvm.org/docs/AutomaticReferenceCounting.html#dealloc)中对ARC下的dealloc过程做了简单说明，从中还是能找出些有用的信息： 

  >A class may provide a method definition for an instance method named dealloc. This method will be called after the final release of the object but before it is deallocated or any of its instance variables are destroyed. The superclass’s implementation of dealloc will be called automatically when the method returns.

 - 大概意思是：dealloc方法在最后一次release后被调用，但此时实例变量（Ivars）并未释放，**父类的dealloc的方法将在子类dealloc方法返回后自动调用**


  > The instance variables for an ARC-compiled class will be destroyed at some point after control enters the dealloc method for the root class of the class. The ordering of the destruction of instance variables is unspecified, both within a single class and between subclasses and superclasses.

 - 理解：ARC下对象的实例变量在根类[NSObject dealloc]中释放（通常root class都是NSObject），变量释放顺序各种不确定（一个类内的不确定，子类和父类间也不确定，也就是说不用care释放顺序）

所以，不用主调`[super dealloc]`是因为自动调了，暂时不知道如何实现的；ARC下实例变量在根类NSObject析构时析构，下面就探究下。

------

## NSObject的析构过程
通过apple的runtime源码，不难发现NSObject执行`dealloc`时调用`_objc_rootDealloc`继而调用`object_dispose`随后调用`objc_destructInstance`方法，前几步都是条件判断和简单的跳转，最后的这个函数如下：
```
void *objc_destructInstance(id obj) 
{
    if (obj) {
        Class isa_gen = _object_getClass(obj);
        class_t *isa = newcls(isa_gen);

        // Read all of the flags at once for performance.
        bool cxx = hasCxxStructors(isa);
        bool assoc = !UseGC && _class_instancesHaveAssociatedObjects(isa_gen);

        // This order is important.
        if (cxx) object_cxxDestruct(obj);
        if (assoc) _object_remove_assocations(obj);
        
        if (!UseGC) objc_clear_deallocating(obj);
    }

    return obj;
}
```

简单明确的干了三件事：
  1. 执行一个叫`object_cxxDestruct`的东西干了点什么事
  2. 执行`_object_remove_assocations`去除和这个对象assocate的对象（常用于category中添加带变量的属性，这也是为什么ARC下没必要remove一遍的原因）
  3. 执行`objc_clear_deallocating`，清空引用计数表并清除弱引用表，将所有`weak`引用指nil（这也就是weak变量能安全置空的所在）

所以，所探寻的ARC自动释放实例变量的地方就在`cxxDestruct`这个东西里面没跑了。

------

## 探寻隐藏的.cxx_destruct

上面找到的名为`object_cxxDestruct`的方法最终成为下面的调用：

```
static void object_cxxDestructFromClass(id obj, Class cls)
{
    void (*dtor)(id);

    // Call cls's dtor first, then superclasses's dtors.

    for ( ; cls != NULL; cls = _class_getSuperclass(cls)) {
        if (!_class_hasCxxStructors(cls)) return; 
        dtor = (void(*)(id))
            lookupMethodInClassAndLoadCache(cls, SEL_cxx_destruct);
        if (dtor != (void(*)(id))_objc_msgForward_internal) {
            if (PrintCxxCtors) {
                _objc_inform("CXX: calling C++ destructors for class %s", 
                             _class_getName(cls));
            }
            (*dtor)(obj);
        }
    }
}
```
