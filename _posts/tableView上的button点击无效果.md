# iOS tableView上的button点击无效果

* button 设置了 Normal 和 Highlighted 两种背景图
* button 只添加了 TouchUpInside 方法

> Q: 在正常（较为快速）的点击时，根本不显示 Highlighted 效果图，而 长按 时有高亮效果

##### 解决方法
```objective-c
self.tableView.delaysContentTouches = NO;
```

```objective-c
for (UIView *currentView in self.tableView.subviews) {
    if([currentView isKindOfClass:[UIScrollView class]]) {
        ((UIScrollView *)currentView).delaysContentTouches = NO;
        break;
    }
}
```

