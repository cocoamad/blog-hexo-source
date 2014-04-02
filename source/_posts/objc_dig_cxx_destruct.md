title: 对ARC下对象释放过程及.cxx_destruct的探究
date: 2014-04-02 16:39:00
tags: objc刨根问底
---

## 写在最前  
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

  1. 这个对象持有的其他对象的释放去哪儿了？ 
  2. 没有显示的调用`[super dealloc]`，上层的析构去哪儿了？ 



 ## 1
  <!--more-->
http://clang.llvm.org/docs/AutomaticReferenceCounting.html#dealloc
  > The instance variables for an ARC-compiled class will be destroyed at some point after control enters the dealloc method for the root class of the class. The ordering of the destruction of instance variables is unspecified, both within a single class and between subclasses and superclasses.

   - 理解：ARC下对象的实例变量在根类[NSObject dealloc]中释放（通常root class都是NSObject），变量释放顺序各种不确定（一个类内的不确定，子类和父类间也不确定，也就是说不用care释放顺序）


  > Therefore we chose to delay destroying the instance variables to a point at which message sends are clearly disallowed: the point at which the root class’s deallocation routines take over.