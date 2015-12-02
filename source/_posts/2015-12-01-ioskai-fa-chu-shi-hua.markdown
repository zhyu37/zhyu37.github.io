---
layout: post
title: "iOS开发初始化"
date: 2015-12-01 19:49:45 +0800
comments: true
categories: 
---
====
##iOS开发中new和alloc的区别
<p>
`概括的说alloc可以自定义init,new相当于采用默认的初始化`
>1. 一般很少用到[className new],我们一般用到的都是[[className alloc] init].但是不代表没用用的。
2. 这两个的区别是[className new]基本等同于[[className alloc] init]；区别只在于alloc分配内存的时候使用了zone.
`这个zone是个什么东东呢？`
它是给对象分配内存的时候，把关联的对象分配到一个相邻的内存区域内，以便于调用时消耗很少的代价，提升了程序处理速度； 
下面是源代码
<pre><code>
+new 
{ 
id newObject = (*_alloc)((Class)self, 0); 
Class metaClass = self->isa; 
if (class_getVersion(metaClass) > 1) 
return [newObject init]; 
else 
return newObject; 
} 
//而 alloc/init 像这样： 
+alloc 
{ 
return (*_zoneAlloc)((Class)self, 0, malloc_default_zone());  
} 
-init 
{ 
return self; 
} 
</pre></code>
3. 为什么不推荐使用new呢？
<p>因为如果使用new的话就只能采用默认的Init了，显然显示初始化比隐藏初始化好。两者没有什么太大的区别。

##See You