---
layout: post
title: "iOS开发之深拷贝、浅拷贝"
date: 2015-12-01 17:01:32 +0800
comments: true
categories: 
---
=======
##iOS开发之深拷贝与浅拷贝
<p>
`如有错误请大家多多指教，请大家多关注 https://github.com/zhyu37 `
>对象拷贝分为两种方式：浅拷贝和深拷贝，其实就是copy和retain的区别。copy是创建一个新对象，而retain是创建一个指针，引用计数加1.`copy属性`表示两个对象内容相同，新对象的retain引用计数为1，和旧的对象毫无关系了。`retain属性`仅仅是复制了旧对象的指针，内容相同，但是retain引用计数为2.总体来说就是浅拷贝就是指针的复制，深拷贝是真正的创建了新对象。
####非集合对象的copy与mutableCopy方法
<p>这里指的是NSString,NSNumber等等一类的对象。
<pre><code>
NSString *string = @"origin";
NSString *stringCopy = [string copy];
NSMutableString *stringMCopy = [string mutableCopy];
//可以看到 stringCopy 和 string 的地址是一样，进行了指针拷贝；而 stringMCopy 的地址和 string 不一样，进行了内容拷贝；
</pre></code>
<p>再看下面的例子
<pre><code>
NSMutableString *string = [NSMutableString stringWithString: @"origion"];
NSString *stringCopy = [string copy];
NSMutableString *mStringCopy = [string copy];
NSMutableString *stringMCopy = [string mutableCopy];[mStringCopy appendString:@"mm"];//error
[string appendString:@" origion!"];
[stringMCopy appendString:@"!!"];
//以上四个NSString对象所分配的内存都是不一样的。但是对于mStringCopy其实是个imutable对象，所以上述会报错。
对于系统的非容器类对象，我们可以认为，如果对一不可变对象复制，copy是指针复制（浅拷贝）和mutableCopy就是对象复制（深拷贝）。如果是对可变对象复制，都是深拷贝，但是copy返回的对象是不可变的。
</pre></code>

<p>
####集合对象的深拷贝和浅拷贝
指NSArray，NSDictionary等。对于容器类本身，上面讨论的结论也是适用的，需要探讨的是复制后容器内对象的变化。 
<p>
<pre><code>
 //copy返回不可变对象，mutablecopy返回可变对象
 NSArray *array1 = [NSArray arrayWithObjects:@"a",@"b",@"c",nil];
 NSArray *arrayCopy1 = [array1 copy];
 //arrayCopy1是和array同一个NSArray对象（指向相同的对象），包括array里面的元素也是指向相同的指针
    NSLog(@"array1 retain count: %d",[array1 retainCount]);
    NSLog(@"array1 retain count: %d",[arrayCopy1 retainCount]);
 NSMutableArray *mArrayCopy1 = [array1 mutableCopy];
 //mArrayCopy1是array1的可变副本，指向的对象和array1不同，但是其中的元素和array1中的元素指向的是同一个对象。mArrayCopy1还可以修改自己的对象
[mArrayCopy1 addObject:@"de"];
[mArrayCopy1 removeObjectAtIndex:0];
//array1和arrayCopy1是指针复制，而mArrayCopy1是对象复制，mArrayCopy1还可以改变期内的元素：删除或添加。但是注意的是，容器内的元素内容都是指针复制。
</pre></code>
<p>下一个例子
<pre><code>
NSArray *mArray1 = [NSArray arrayWithObjects:[NSMutableString stringWithString:@"a"],@"b",@"c",nil];
NSArray *mArrayCopy2 = [mArray1 copy];
NSLog(@"mArray1 retain count: %d",[mArray1 retainCount]);
NSMutableArray *mArrayMCopy1 = [mArray1 mutableCopy];
NSLog(@"mArray1 retain count: %d",[mArray1 retainCount]);
//mArrayCopy2,mArrayMCopy1和mArray1指向的都是不一样的对象，但是其中的元素都是一样的对象——同一个指针
//一下做测试
NSMutableString *testString = [mArray1 objectAtIndex:0];
//testString = @"1a1";
//这样会改变testString的指针，其实是将@“1a1”临时对象赋给了testString
[testString appendString:@" tail"];
//这样以上三个数组的首元素都被改变了
//由此可见，对于容器而言，其元素对象始终是指针复制。如果需要元素对象也是对象复制，就需要实现深拷贝。
</pre></code>
<p>深拷贝
<pre><code>
NSArray *array = [NSArray arrayWithObjects:[NSMutableString stringWithString:@"first"],[NSStringstringWithString:@"b"],@"c",nil];
NSArray *deepCopyArray=[[NSArray alloc] initWithArray: array copyItems: YES];
NSArray* trueDeepCopyArray = [NSKeyedUnarchiver unarchiveObjectWithData:
[NSKeyedArchiver archivedDataWithRootObject: array]];
//trueDeepCopyArray是完全意义上的深拷贝，而deepCopyArray则不是，对于deepCopyArray内的不可变元素其还是指针复制。或者我们自己实现深拷贝的方法。因为如果容器的某一元素是不可变的，那你复制完后该对象仍旧是不能改变的，因此只需要指针复制即可。除非你对容器内的元素重新赋值，否则指针复制即已足够。
</pre></code>
####自定义对象
如果是我们定义的对象，那么我们自己要实现NSCopying,NSMutableCopying这样就能调用copy和mutablecopy了。
<pre><code>
@interface MyObj : NSObject<NSCopying,NSMutableCopying>
{
         NSMutableString *name;
         NSString *imutableStr;
         int age;
}
@property (nonatomic, retain) NSMutableString *name;
@property (nonatomic, retain) NSString *imutableStr;
@property (nonatomic) int age;
@end
@implementation MyObj
@synthesize name;
@synthesize age;
@synthesize imutableStr;
- (id)init
{
         if (self = [super init])
         {
                   self.name = [[NSMutableString alloc]init];
                   self.imutableStr = [[NSString alloc]init];
                   age = -1;
         }
         return self;
}
- (void)dealloc
{
         [name release];
         [imutableStr release];
         [super dealloc];
}
- (id)copyWithZone:(NSZone *)zone
{
         MyObj *copy = [[[self class] allocWithZone:zone] init];
         copy->name = [name copy];
         copy->imutableStr = [imutableStr copy];
//       copy->name = [name copyWithZone:zone];;
//       copy->imutableStr = [name copyWithZone:zone];//
         copy->age = age;
         return copy;
}
- (id)mutableCopyWithZone:(NSZone *)zone
{
         MyObj *copy = NSCopyObject(self, 0, zone);
         copy->name = [self.name mutableCopy];
         copy->age = age;
         return copy;
}
</pre></code>

##See You