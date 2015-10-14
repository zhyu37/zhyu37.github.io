---
layout: post
title: "ios本地时间的运用NSDate"
date: 2015-10-15 00:06:15 +0800
comments: true
categories: 
---
====
##ios本地时间的运用NSDate
>看来NSDate之后感慨大一时候，在ACM中写敢于时间的问题，想想感觉当时太单纯，哈哈。

<p>NSDate的初始化</p>

<pre><code>
//获取当前日期
NSDate *now = [NSDate date];

//如果想获取几天后的时间，如果放在大一我还在傻乎乎的判断每个月有多少天哪一年是闰年，当然傻乎乎的时光是美好的。。。。。
NSTimeInterval day = 24 * 60 * 60; 

NSDate *will = [[NSDate alloc] initWithTimeInterval:day sinceDate:[NSDate date]];

//如果出现了 时区的问题
NSTimeZone *zone = [NSTimeZone systemTimeZone];
         
NSInteger interval = [zone secondsFromGMTForDate: date];
         
NSDate *localDate = [date  dateByAddingTimeInterval: interval];

</pre></code>

<p>当然NSDate -> NSString 的转换也是不可或缺的</p>

<pre><code>
NSDateFormatter *dateFormatter =[[NSDateFormatter alloc] init];
 
// 设置日期格式
[dateFormatter setDateFormat:@"年月日 YYYY/mm/dd 时间 hh:mm:ss"];
         
NSString *dateString = [dateFormatter stringFromDate:[NSDate date]];
         
// 打印结果：dateString = 年月日 2013/10/16 时间 05:15:43
        NSLog(@"dateString = %@",dateString);
         
// 设置日期格式
[dateFormatter setDateFormat:@"YYYY-MM-dd"];
         
NSString *year = [dateFormatter stringFromDate:[NSDate date]];
         
// 打印结果：年月日 year = 2013-08-16
NSLog(@"年月日 year = %@",year);
         
// 设置时间格式
[dateFormatter setDateFormat:@"hh:mm:ss"];
         
NSString *time = [dateFormatter stringFromDate:[NSDate date]];

// 打印结果：时间 time = 05:15:43
NSLog(@"时间 time = %@",time);
</pre></code>

<p>两个时间的比较</p>

<pre><code>
// 当前时间
NSDate *currentDate = [NSDate date];
         
// 比当前时间晚一个小时的时间
NSDate *laterDate = [[NSDate alloc] initWithTimeInterval:60*60 sinceDate:[NSDate date]];
         
// 比当前时间早一个小时的时间
NSDate *earlierDate = [[NSDate alloc] initWithTimeInterval:-60*60 sinceDate:[NSDate date]];
         
// 比较哪个时间迟
if ([currentDate laterDate:laterDate]) {
// 打印结果：current-2013-08-16 09:25:54 +0000比later-2013-08-16 10:25:54 +0000晚
NSLog(@"current-%@比later-%@晚",currentDate,laterDate);
}
         
// 比较哪个时间早
if ([currentDate earlierDate:earlierDate]) {
// 打印结果：current-2013-08-16 09:25:54 +0000 比 earlier-2013-08-16 08:25:54 +0000 
NSLog(@"current-%@ 比 earlier-%@ 早",currentDate,earlierDate);
}
         
if([currentDatecompare:earlierDate]==NSOrderedDescending) 
{
// 打印结果
NSLog(@"current 晚");
}
if ([currentDate compare:currentDate]==NSOrderedSame) 
{
// 打印结果
NSLog(@"时间相等");
}
if ([currentDate compare:laterDate]==NSOrderedAscending) {
// 打印结果
NSLog(@"current 早");
}
</pre></code>

##See You