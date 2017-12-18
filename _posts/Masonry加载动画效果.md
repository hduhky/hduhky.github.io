# Masonry加载动画效果

```objective-c
	[self.progressTopView mas_updateConstraints:^(MASConstraintMaker *make) {
        make.width.mas_equalTo(width);
    }];
    
    [UIView animateWithDuration:.2 animations:^{
        [self layoutIfNeeded];
    }];
```

