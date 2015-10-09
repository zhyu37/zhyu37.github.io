---
layout: post
title: "菜鸟的努力-ios的HTTP搭建"
date: 2015-10-09 19:27:16 +0800
comments: true
categories: 
---
写接口  当然要用AFNetWorking

1.  既然想通过HTTP 连接服务器

	需要写一个HTTPClient的文件  用来写发送get请求 和 发送post请求

	

	
+ (AFHTTPRequestOperation *)sendGetRequestWithPathString:(NSString *)pathString

	//在数据传输 之前 要进行暗号的 核对
 		tokened:(BOOL)tokened
		//想要传给服务器的数据

	params:(id)params
		//当传输 成功之后 执行的

	 success:(TDCHTTPRequestSuccessBlock)success
		//当传输  失败之后  执行的
		 failure:(TDCHTTPRequestFailureBlock)failure;
+ (AFHTTPRequestOperation *)sendPostRequestWithPathString:(NSString *)pathString
		 tokened:(BOOL)tokened
		 params:(id)params
		 paramType:(JHHTTPParamType)paramType
		 success:(TDCHTTPRequestSuccessBlock)success
		 failure:(TDCHTTPRequestFailureBlock)failure;

		

用到一个  JHHTTPClient   第三方的HTTPClient
网盘链接   http://pan.baidu.com/s/1c0qcnD2

HTTPClient.m 
+ (AFHTTPRequestOperation *)sendGetRequestWithPathString:(NSString *)pathString tokened:(BOOL)tokened params:(id)params success:(TDCHTTPRequestSuccessBlock)success failure:(TDCHTTPRequestFailureBlock)failure
	{

//#define kHTTPServer  (@"http://183.221.242.75:8089/APPInterface/")

return [JHHTTPClient sendBaseRequestWithBaseString:kHTTPServer
	//pathString =  @"user/Login.ashx?act=add_phone”;接口链接

   pathString:pathString
	//方法类型   Get Post  Put  Delete


methodType:JHHTTPMethodTypeGet
	//参数类型  Default Path KeyValue JSON XML

  paramType:JHHTTPParamTypeDefault
	//NSDictionary *params = @{@"phone": phone};  传递 接口所需要 的参数

   params:params                                          files:nil                                            headers:nil
                                           success:^(AFHTTPRequestOperation *operation, id response) {
	//这个 类是用来 保存 服务器 发送回来 除了数据 之外的 信息
	//{"Data":333409,"code":0,"Msg":"","num":0}
	//Data  为数据  Status 为

   TDCModel *model = [TDCModel objectWithKeyValues:response];
	//#define kHTTPSuccess (200) 

   if(kHTTPSuccess != model.code) {
                                                       if (failure) {

//
   failure(operation, [TDCError errorWithCode:model.code description:model.message]);

   }

   }else {
    if (success) {

//#define kHTTPData (@"Data")
                                                     success(operation, [response objectForKey:kHTTPData]);

}
 }
}
                                            failure:failure progress:nil];
}

+ (AFHTTPRequestOperation *)sendPostRequestWithPathString:(NSString *)pathString tokened:(BOOL)tokened params:(id)params paramType:(JHHTTPParamType)paramType success:(TDCHTTPRequestSuccessBlock)success failure:(TDCHTTPRequestFailureBlock)failure
	{
    


    return [JHHTTPClient sendBaseRequestWithBaseString:kHTTPServer
                                     pathString:pathString
                              methodType:JHHTTPMethodTypePost
  paramType:paramType
   params:params
    files:nil
   headers:nil
                                             success:^(AFHTTPRequestOperation *operation, id response) {
  TDCModel *model = [TDCModel objectWithKeyValues:response];

 if (model.code == 1 || model.code == 0) {

 if (success) {
                                            success(operation, [response objectForKey:kHTTPData]);

 }

  }else {

  if (failure) {
 if ([model.message isEqualToString:@""])
{
                                                    failure(operation, [TDCError errorWithCode:model.code description:@"服务器内部错误"]);

}
                                                       else
{
                                                            failure(operation, [TDCError errorWithCode:model.code description:model.message]);
 }
}
}
}
                                          failure:^(AFHTTPRequestOperation *operation, NSError *error) {
if (3840 == error.code) {
error = [TDCError errorWithCode:TDCErrorCodeServerInvalid description:kStringServerException];
}
if (failure) {
                                            failure(operation, error);
}
} progress:nil];
                                                   
}	


