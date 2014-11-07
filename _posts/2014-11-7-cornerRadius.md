---
layout: post
title: 圆角UIView的几种实现方案
---
##圆角UIView的几种实现方案

在项目里经常会遇到设计上有很多圆角的问题，最常见的解决方案是直接设置layer的两个属性：
```
view.layer.cornerRadius = 5.0f;
view.layer.maskToBounds = YES;
```

