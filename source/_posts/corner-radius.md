---
title: 设置圆角的四种方法
date: 2022-09-28 16:11:15
tags:
---

# 设置圆角的四种方法

### 1.使用layer的cornerRadius

当设置了cornerRadius，同时设置了masksToBounds时候，在iOS9一下一定会触发离屏渲染；iOS9以后视情况发生离屏渲染（根据画家算法只要一个渲染步骤不会触发）

<!-- more -->

### 2.使用mask

由于使用mask会触发离屏渲染，故此种方法一定会产生离屏渲染

```
CAShapeLayer *mask = [CAShapeLayer new];
mask.path = [UIBezierPath bezierPathWithOvalInRect:view.bounds].CGPath;
view.layer.mask = mask;
```

### 3.使用Core Graphics重绘

此种方式，会消耗CPU，同时新生成的图片会增加内存开销，但不会产生离屏渲染

### 4.混合图层

UI提供中心透明，四周与背景同色图片。不会产生离屏渲染



# 总结

1.优选“混合图层”方案；

2.iOS9以上优选“cornerRadius”，如产生离屏渲染，在使用“混合图层”方案；

#### tip：

iOS9以后，UILabel和UIButton设置layer.cornerRadius、layer.backgroundColor，不设置layer.maskstoBounds不会触发离屏渲染
