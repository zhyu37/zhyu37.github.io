---
layout: post
title: "实现description方法"
date: 2015-11-25 16:05:11 +0800
comments: true
categories: 
---
====
##实现description方法

>当想打印对象的信息时，对象会收到description消息，该方法返回的描述信息回取代“格式字符串”里的“%@”。

<p>
<pre><code>
NSArray *object = @[@"a string, @(123)"];
NSLog(@"object = %@",object);

//如果是打印 会输出
object = (
	"a string",
	123
)

//如果在自定义的类上这么做  则会打印
object = <EOCPerson: 0x7fd9a1660600>

</pre></code>
>但是 这样的打印 如果不需要 类名 和 地址的话 ，这个打印毫无意义。so 我们需要复写 description方法。

<pre><code>
EOCPerson *person = [[EOCPerson alloc] initWithFirstName:@"Bob" lastName:@"Smith"];
NSLog(@"person = %@", person);

该类的description方法通常是这样实现的
-(NSString*) description{
	return [NSStringstringWithFormat:@"<%@: %p, \"%@ %@\">",[self class], self, _firstName, _lastName];
}

//打印结果
person = <EOCPerson: 0x7fd9a1660600, "Bob Smith">
</pre></code>

>`作者建议:`在新实现的description方法中，也应该像默认的实现那样，打印出类的名字和指针地址，因为这些内容有时也会用到。NSArray类的对象就没有打印这两项内容，显然没有固定规律可循。`用NSDicionary来实现此功能可以使代码更以维护.`

<p>debugDescription
>此方法的用意与description非常相似。区别在于前者是开发者在调试器中以控制台命令打印对象时才调用的。用法和description类似.

##See You