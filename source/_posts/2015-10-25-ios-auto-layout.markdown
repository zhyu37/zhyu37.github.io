---
layout: post
title: "Ios Auto Layout"
date: 2015-10-25 20:27:46 +0800
comments: true
categories: 
---
======
##Ios Auto Layout
> 一开始 学的时候frame的绝对位置确实比autLayout好用，但是随着iphone 的尺寸越来越多 就会发现在4寸屏上适配成功，发现到plus 上就不对了。so 我选择了autoLayout 因为他是相对布局 所以它更适合 尺寸的变换和 竖屏横屏的切换。但也不能说frame 没用处。

<p>autolayout的VFL(Visual Format Language)语法</p>

<pre><code>
NSString *vfl = @"V:|-5-[_view]-10-[_imageView(20)]-10-[_backBtn]-5-|";

//其实这段话就是说，在垂直方向从上到下，view离父视图5点，imageView距离view 10点，同时imageView是20点高，backBtn离imageView底部10点，距离父视图底部5点。
</pre></code>

<p>手动Constraint书写</p>
<pre><code>
[self.view addConstraint: [NSLayoutConstraint constraintWithItem:blueView
attribute:NSLayoutAttributeLeft
relatedBy:NSLayoutRelationEqual
toItem:redView
attribute:NSLayoutAttributeLeft
multiplier:1
constant:0]];
</pre></code>

>通过上面的 讲解  发现AutoLayout还需要花费经历 再去学习一番。SO 我在github上发现了一位大神写的第三方的东西`Masonry`.https://github.com/SnapKit/Masonry  
>开源项目Masonry旨在让自动布局（Auto Layout）的代码更简洁、可读性更强。

>Masonry ，“一个轻量级的布局框架，采用更优雅的语法封装自动布局”，不需要使用XIB和Storyboard。它的创造者Jonas Budelmann 论证 了尽管自动布局很强大，但它很快就变得冗长而不可读。

>Masonry是一种领域特定语言（DSL），为自动布局的所有功能提供便捷的方法，包括建立和修改约束、存取属性、设置优先级以及调试支持。
<p>view的x位于父视图中心，y距顶部5，width为10，hight为20。还是同样地意思 我用这种框架来写一下</p>
<pre><code>
[view mas_makeConstraints:^(MASConstraintMaker *make) {
        make.centerX.equalTo(self.mas_centerX);
        make.top.equalTo(self.mas_top).offset(5);
		make.width.equalTo(@(10));
        make.height.equalTo(@(20));
    }];
</pre></code>
<p>感觉还是很好用的</p>

##See You