title: ios程序员6级考试
date: 2014-03-06 23:10:58
tags: ios6级考试
---

ios面试题看过来，挂科男or挂科难？  
题目多来源于项目中遇到的错误和平时的误区，要是都能了如指掌，恭喜你。
 > Let's play a game. -- sunnyxx

##爸爸去哪儿？

```
@implementation Son : Father
- (id)init
{
    self = [super init];
    if (self)
    {
        NSLog(@"%@", NSStringFromClass([self class]));
        NSLog(@"%@", NSStringFromClass([super class]));
    }
    return self;
}
@end
```
<!--more-->

##以父之名
```
Father *father = [Father new];
BOOL b1 = [father responseToSelector:@selector(responseToSelector:)];
BOOL b2 = [Father responseToSelector:@selector(responseToSelector:)];
NSLog(@"%d, %d", b1, b2);
```

##睡梦中的主线程

```
...
// 当前在主线程

[request startAsync]; // 后台线程异步调用，完成后会在主线程调用competionBlock
sleep(100); // sleep主线程，使得下面的代码再后台线程完成后才能执行
[request setCompetionBlock:^{
    NSLog(@"Can I be print?");
}];
...
```

##frame
```
- (void)viewDidLoad
{
  [super viewDidLoad];
  UIView *view = [[UIView alloc] initWithFrame:CGRectMake(0, 0, self.bounds.size.width * 0.5, self.bounds.size.height * 0.5)];
  [self.view addSubview:view];
}
```
