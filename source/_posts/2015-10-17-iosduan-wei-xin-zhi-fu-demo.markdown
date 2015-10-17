---
layout: post
title: "ios端 微信支付Demo"
date: 2015-10-17 09:08:37 +0800
comments: true
categories: 
---
======
##ios端 微信支付
>最近项目上用到了微信支付，看官方文档我也不想说什么，只是没有友盟、环信之类写的清楚，Demo居然 还是一个学C的ios实习生写的我也是醉了。所以我精简了一些一般常用的，希望大家指点不足。

<p>准备工作  从官方文档下载SDK</p>
* SDKExpot(里面包含三个文件)
* ApiXml.h/.m
* WXUtil.h/.m

>把这些文件导入到工程里，然后我们再里面创建一个工具类

<p>WeChatPayManager.h</p>
<pre><code>
#import Foundation/Foundation.h
#import "WXApi.h"
#import "WXUtil.h"
#import "ApiXml.h"

// 账号帐户资料
// 更改商户把相关参数后可测试   因为涉及到隐私所以就不公布了  哈哈
//appid  公众账号ID
#define APP_ID          @""  
//appsecret,看起来好像没用     
#define APP_SECRET      @""  
//商户号，mch_id
#define MCH_ID          @""
//商户API密钥，填写相应参数
#define PARTNER_ID      @""
//支付结果回调页面
#define NOTIFY_URL      @"http://wxpay.weixin.qq.com/pub_v2/pay/notify.v2.php"
//获取服务器端支付数据地址（商户自定义）(在这里，签名算法直接放在APP端，故不需要自定义)
#define SP_URL          @"http://wxpay.weixin.qq.com/pub_v2/app/app_pay.php"

@interface TDUWeChatPayManager : NSObject
+ (instancetype)sharedTDUWeChatPayManager;

//预支付网关url地址
@property (nonatomic,strong) NSString* payUrl;
@property (nonatomic,strong) NSString* chatUrl;

//debug信息
@property (nonatomic,strong) NSMutableString *debugInfo;
@property (nonatomic,assign) NSInteger lastErrCode;//返回的错误码

//商户关键信息
@property (nonatomic,strong) NSString *appId,*mchId,*spKey;

//获取当前的debug信息
-(NSString *) getDebugInfo;

//获取预支付订单信息（核心是一个prepayID）
- (NSMutableDictionary*)getPrepayWithOrder
 Name:(NSString*)name
price:(NSString*)price
 device:(NSString*)device
 norder:(NSString*)orderno;

//查询订单  返回 所需要的信息
- (NSString*)getOrderIDorderno:(NSString*)orderno;

//这里就是调用支付的方法  name 为你要传的商品名称  price 支付的金额注意金额 单位 是分  
//orderno  是一组随机数  就是文档里的  随机字符串	nonce_str 我这里使用是因为 有的地方 要用
- (void)wxPayWithOrderName:(NSString*)name price:(NSString*)price orderno:(NSString*)orderno;

@end

</pre></code>

<p>WeChatPayManager.m</p>

<pre><code>
#import "TDUWeChatPayManager.h"

@interface TDUWeChatPayManager()

@property (nonatomic, copy) NSString *orderno;

@end
static TDUWeChatPayManager *manager = nil;
@implementation TDUWeChatPayManager

+ (instancetype)sharedTDUWeChatPayManager
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        manager = [[TDUWeChatPayManager alloc] initWithAppID:APP_ID mchID:MCH_ID spKey:PARTNER_ID];
    });
    return manager;
}


//初始化函数
-(id)initWithAppID:(NSString*)appID mchID:(NSString*)mchID spKey:(NSString*)key
{
    self = [super init];
    if(self)
    {
        //初始化私有参数，主要是一些和商户有关的参数
        self.payUrl    = @"https://api.mch.weixin.qq.com/pay/unifiedorder";
        self.chatUrl = @"https://api.mch.weixin.qq.com/pay/orderquery";
        if (self.debugInfo == nil){
            self.debugInfo  = [NSMutableString string];
        }
        [self.debugInfo setString:@""];
        self.appId = appID;//微信分配给商户的appID
        self.mchId = mchID;//
        self.spKey = key;//商户的密钥
    }
    return self;
}

//获取debug信息
-(NSString*) getDebugInfo
{
    NSString *res = [NSString stringWithString:self.debugInfo];
    [self.debugInfo setString:@""];
    return res;
}

//创建package签名
-(NSString*) createMd5Sign:(NSMutableDictionary*)dict
{
    NSMutableString *contentString  =[NSMutableString string];
    NSArray *keys = [dict allKeys];
    //按字母顺序排序
    NSArray *sortedArray = [keys sortedArrayUsingComparator:^NSComparisonResult(id obj1, id obj2) {
        return [obj1 compare:obj2 options:NSNumericSearch];
    }];
    //拼接字符串
    for (NSString *categoryId in sortedArray) {
        if (   ![[dict objectForKey:categoryId] isEqualToString:@""]
            && ![categoryId isEqualToString:@"sign"]
            && ![categoryId isEqualToString:@"key"]
            )
        {
            [contentString appendFormat:@"%@=%@&", categoryId, [dict objectForKey:categoryId]];
        }
        
    }
    //添加key字段
    [contentString appendFormat:@"key=%@", self.spKey];
    //得到MD5 sign签名
    NSString *md5Sign =[WXUtil md5:contentString];
    
    //输出Debug Info
    [self.debugInfo appendFormat:@"MD5签名字符串：\n%@\n\n",contentString];
    
    return md5Sign;
}

//获取package带参数的签名包
-(NSString *)genPackage:(NSMutableDictionary*)packageParams
{
    NSString *sign;
    NSMutableString *reqPars=[NSMutableString string];
    //生成签名
    sign        = [self createMd5Sign:packageParams];
    //生成xml的package
    NSArray *keys = [packageParams allKeys];
    [reqPars appendString:@"<xml>\n"];
    for (NSString *categoryId in keys) {
        [reqPars appendFormat:@"<%@>%@</%@>\n",categoryId, [packageParams objectForKey:categoryId],categoryId];
    }
    [reqPars appendFormat:@"<sign>%@</sign>\n</xml>", sign];
    
    return [NSString stringWithString:reqPars];
}

//提交预支付
-(NSString *)sendPrepay:(NSMutableDictionary *)prePayParams
{
    NSString *prepayid = nil;
    
    //获取提交支付
    NSString *send      = [self genPackage:prePayParams];
    
    //输出Debug Info
    [self.debugInfo appendFormat:@"API链接:%@"
     , self.payUrl];
    [self.debugInfo appendFormat:@"发送的xml:%@"
     , send];
    
    //发送请求post xml数据
    NSData *res = [WXUtil httpSend:self.payUrl method:@"POST" data:send];
    
    //输出Debug Info
    [self.debugInfo appendFormat:@"服务器返回：%@"
     
     ,[[NSString alloc] initWithData:res encoding:NSUTF8StringEncoding]];
    
    XMLHelper *xml  = [[XMLHelper alloc] init];
    
    //开始解析
    [xml startParse:res];
    
    NSMutableDictionary *resParams = [xml getDict];
    
    //判断返回
    NSString *return_code   = [resParams objectForKey:@"return_code"];
    NSString *result_code   = [resParams objectForKey:@"result_code"];
    if ( [return_code isEqualToString:@"SUCCESS"] )
    {
        //生成返回数据的签名
        NSString *sign      = [self createMd5Sign:resParams ];
        NSString *send_sign =[resParams objectForKey:@"sign"] ;
        
        //验证签名正确性
        if( [sign isEqualToString:send_sign]){
            if( [result_code isEqualToString:@"SUCCESS"]) {
                //验证业务处理状态
                prepayid    = [resParams objectForKey:@"prepay_id"];
                return_code = 0;
                
                [self.debugInfo appendFormat:@"获取预支付交易标示成功！"
                 ];
            }
        }else{
            self.lastErrCode = 1;
            [self.debugInfo appendFormat:@"gen_sign=%@_sign=%@"
             ,sign,send_sign];
            [self.debugInfo appendFormat:@"服务器返回签名验证错误！！！"
             ];
        }
    }else{
        self.lastErrCode = 2;
        [self.debugInfo appendFormat:@"接口返回错误！！！"
         ];
    }
    
    return prepayid;
}

- (NSMutableDictionary*)getPrepayWithOrderName:(NSString*)name
                                         price:(NSString*)price
                                        device:(NSString*)device
                                        norder:(NSString*)orderno
{
    //订单标题，展示给用户
    NSString* orderName = name;
    //订单金额,单位（分）
    NSString* orderPrice = price;//以分为单位的整数
    //支付设备号或门店号
    NSString* orderDevice = device;
    //支付类型，固定为APP
    NSString* orderType = @"APP";
    //发器支付的机器ip,暂时没有发现其作用
    NSString* orderIP = @"196.168.7.96";
    
    //================================
    //预付单参数订单设置
    //================================
    srand( (unsigned)time(0) );
    NSString *noncestr  = [NSString stringWithFormat:@"%d", rand()];
    self.orderno  = [NSString stringWithFormat:@"%ld",time(0)];
    NSMutableDictionary *packageParams = [NSMutableDictionary dictionary];
    
    [packageParams setObject: self.appId        forKey:@"appid"];       //开放平台appid
    [packageParams setObject: self.mchId        forKey:@"mch_id"];      //商户号
    [packageParams setObject: [TDUGlobal sharedTDUGlobal].deviceToken        forKey:@"device_info"]; //支付设备号或门店号
    [packageParams setObject: noncestr          forKey:@"nonce_str"];   //随机串
    [packageParams setObject: orderType            forKey:@"trade_type"];  //支付类型，固定为APP
    [packageParams setObject: orderName        forKey:@"body"];        //订单描述，展示给用户
    [packageParams setObject: NOTIFY_URL        forKey:@"notify_url"];  //支付结果异步通知
    [packageParams setObject: orderno           forKey:@"out_trade_no"];//商户订单号
    [packageParams setObject: orderIP    forKey:@"spbill_create_ip"];//发器支付的机器ip
    [packageParams setObject: orderPrice       forKey:@"total_fee"];       //订单金额，单位为分
    
    //获取prepayId（预支付交易会话标识）
    NSString *prePayid;
    prePayid = [self sendPrepay:packageParams];
    
    if(prePayid == nil)
    {
        [self.debugInfo appendFormat:@"获取prepayid失败！"
         ];
        return nil;
    }
    
    //获取到prepayid后进行第二次签名
    NSString    *package, *time_stamp, *nonce_str;
    //设置支付参数
    time_t now;
    time(&now);
    time_stamp  = [NSString stringWithFormat:@"%ld", now];
    nonce_str = [WXUtil md5:time_stamp];
    //重新按提交格式组包，微信客户端暂只支持package=Sign=WXPay格式，须考虑升级后支持携带package具体参数的情况
    //package       = [NSString stringWithFormat:@Sign=%@,package];
    package         = @"Sign=WXPay";
    //第二次签名参数列表
    NSMutableDictionary *signParams = [NSMutableDictionary dictionary];
    [signParams setObject: self.appId  forKey:@"appid"];
    [signParams setObject: self.mchId  forKey:@"partnerid"];
    [signParams setObject: nonce_str    forKey:@"noncestr"];
    [signParams setObject: package      forKey:@"package"];
    [signParams setObject: time_stamp   forKey:@"timestamp"];
    [signParams setObject: prePayid     forKey:@"prepayid"];
    
    //生成签名
    NSString *sign  = [self createMd5Sign:signParams];
    
    //添加签名
    [signParams setObject: sign         forKey:@"sign"];
    
    [self.debugInfo appendFormat:@"第二步签名成功，sign＝%@"
     ,sign];
    
    //返回参数列表
    return signParams;
}

//官方文档中的  统一下单
- (void)wxPayWithOrderName:(NSString*)name price:(NSString*)price orderno:(NSString*)orderno
{
    //创建支付签名对象 && 初始化支付签名对象
    TDUWeChatPayManager *wxpayManager = [TDUWeChatPayManager sharedTDUWeChatPayManager];
    
    //获取到实际调起微信支付的参数后，在app端调起支付
    //生成预支付订单，实际上就是把关键参数进行第一次加密。
    NSString* device = @"013467007045764";
    NSMutableDictionary *dict = [wxpayManager getPrepayWithOrderName:name price:price device:device norder:orderno];
    
    if(dict == nil){
        //错误提示
        NSString *debug = [wxpayManager getDebugInfo];
        return;
    }
    
    NSMutableString *stamp  = [dict objectForKey:@"timestamp"];
    
    //调起微信支付
    PayReq* req             = [[PayReq alloc] init];
    req.openID              = [dict objectForKey:@"appid"];
    req.partnerId           = [dict objectForKey:@"partnerid"];
    req.prepayId            = [dict objectForKey:@"prepayid"];
    req.nonceStr            = [dict objectForKey:@"noncestr"];
    req.timeStamp           = stamp.intValue;
    req.package             = [dict objectForKey:@"package"];
    req.sign                = [dict objectForKey:@"sign"];
    
    //        BOOL flag = [WXApi sendReq:req];
    [WXApi sendReq:req];
}

//官方文档中的  查询订单
- (NSString*)getOrderIDorderno:(NSString*)orderno
{
    srand( (unsigned)time(0) );
    NSString *noncestr  = [NSString stringWithFormat:@"%d", rand()];
    NSMutableDictionary *packageParams = [NSMutableDictionary dictionary];
    
    [packageParams setObject: self.appId        forKey:@"appid"];       //开放平台appid
    [packageParams setObject: self.mchId        forKey:@"mch_id"];      //商户号
    [packageParams setObject: orderno           forKey:@"out_trade_no"];//商户订单号
    [packageParams setObject: noncestr          forKey:@"nonce_str"];   //随机串
    
    NSString *prePayid;
    prePayid = [self querySendPrepay:packageParams];
    
    return prePayid;
    
}

//提交预支付
-(NSString *)querySendPrepay:(NSMutableDictionary *)prePayParams
{
    NSString *prepayid = nil;
    
    //获取提交支付
    NSString *send      = [self genPackage:prePayParams];
    
    //输出Debug Info
    [self.debugInfo appendFormat:@"API链接:%@"
     , self.payUrl];
    [self.debugInfo appendFormat:@"发送的xml:%@"
     , send];
    
    //发送请求post xml数据
    NSData *res = [WXUtil httpSend:self.chatUrl method:@"POST" data:send];
    
    //输出Debug Info
    [self.debugInfo appendFormat:@"服务器返回：%@"
     
     ,[[NSString alloc] initWithData:res encoding:NSUTF8StringEncoding]];
    
    XMLHelper *xml  = [[XMLHelper alloc] init];
    
    //开始解析
    [xml startParse:res];
    
    NSMutableDictionary *resParams = [xml getDict];
    
    //判断返回
    NSString *return_code   = [resParams objectForKey:@"return_code"];
    NSString *result_code   = [resParams objectForKey:@"result_code"];
    if ( [return_code isEqualToString:@"SUCCESS"] )
    {
        if ([result_code isEqualToString:@"SUCCESS"]) {
            return resParams[@"transaction_id"];
        }
    }
    
    return nil;
}
@end

</pre></code>

<p>AppDelegate.m</p>

<pre><code>
#import "WXApi.h"
@interface AppDelegate ()WXApiDelegate

#pragma mark - WXApiDelegate

//当支付完成之后  之后调用
-(void) onResp:(BaseResp*)resp
{
    //启动微信支付的response
    NSString *strMsg = [NSString stringWithFormat:@"errcode:%d", resp.errCode];
    if([resp isKindOfClass:[PayResp class]]){
        
        //支付返回结果，实际支付结果需要去微信服务器端查询
        switch (resp.errCode) {
            case 0:
            //通过 通知调用 支付界面的  上传服务器的接口 
                [[NSNotificationCenter defaultCenter] postNotificationName:TDUWeChatPaySuccessNotification object:nil];
                strMsg = @"支付结果：成功！";
                break;
            case -1:
                strMsg = @"支付结果：失败！";
                break;
            case -2:
                strMsg = @"用户已经退出支付！";
                break;
            default:
                strMsg = [NSString stringWithFormat:@"支付结果：失败！retcode = %d, retstr = %@", resp.errCode,resp.errStr];
                break;
        }
    }
}


- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url
{
    return  [WXApi handleOpenURL:url delegate:self];
}

- (BOOL)application:(UIApplication *)application
                     openURL:(NSURL *)url
           sourceApplication:(NSString *)sourceApplication
                  annotation:(id)annotation
{
    BOOL isSuc = [WXApi handleOpenURL:url delegate:self];
    
    NSLog(@"url %@ isSuc %d",url,isSuc == YES ? 1 : 0);
    
    return  isSuc;
}

</pre></code>

<p>为了 适配io9  还要有一些改变 可以去 微信官方文档上 或者是 网上去找 ，然后就是 把 微信appid添加 到项目中  这个 Demo 中的一张图片上 解释了</p>

>基本上  就是这样  有不懂的 可以联系我QQ466096851  我上面的版本有的地方是为了 服务器要求上传订单号 ，但是 微信官方文档上面的 支付结果通知  说的返回的 根本没有 或者  是我没有找到，可以根据自己实际情况 做改变.

##See You

