---
layout: post
title: 圆角UIView的几种实现方案
---

在项目里经常会遇到设计上有很多图片显示成圆角的问题，最常见的解决方案是直接设置layer的两个属性：
{% highlight cpp %}
view.layer.cornerRadius = 5.0f;
view.layer.maskToBounds = YES;
{% endhighlight %}
如果在UITableViewCell中使用这种方法绘制圆角，会造成table滚动时的卡顿。卡顿的原因主要是系统的渲染分成了两个部分：屏幕上（onscreen）的渲染和屏幕外（offscreen）CALayer的渲染。offscrren渲染完全是计算出来的，不能用到硬件加速，而每次table滚动时，都需要重新渲染这两个部分，造成GPU计算量大而卡顿。
要解决这种卡顿一般实现方案有三种：

###直接使用圆角图片覆盖原图
![mask picture](http://dream.ph.126.net/PPty8ZAqxoik7_LHyZAZyQ==/3853788255403936)
这种方法适用于背景是纯色的情况，直接把圆角mask图片覆盖原图即可。这种方法比较灵活，还可以适用于各种形状的mask。

###在子线程绘制圆角图片，绘制好后到主线程显示
这种方法不使用CoreAnimation的来渲染圆角，因此把最费时的offscreen计算省下来。主要思路是在子线程上绘制图片，使用BezierPath对图片进行裁剪，将最后生成的UIImage传到主线程显示。
裁剪图片的代码如下：
{% highlight cpp %}
-(UIImage *)makeRoundedCornersForImage:(UIImage *)img withCornerRadius:(float)cornerRadius
{
    CGRect imageRect = CGRectMake(0, 0, img.size.width, img.size.height);
    UIGraphicsBeginImageContextWithOptions(img.size, NO, 1.0);
    [[UIBezierPath bezierPathWithRoundedRect:imageRect
                                cornerRadius:cornerRadius] addClip];
    [img drawInRect:imageRect];
    UIImage *roundImg = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return roundImg;
}
{% endhighlight %}

###使用shouldRasterize属性
使用CALayer的cornerRadius和maskToBounds属性最主要的性能瓶颈在于每次渲染时繁复的blend layer计算，造成滚动时页面卡顿的现象。这时可以使用CALayer的shouldRasterize属性，设置layer.shouldRasterize = YES。当这个属性设为YES的时候，CA会cache住栅格化的图形，滚动时直接从cache中取出图形显示，避免了反复的层计算。
由于shouldRasterize会cache和重用layer，所以它适用于layer的内容不经常变化的情况。如果layer的内容经常变化，每次变化都需要重新栅格化layer、重新cache，反而会使得性能更差。
使用shouldRasterize的例子为：
{% highlight cpp %}
self.layer.backgroundColor = [UIColor clearColor].CGColor;
self.layer.shouldRasterize = YES;
self.layer.masksToBounds = YES;
self.layer.cornerRadius = 5.0f;
{% endhighlight %}


###参考
- [Applying Rounded Corners](http://articles.cocoahope.com/blog/2013/03/06/applying-rounded-corners)
- [UILabel layer cornerRadius negatively impacting performance](http://stackoverflow.com/questions/4735623/uilabel-layer-cornerradius-negatively-impacting-performance)
- [Help! My tables don’t scroll smoothly!](http://iosinjordan.tumblr.com/post/56778173518/help-my-tables-dont-scroll-smoothly)
