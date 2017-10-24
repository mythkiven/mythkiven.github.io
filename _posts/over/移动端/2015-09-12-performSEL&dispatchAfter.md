---
layout: post
title:  "Cancel performSelector & dispatch_after"
categories: OC  
tags: OC
author: 3行代码
---

* content
{:toc}


前几天项目里有个需求，要延时执行一个SEL，同时这个SEL在没有被执行前也可以被cancel。
最先想到的是performSelector...afterDelay...，因为其有cancel方法：

## performSelector

```
- (id)performSelector:(SEL)aSelector;  
- (id)performSelector:(SEL)aSelector withObject:(id)object;  
- (id)performSelector:(SEL)aSelector withObject:(id)object1 withObject:(id)object2;
```
这三个方法，均为同步执行，与线程无关，主线程和子线程中均可调用。等同于直接调用该方法。在需要动态的去调用方法的时候去使用。

```
- (void)performSelector:(SEL)aSelector withObject:(id)anArgument afterDelay:(NSTimeInterval)delay inModes:(NSArray *)modes;
- (void)performSelector:(SEL)aSelector withObject:(id)anArgument afterDelay:(NSTimeInterval)delay;
```
这两个方法为异步执行,且只能在主线程中执行，其它线程不执行，即使delay传参为0，也不会立即执行，而是在next runloop执行。

```
+ (void)cancelPreviousPerformRequestsWithTarget:(id)aTarget selector:(SEL)aSelector object:(id)anArgument;
+ (void)cancelPreviousPerformRequestsWithTarget:(id)aTarget;
```
在方法未执行之前，可以使用以上方法进行取消。

```
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait modes:(NSArray *)array;
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait;
```
1、这两个方法，在主线程和子线程中均可执行，均会在主线程中调aSelector方法,
2、设置wait为NO:等待当前线程执行完以后，主线程才会执行aSelector方法，
3、设置为YES：不等待当前线程执行完，就在主线程上执行aSelector方法。

## 取消一个待执行的Block

#### 1、方案1
```
-(void)run{
    [self performSelector:@selector(TEST) withObject:@"12" afterDelay:2];  
}
-(void)cancelRun{
    [NSObject cancelPreviousPerformRequestsWithTarget:self selector:@selector(TEST) object:@"12"];
}
```
但是操作需要添加到主线程才行，子线程不会执行延时的TEST:
```
dispatch_async(dispatch_get_main_queue(), ^{ [self run]  });
```

原因是performSelector afterDelay 是注册到当前线程的定时器里的，然后子线程没有runloop(需要手动添加)，没法delay执行SEL，so添加到主线程操作。

#### 2、方案2

```
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, 0.2*NSEC_PER_SEC);
dispatch_after(time, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        [weakSelf TEST];
    });
```

注意，这里也需要取消操作。我并没有找到GCD直接取消block的操作，需要自己进行构造的，在github上找到一个方案，实际操作也OK。paste如下：

```
#import <Foundation/Foundation.h>
typedef void(^SMDelayedBlockHandle)(BOOL cancel);
static SMDelayedBlockHandle perform_block_after_delay(CGFloat seconds, dispatch_block_t block) {
    
    if (nil == block) {
        return nil;
    }
    
    // block定义在栈上，可以使用__block修改变量，需将其copy到堆上。
    __block dispatch_block_t blockToExecute = [block copy];
    __block SMDelayedBlockHandle delayHandleCopy = nil;

    // 【这里，block被包裹进 一个取消的handle中：】
    SMDelayedBlockHandle delayHandle = ^(BOOL cancel){      
        if (NO == cancel && nil != blockToExecute) {
            dispatch_async(dispatch_get_main_queue(), blockToExecute);
        }
#if !__has_feature(objc_arc)
        [blockToExecute release];
        [delayHandleCopy release];
#endif
        blockToExecute = nil;
        delayHandleCopy = nil;
    };
    // delayHandle copy到堆上.
    delayHandleCopy = [delayHandle copy];
    
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, seconds * NSEC_PER_SEC), dispatch_get_main_queue(), ^{
        if (nil != delayHandleCopy) {
            delayHandleCopy(NO);
        }
    });
    return delayHandleCopy;
};

static void cancel_delayed_block(SMDelayedBlockHandle delayedHandle) {
    if (nil == delayedHandle) {
        return;
    }
    delayedHandle(YES);
}
```

方法奇妙之处就在于block copy到heap上执行，当需要中途取消时候，直接置空heap上的copy。

实际可以这样使用：
```
- (void)delayBlock {
    SMDelayedBlockHandle handle = perform_block_after_delay(2.0f, ^{
                [_delayedBlockHandle release];
                _delayedBlockHandle = nil;
            });
    _delayedBlockHandle = [handle retain];
}
 
- (void)cancelBlock {
    if (nil == _delayedBlockHandle) {
        return;
    }
    _delayedBlockHandle(YES);
    [_delayedBlockHandle release];
    _delayedBlockHandle = nil;
}

```

参考博客：[博客](https://blog.spacemanlabs.com/2011/12/cancel-dispatch_after/)











