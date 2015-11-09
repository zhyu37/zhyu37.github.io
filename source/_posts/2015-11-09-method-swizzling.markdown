---
layout: post
title: "Method Swizzling"
date: 2015-11-09 13:08:45 +0800
comments: true
categories: 
---
====
##第十三条  用“方法调配技术”调试“黑盒方法”
>在没有一个类的源码时，例如调用别人API时。想改变其中一个方法的实现，除了继承重写、借助重类名方法之外，现在还发现了一种新的方法——Method Swizzling。

<p>一个NSString类的例子</p>
<pre><code>
//头文件 
#import  <objc/runtime.h>

//具体实现
 //1.测试字符串
    NSString *str = @"Hello World";
    
//2.打印正常方法调用的值
    NSLog(@"%@ %@",[str lowercaseString],[str uppercaseString]);
    
Method originalMethod = class_getInstanceMethod([NSString class], @selector(lowercaseString));
    Method swappedMethod = class_getInstanceMethod([NSString class], @selector(uppercaseString));
    method_exchangeImplementations(originalMethod, swappedMethod);
    
 //4.打印方法调配后的值
    NSLog(@"%@ %@",[str lowercaseString],[str uppercaseString]);
</pre></code>

<p>打印结果
<pre><code>
2015-11-09 12:49:23.361 ChangeMethods[810:459887] hello world HELLO WORLD
2015-11-09 12:49:23.362 ChangeMethods[810:459887] HELLO WORLD hello world
</pre></code>

>在Objective-C中调用一个方法，其实是向一个对象发送消息，查找消息的唯一依据是selector的名字。利用Objective-C的动态特性，可以实现在运行时偷换selector对应的方法实现，达到给方法挂钩的目的。
每个类都有一个方法列表，存放着selector的名字和方法实现的映射关系。IMP有点类似函数指针，指向具体的Method实现。我们可以利用 method_exchangeImplementations 来交换2个方法中的IMP，
我们可以利用 class_replaceMethod 来修改类，
我们可以利用 method_setImplementation 来直接设置某个方法的IMP
##See You