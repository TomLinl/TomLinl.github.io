---
title: 微信APP支付服务端接入(PHP)
date: 2018-04-07 16:01:16
tags: 微信支付
---
第一步：申请微信APP支付
微信开放平台是商户APP接入微信支付开放接口的申请入口，通过此平台可申请微信APP支付。
<!--more-->
第二步：配置商户支付密钥
微信商户平台是微信支付相关的商户功能集合，登录微信商户平台后按以下步骤操作：
1.点击帐户中心——操作证书——安装安全控件——安装操作证书（要验证手机号码）
2.点击帐户中心——API安全——设置API密钥——确认——新密钥——手机验证码——登录密码
第三步：集成和开发
集成服务端SDK
{% asset_img sdk.png %}
微信APP支付统一下单示例
```javascript
$input = new WxPayUnifiedOrder();
$input->SetBody("test"); //商品描述
$input->SetOut_trade_no("商户订单号");
$input->SetTotal_fee("1"); //订单总金额，注意单位为分
$input->SetTime_start(date("YmdHis"));
$input->SetTime_expire(date("YmdHis", time() + 600));
$input->SetNotify_url("商户外网可以访问的异步地址");
$input->SetTrade_type("APP");
$order = WxPayApi::unifiedOrder($input); //调用统一下单接口

```
调起支付
商户服务器生成支付订单，先调用【统一下单API】生成预付单，获取到prepay_id后将参数再次签名传输给APP发起支付。以下是调起微信支付的关键代码：
```javascript
$order_data = $WxPayApi->GetAppParameters($order); //调起支付所需的请求参数
```
GetAppParameters()是外加的代码，微信SDK中没有这个方法，下面是在微信SDK中代码APP支付补充部分。
在WxPay.Api.php中补充代码：
```javascript
/**
 *
 * 获取App支付的参数
 * @param array $UnifiedOrderResult 统一支付接口返回的数据
 * @throws WxPayException
 * 
 * @return $parameters
 */
public function GetAppParameters($UnifiedOrderResult)
{
    if(!array_key_exists("appid", $UnifiedOrderResult)
        || !array_key_exists("prepay_id", $UnifiedOrderResult)
        || $UnifiedOrderResult['prepay_id'] == ""
        || !array_key_exists("mch_id", $UnifiedOrderResult)
        || $UnifiedOrderResult['mch_id'] == "")
    {
        throw new WxPayException("参数错误");
    }
    $app = new WxPayAppPay();
    $app->SetAppid($UnifiedOrderResult["appid"]);
    $app->SetPrepayId($UnifiedOrderResult["prepay_id"]);
    $app->SetPartnerId($UnifiedOrderResult['mch_id']);
    $timeStamp = time();
    $app->SetTimeStamp("$timeStamp");
    $app->SetNonceStr(WxPayApi::getNonceStr());
    $app->SetPackage("Sign=WXPay");
    $app->SetPaySign($app->MakeSign());
    $parameters = $app->GetValues();
    return $parameters;
}
```
在WxPay.Data.php中补充代码：
```javascript
/**
 *
 * 提交App输入对象
 * @author lin
 *
 */
class WxPayAppPay extends WxPayDataBase
{
    /**
     * 设置微信分配的公众账号ID
     * @param string $value
     **/
    public function SetAppid($value)
    {
        $this->values['appid'] = $value;
    }
    /**
     * 设置预支付交易会话ID
     * @param string $value
     **/
    public function SetPrepayId($value)
    {
        $this->values['prepayid'] = $value;
    }

    /**
     * 设置商户号ID
     * @param string $value
     **/
    public function SetPartnerId($value)
    {
        $this->values['partnerid'] = $value;
    }

    /**
     * 设置支付时间戳
     * @param string $value
     **/
    public function SetTimeStamp($value)
    {
        $this->values['timestamp'] = $value;
    }
    /**
     * 随机字符串
     * @param string $value
     **/
    public function SetNonceStr($value)
    {
        $this->values['noncestr'] = $value;
}

    /**
     * 设置订单详情扩展字符串
     * @param string $value
     **/
    public function SetPackage($value)
    {
        $this->values['package'] = $value;
    }

    /**
     * 设置签名方式
     * @param string $value
     **/
    public function SetPaySign($value)
    {
        $this->values['sign'] = $value;
    }
}
```
回调处理的示列
```javascript
$WxPay = new \WxPayResults();
header('Content-type: text/xml');
$returnResult = $GLOBALS['HTTP_RAW_POST_DATA']; //接收微信发送的信息
$res = $WxPay::Init($returnResult);
if(res['result_code'] == 'SUCCESS'){
/*
* 业务处理
*/
$success = "<xml><return_code><![CDATA[SUCCESS]]></return_code><return_msg><![CDATA[OK]]></return_msg></xml>";
exit($success);//回调结果通知
}
```