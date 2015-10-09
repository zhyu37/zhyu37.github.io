---
layout: post
title: "Effective Objective-C 2.0--10.在既有类中使用关联对象存放自定义数据"
date: 2015-10-10 00:47:52 +0800
comments: true
categories: 
---
当我们想在对象中存放其他信息时，一般会采用创建其子类后改用这个子类对象。然而并非所有情况都可以这么做。so Object-c中有一个强大的特性——关联对象(Associated Object)

关联策略

关联策略
等价属性
说明

OBJC_ASSOCIATION_ASSIGN
@property (assign) or @property (unsafe_unretained)
弱引用关联对象

OBJC_ASSOCIATION_RETAIN_NONATOMIC
@property (strong, nonatomic)
强引用关联对象，且为非原子操作

OBJC_ASSOCIATION_COPY_NONATOMIC
@property (copy, nonatomic)
复制关联对象，且为非原子操作

OBJC_ASSOCIATION_RETAIN
@property (strong, atomic)
强引用关联对象，且为原子操作

OBJC_ASSOCIATION_COPY
@property (copy, atomic)
复制关联对象，且为原子操作


其中，第 2
 种与第 4
 种、第 3
 种与第 5
 种关联策略的唯一差别就在于操作是否具有原子性。由于操作的原子性不在本文的讨论范围内，所以下面的实验和讨论就以前三种以例进行展开。

方法

//以给定的键和策略为某对象设置关联对象值
void objc setAssociatedObject(id object, const void *key, id value, objc AssociationPolicy policy);

//根据给给定的键从某对象中获取相应地关联对象值
 id objc getAssociatedObject(id object, const void *key);

//移除所指定对象的全部关联对象
void objc removeAssociatedObjects(id object);

key 值

关于前两个函数中的 key
 值是我们需要重点关注的一个点，这个 key 值必须保证是一个对象级别（为什么是对象级别？看完下面的章节你就会明白了）的唯一常量。一般来说，有以下三种推荐的 key
 值：

1. 声明 static char kAssociatedObjectKey;，使用 &kAssociatedObjectKey 作为 key 值;
2. 声明 static void *kAssociatedObjectKey = &kAssociatedObjectKey; ，使用 kAssociatedObjectKey 作为 key值；
3. 用 selector，使用 getter方法的名称作为 key值



我们把对象想象成NSDicitionary,把关联对象想象成字典的条目。存取关联对象的值就相当于NSDicitionary中调用[object setObject:value forKey:key]和[object setObjectForKey:key]。两者之间有个最重要的差别key(键) 是一个不透明的指针。所以通常使用静态全局变量做键。
