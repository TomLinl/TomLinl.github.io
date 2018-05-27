---
title: 支付宝APP支付服务端接入(PHP)
date: 2018-04-07 16:04:00
tags: 
    - 支付宝支付
    - PHP
---
第一步：创建应用并获取APPID
要在您的应用中接入支付宝App支付能力，需要通过创建应用的方式接入蚂蚁相关接口并进行开发，在创建应用后即生成应用的标识APPID，使用支付宝账号登录开放平台后，在“我的应用”中查看APPID。
<!--more-->
第二步：配置应用
添加app支付功能——>签约——>配置密钥
第三步：集成和开发
集成服务端SDK
{% asset_img sdk.png %}
获取PHP版资源SDK要注意开发环境最好达到官方建议的环境要求，否则集成过程中要对SDK作调整来适配环境。
PHP服务端SDK生成APP支付订单信息示例
```php
$aop = new AopClient;
$aop->gatewayUrl = "https://openapi.alipay.com/gateway.do";
$aop->appId = "app_id";
$aop->rsaPrivateKey = '请填写开发者私钥去头去尾去回车，一行字符串';
$aop->format = "json";
$aop->charset = "UTF-8";
$aop->signType = "RSA2";
$aop->alipayrsaPublicKey = '请填写支付宝公钥，一行字符串';
//实例化具体API对应的request类,类名称和接口名称对应,当前调用接口名称：alipay.trade.app.pay
$request = new AlipayTradeAppPayRequest();
//SDK已经封装掉了公共参数，这里只需要传入业务参数
$bizcontent = "{\"body\":\"我是测试数据\"," 
                . "\"subject\": \"App支付测试\","
                . "\"out_trade_no\": \"20170125test01\","
                . "\"timeout_express\": \"30m\"," 
                . "\"total_amount\": \"0.01\","
                . "\"product_code\":\"QUICK_MSECURITY_PAY\""
                . "}";
$request->setNotifyUrl("商户外网可以访问的异步地址");
$request->setBizContent($bizcontent);
//这里和普通的接口调用不同，使用的是sdkExecute
$response = $aop->sdkExecute($request);
//htmlspecialchars是为了输出到页面时防止被浏览器将关键参数html转义，实际打印到日志以及http传输不会有这个问题
echo htmlspecialchars($response);//就是orderString 可以直接给客户端请求，无需再做处理。
```
PHP服务端验证异步通知信息参数示例
```cpp
$aop = new AopClient;
$aop->alipayrsaPublicKey = '请填写支付宝公钥，一行字符串';
$flag = $aop->rsaCheckV1($_POST, NULL, "RSA2");

```