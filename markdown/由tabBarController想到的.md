# 由tabBarController想到的

##### 	今天突然纠结一个问题，在tabBarController中添加了两个viewController，当我选择不同的tabBarItem时，可以切换不同的viewController，那么，如何来监听它呢?

> 通常，我们会在创建tabBarController时同时设置它的childViewController，并将self设置为tabBarController的delegate，来监听点击了哪一个tabBarItem并且didSelect了哪一个viewController；但是，我如果不想在tabBarController这个类之外来监听呢？该怎么做？

##### 	我先去翻阅了关于 `tabBarController` 的头文件，希望找到一些线索，找到了 `selectedViewController` 与 `selectedIndex` 这两个相关的属性，

##### 	没错，我要做的就是监听这两个属性的变化。

##### 	以及👇这个相关的协议方法。（在当前情况下不适用，略过）

```objective-c
- (void)tabBarController:(UITabBarController *)tabBarController didSelectViewController:(UIViewController *)viewController;
```

##### 	我一看，没有别的方法可以监听变化了，那么就对这两个属性下手，想到KVO，的确可以实现。

```objective-c
//在tabBarController.m中，设置了对tabBarController selectedViewController 属性的观察者
[self addObserver:self forKeyPath:@"selectedViewController" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];

//观察tabBarController selectedViewController 属性的变化，因为知道object 为 self 也就是 EOCTabBarController，所以我把方法中的object 类型 直接改写为了 EOCTabBarController。
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(EOCTabBarController *)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    if ([keyPath isEqualToString:@"selectedViewController"]) {
        NSLog(@"%li", object.selectedIndex);
        switch (object.selectedIndex) {
            case 0: {
                vc1.navigationController.navigationBar.barTintColor = [UIColor blueColor];
            }
                break;
            default: {
                vc2.navigationController.navigationBar.barTintColor = [UIColor redColor];
            }
                break;
        }
    }
}
```

##### 	但是想到 `tabBarController` 内部的 `selectedViewController` 变化都需要这么麻烦的来监听，肯定有更简便的方法我没有发现。想到我选择不同的 `viewController` 是在 `tabBar` 上操作的，那么去 `tabBar` 看看，应该能有收获，点进去一看，果然，一个协议方法

 ```objective-c
- (void)tabBar:(UITabBar *)tabBar didSelectItem:(UITabBarItem *)item;
 ```

##### 	这么一想就通了， `tabBarController` 管理着 `tabBar` 并且是它的 `delegate` ，因此遵循这个方法，所以，依靠这个方法就可以轻松的实现我想要达到的效果。

 ```objective-c
//首先取出tabBarController管理的viewControllers，并同时得到它的tabBarItem。
- (void)viewDidLoad {
   [super viewDidLoad];

   [self.childViewControllers enumerateObjectsUsingBlock:^(__kindof UIViewController * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
       switch (idx) {
           case 0: {
               vc1 = obj;
               item1 = vc1.tabBarItem;
           }
               break;
           default: {
               vc2 = obj;
               item2 = vc2.tabBarItem;
           }
               break;
       }
   }];
}

//根据不同的item可以找到对应的viewController，😄。
- (void)tabBar:(UITabBar *)tabBar didSelectItem:(UITabBarItem *)item {
   if (item == item1) {
       vc1.navigationController.navigationBar.barTintColor = [UIColor blueColor];
   } else {
       vc2.navigationController.navigationBar.barTintColor = [UIColor redColor];
   }
}
 ```

#### 	折腾完这事儿之后，对协议的思想有了更深的理解，这波不亏，hhh。
