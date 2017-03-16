# 利用 `Block` 打破 `NSTimer` 的*retain cycle*

> ​	在用到NSTimer类时，除非是一次性的计时器，我们需要调用 `invalidate` 方法使其失效，否则就很可能引入*retain cycle*，比如👇的例子：

```objective-c
//  EOCTimer.h
#import <Foundation/Foundation.h>

@interface EOCTimer : NSObject

- (void)startPolling;

- (void)stopPolling;

@end
  
//  EOCTimer.m
#import "EOCTimer.h"
@implementation EOCTimer {
    NSTimer *_pollTimer;
}

- (void)dealloc {
    NSLog(@"timer dealloc");
    [_pollTimer invalidate];
}

- (void)stopPolling {
    [_pollTimer invalidate];
    _pollTimer = nil;
}

- (void)startPolling {
    _pollTimer = [NSTimer scheduledTimerWithTimeInterval:5
                                                  target:self
                                                selector:@selector(p_doPoll)
                                                userInfo:nil
                                                 repeats:YES];
}

- (void)p_doPoll {
    NSLog(@"p_doPoll");
}

```

​	在这个例子中，由于创建的 `_pollTimer` 目标对象为 `self` ，所以需要保留此实例，然而，又因为 `_pollTimer` 是由该实例变量存放的，所以实例也保留了计时器，于是就产生了*retain cycle*。除非我们调用 `stopPolling` 方法，否则就不能回收该实例。

​	在这种情况下，我们没有办法确保 `stopPolling` 方法一定会被调用，并且，*通过调用某方法来避免内存泄漏的做法也不是个好方法。*

> 这个问题可以通过 `block` 来解决，可以利用分类为 `NSTimer` 类添加此功能：

```objective-c
//  NSTimer+EOCBlockSupport.h
#import <Foundation/Foundation.h>

@interface NSTimer (EOCBlockSupport)

+ (NSTimer *)eoc_scheduledTimerWithTimeInterval:(NSTimeInterval)interval
  										  block:(void (^) ())block
                                      	repeats:(BOOL)repeats;
  
//  NSTimer+EOCBlockSupport.m
#import "NSTimer+EOCBlockSupport.h"

@implementation NSTimer (EOCBlockSupport)

+ (NSTimer *)eoc_scheduledTimerWithTimeInterval:(NSTimeInterval)interval
                                          block:(void (^)())block
                                        repeats:(BOOL)repeats {
    return [self scheduledTimerWithTimeInterval:interval
                                         target:self
                                       selector:@selector(eoc_blockInvoke:)
                                       userInfo:[block copy]
                                        repeats:repeats];
}

+ (void)eoc_blockInvoke:(NSTimer *)timer {
    void (^block) () = timer.userInfo;
    if (block) {
        block();
    }
}
```

​	这样，计时器现在的 `target` 是 `NSTimer` 类对象，这是个单例，保存在堆内存区，无须回收，所以计时器是否会保留它，其实都无所谓。同时，我们把计时器需要执行的方法封装为 `block` ，作为 `userInfo` 参数传入。为了防止该 `block` 对象在要被执行时已经失效（超出了它的作用域范围），我们通过 `copy` 方法将它从拷贝到堆区。

> 修改后的 `startPolling` 方法如下👇：

```objective-c
- (void)startPolling {
    _pollTimer = [NSTimer eoc_scheduledTimerWithTimeInterval:5
                                                       block:^{
                                                           [self p_doPoll];
                                                       }
                                                     repeats:YES];
}
```

​	很明显，还是存在*retain cycle*，因为 `block` 捕获了 `self` 变量，所以 `block` 要保留实例；而计时器又通过 `userInfo` 参数保留了 `block` ；最后，实例本身还要保留计时器。

> 我们可以使用 weak 引用来打破*retain cycle*，如下👇：

```objective-c
- (void)startPolling {
      __weak EOCTimer *weakSelf = self;
    _pollTimer = [NSTimer eoc_scheduledTimerWithTimeInterval:5
                                                       block:^{
                                                         EOCTimer *strongSelf = self;
                                                         if (strongSelf) {
                                                           [strongSelf p_doPoll];
                                                         } else {
                                                           NSLog(@"strongSelf = nil");
                                                         }
                                                       }
                                                     repeats:YES];
}
```

> 这里用到了一种 *weak-strong-dance* 的写法，或者使用如下👇方法：

```objective-c
- (void)startPolling {
      __weak typeof(self) weakSelf = self;
    _pollTimer = [NSTimer eoc_scheduledTimerWithTimeInterval:5
                                                       block:^{
                                                         [weakSelf p_doPoll];
                                                       }
									                 repeats:YES];
}
```