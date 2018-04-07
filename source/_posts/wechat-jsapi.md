---
title: 微信支付JSAPI(PHP)
date: 2018-04-07 17:46:29
tags: 微信支付
---
公众号支付是用户在微信中打开商户的H5页面，商户在H5页面通过调用微信支付提供的JSAPI接口调起微信支付模块完成支付。适用于微信浏览器内完成微信支付。配置参考[官方文档](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_3)
<!--more-->
实现方法
统一下单示例
```javascript
        $tools = new JsApiPay();
        $openId = $tools->GetOpenid();//获取openid
        $input = new WxPayUnifiedOrder();
        $input->SetBody($name);//商品描述
        $input->SetOut_trade_no($out_trade_no);//订单号
        $input->SetTotal_fee($amount);//订单总金额，单位分
        $input->SetTime_start(date("YmdHis"));
        $input->SetTime_expire(date("YmdHis", time() + 600));
        $input->SetNotify_url("回调域名");
        $input->SetTrade_type("JSAPI");//交易类型
        $input->SetOpenid($openId);
        $order = WxPayApi::unifiedOrder($input);//统一下单接口
        $jsApiParameters = $tools->GetJsApiParameters($order);
```
调起微信支付示例
```javascript
<script type="text/javascript">
    //调用微信JS api 支付
    function jsApiCall()
    {
        WeixinJSBridge.invoke(
            'getBrandWCPayRequest',{
                "appId":"wx2421b1c4370ec43b",     //公众号名称，由商户传入     
                "timeStamp":"1395712654",         //时间戳，自1970年以来的秒数     
                "nonceStr":"e61463f8efa94090b1f366cccfbbb444", //随机串     
                "package":"prepay_id=u802345jgfjsdfgsdg888",     
                "signType":"MD5",         //微信签名方式：     
                "paySign":"70EA570631E4BB79628FBCA90534C63FF7FADD89" //微信签名
        },
        function(res){
            if(res.err_msg == "get_brand_wcpay_request:ok"){
                if(res.err_msg == "get_brand_wcpay_request:ok" ) {}     // 使用以上方式判断前端返回,微信团队郑重提示：res.err_msg将在用户支付成功后返回    ok，但并不保证它绝对可靠。 
            }
        }
    );
    }

    function callpay()
    {
        if (typeof WeixinJSBridge == "undefined"){
            if( document.addEventListener ){
                document.addEventListener('WeixinJSBridgeReady', jsApiCall, false);
            }else if (document.attachEvent){
                document.attachEvent('WeixinJSBridgeReady', jsApiCall);
                document.attachEvent('onWeixinJSBridgeReady', jsApiCall);
            }
        }else{
            jsApiCall();
        }
    }

    $(function(){
        callpay();
    })
</script>
```
回调处理示列
```javascript
$WxPay = new \WxPayResults();
header('Content-type: text/xml');
$returnResult = $GLOBALS['HTTP_RAW_POST_DATA']; //接收微信发送的信息
$res = $WxPay::Init($returnResult);
if(res['result_code'] == 'SUCCESS'){
/*
* 业务处理
*/
$success = array('return_code' => 'SUCCESS', 'return_msg' => 'OK');
exit(ToXml($success));//转成xml通知给微信
}

```