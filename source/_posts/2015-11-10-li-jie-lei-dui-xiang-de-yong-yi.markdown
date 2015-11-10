---
layout: post
title: "查询类的类型"
date: 2015-11-10 09:20:25 +0800
comments: true
categories: 
---
===
##第十四条 理解“类对象”的用意
>每个实例都用一个指向Class对象的指针，用来表示其类型。如果对象类型无法在编译期确定，那么就应该使用类型信息查询方法来探知。尽量使用类型信息查询方法来确定对象类型，而不要直接比较类对象，因为某些对象可能实现了消息转发功能。

<pre><code>
NSMutableDictionary *dict = [NSMutableDictionary new];
//判断对象是否是否个特定类的实例
[dict isMemberofClass:[NSDictionary class]];//NO
[dict isMemberofClass:[NSMutableDictionary class]];//Yes
//判断对象是否为某个类或其派生类的实例
[dict isKindofClass:[NSDictionary class]];//Yes
[dict isKindofClass:[NSArray class]];//No
</pre></code>

##See You