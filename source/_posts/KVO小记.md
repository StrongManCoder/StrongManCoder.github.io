---
title: KVO小记
date: 2016-09-13 11:46:21
tags: [KVO,KVC,监听]
---

# KVO浅析
### KVO Key-Value Observing 键值监听
KVO是一个观察者模式。观察一个对象的属性，注册一个指定的路径，若这个对象的的属性修改，则KVO会自动通知观察者。

## 使用步骤
### 1.注册监听


```/**
     *  注册一个监听
     *
     *  @param observer 观察者
     *  @param keyPath  属性名字
     *  @param options  属性的变化
     *  @param context  void类型
     */
- (void)addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(nullable void *)context;
```
### 2.回调函数


```/**
 *  当监控的某个属性的值改变了就会调用
 *
 *  @param keyPath 属性名（哪个属性改了？）
 *  @param object  哪个对象的属性被改了？
 *  @param change  属性的修改情况（属性原来的值、属性最新的值）
 *  @param context void类型
 */
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
    NSLog(@"%@对象的%@属性改变了：%@", object, keyPath, change);
}
```

### 3.移除观察,释放内存，在dealloc函数释放



```/**
 *  移除观察者
 *
 *  @param observer 观察者
 *  @param keyPath  观察的属性
 */
- (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath;
- (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath context:(nullable void *)context NS_AVAILABLE(10_7, 5_0);
```


KVO的观察者是由两种模式的。一种是自动通知，一种是手动通知。
**自动通知**自动监听对象的属性，不管这个属性的前后属性变化的值是否一样，都会通知观察者。
手动通知重写willChangeValueForKey:和didChangeValueForKey: 方法通知观察者。
一般都是用自动通知，方便快捷。
下面两种写法，都会举例说明。

### 自动通知,主要看监听回调的分析
场景：模拟人年龄的变化，看接受通知的次数

```- (void)viewDidLoad {
    [super viewDidLoad];

    self.person = [[Person alloc]init];
    self.person.age = 15;

    /**
     *  注册监听
     */
    [self.person addObserver:self forKeyPath:@"age" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];
    /**
     *  第二次赋值 20
     */
    self.person.age = 20;
    /**
     *  再次赋值 20
     */
    self.person.age = 20;
    /**
     *  详细见控制台输出,只比较新旧值
     */    
}

/**
 *  监听回调
 */
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context{

    NSLog(@"\n change = %@ \n keyPath =%@ object =%@ context=%@",change ,keyPath,object,context);



    /** ============================ 控制台输出 ===================================
     * 


     change = {
     kind = 1;
     new = 20;        --------->新的值是 20 注册监听后，observeValueForKeyPath会回调输出
     old = 15;        --------->旧的值是 15 注册监听前，这个属性初始化时候就是15。
     }
     keyPath =age object =<Person: 0x7feec962a9d0> context=(null)


     change = {
     kind = 1;
     new = 20;        --------->新的值是 20 再次传入20，observeValueForKeyPath这个还是会回调输出
     old = 20;        --------->旧的值是 20
     }
     keyPath =age object =<Person: 0x7feec962a9d0> context=(null)

     */


    /**
     *  由控制台输出结果得出结论，只要是对属性进行改变，不管属性的值是否变化，不区分新旧值的变化，observeValueForKeyPath都是回调，通知观察者的。
     */

}


-(void)dealloc{

    /**
     *  移除观察
     */
    [self.person removeObserver:self forKeyPath:@"age"];
}
```

### 手动通知
场景：模拟人年龄的变化，看接受通知的次数
需要在Person类里面重写方法，具体看实现代码

```- (void)viewDidLoad {
    [super viewDidLoad];

    self.person = [[Person alloc]init];
    self.person.age = 15;

    /**
     *  注册监听
     */
    [self.person addObserver:self forKeyPath:@"age" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];
    /**
     *  第二次赋值 20
     */
    self.person.age = 20;
    /**
     *  再次赋值 20
     */
    self.person.age = 20;

    /**
     *  详细见控制台输出,只比较新旧值
     */

}

/**
 *  监听回调
 */
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context{

    NSLog(@"\n change = %@ \n keyPath =%@ object =%@ context=%@",change ,keyPath,object,context);

    /** ============================ 控制台输出 ===================================
     * 
     change = {
     kind = 1;
     new = 20;        --------->新的值是 20 注册监听后，observeValueForKeyPath会回调输出
     old = 15;        --------->旧的值是 15 注册监听前，这个属性初始化时候就是15。
     }
     keyPath =age object =<Person: 0x7feec962a9d0> context=(null)

     */
   /**
     *  和自动通知输出作对比，这样明白理解。主要是在Person类里面重写了 automaticallyNotifiesObserversForKey 这个方法。以及重写age的setter方法
     *  由控制台输出结果得出结论，只要监听对象的属性值前后发生改变，observeValueForKeyPath就回调，通知观察者的。
     */

}

-(void)dealloc{

    /**
     *  移除观察
     */
    [self.person removeObserver:self forKeyPath:@"age"];
}
```

### Person类的实现


```//
//  Person.m
//  KVO
//
//  Created by XXXXXX on 15/11/9.
//  Copyright © 2015年 Simon. All rights reserved.
//

#import "Person.h"

@implementation Person
-(void)setAge:(float)age{

    /**
     *  判断两值是否一样，一样就不复赋值了
     */
    if (_age!= age) {
        [self willChangeValueForKey:@"age"];

        _age = age;

        [self didChangeValueForKey:@"age"];
    }
}

/**
 *  是否对这个key开启自动发送通知
 *
 *  @param theKey 监听的属性key
 *
 *  @return 布尔值
 */
+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)theKey {
    BOOL automatic = NO;

    if ([theKey isEqualToString:@"age"])
    {
        automatic = NO;
    }
    else
    {
        automatic=[super automaticallyNotifiesObserversForKey:theKey];
    }
    return automatic;
}
@end
```

**观察了自动通知和手动通知，各有所长，看你们喜欢哪个。记住手动通知必须重写方法，只有新旧值前后不一样才会通知。自动通知就不管新旧值是否一样，都说告诉观察者。**
实现原理可以参考这个博客，写的很详细.
[[深入浅出Cocoa]详解键值观察（KVO）及其实现机理](http://blog.csdn.net/kesalin/article/details/8194240)


