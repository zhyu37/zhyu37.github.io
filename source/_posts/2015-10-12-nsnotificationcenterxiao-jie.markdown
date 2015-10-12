---
layout: post
title: "NSNotificationCenter小结"
date: 2015-10-12 23:41:11 +0800
comments: true
categories: 
---
====
##NSNotification 消息机制
>最近在项目中发现这个NSNotification还是很好用的。那我就简单介绍一下步骤，先要注册对象，当有人post的时候调用注册的方法，当然不能忘记`remove`注册了的对象。


<p>注册对象</p>

<pre><code>
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(test) name:@"test" object:nil];
}
</code></pre>

<p>当对象被post之后  会调用 写好的test方法</p>

<pre><code>
 [[NSNotificationCenter defaultCenter] postNotificationName:@"test" object:nil];
    });
</pre></code>

<p>删除对象</p>

<pre><code>
[[NSNotificationCenter defaultCenter] removeObserver:self name:@"test" object:nil];
</code></pre>

`注册和删除对象 一定要成对出现  ，当然也可以在dealloc里面 一起删除`

<pre><code>
- (void)dealloc{
    [[NSNotificationCenter defaultCenter] removeObserver:self];
    [super dealloc];
}
</pre></code>

<p>在这个类的.h 文件中添加下面的的外部申明</p>

<pre><code>
extern NSString * const testNotification;
</pre></code>

<p>在.m中定义常量</p>

<pre><code>
NSString * const testNotification = @"BNRColorChanged";
</pre></code>

`如果安上面的申请方法 我们的注册 和 删除对象也可以改变一下`

<pre><code>

[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(test) name:testNotification object:nil];
}

post 删除 同理

</pre></code>

####See You 