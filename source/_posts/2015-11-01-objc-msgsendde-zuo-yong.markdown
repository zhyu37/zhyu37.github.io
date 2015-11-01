---
layout: post
title: "objc_msgSend的作用"
date: 2015-11-01 21:51:47 +0800
comments: true
categories: 
---
=====
##第十一条  objc_msgSend的作用

>objc_msgSend是obj-c中经常用到的一个功能，当你在对象上调用方法时就用用到这个——消息传递。

<pre><code>
id returnValue = [someObject messageName:parameter];
</pre></code> 
<li>someObject 是对象
<li>messageName 为对象所属的方法
<li>parameter 为方法所需要的参数

`messageName:parameter 叫选择子(selector) `     
`someObject  叫接收者(receiver)`

>objc_msgSend的底层使用C写的，C的方法调用是`静态绑定(static bingding)`把方法硬编码到指令中，当需要时直接调用。C还可以使用`动态绑定(dynamic bingding)`在指令中只有一个函数调用指令，在运行期读取出来。objc是一门真正的动态语言.

<pre><code>
//objc 的底层会把id returnValue = [someObject messageName:parameter];转换成
id returnValue = objc_msgSend(someObject,
								@selector(messageName),
								parameter);
</pre></code>

>在objc_msgSend中还用到了`尾调用优化`就是在结尾的时候调用另一个函数时，编译器是生成调转另一个函数的指令码，不会推入堆栈中。

##See You
