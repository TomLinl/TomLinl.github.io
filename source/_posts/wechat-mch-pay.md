---
title: 微信支付企业付款(PHP)
date: 2018-04-07 16:56:01
tags: 
    - 微信支付
    - PHP
---
企业付款为企业提供付款至用户零钱的能力，支持通过API接口付款，或通过微信支付商户平台（pay.weixin.qq.com）网页操作付款。配置和说明可以参考[官方文档](https://pay.weixin.qq.com/wiki/doc/api/tools/mch_pay.php?chapter=14_1/) 
<!--more-->
实现方法
在微信官方SDK的WxPay.Api.php添加参考代码：
企业向个人付款
``` javascript 
    public function mchPay($params)
    {

      $url = "https://api.mch.weixin.qq.com/mmpaymkttransfers/promotion/transfers";
        //检测必填参数
        if($params["partner_trade_no"] == null) {
            throw new WxPayException("企业付款申请接口中，缺少必填参数partner_trade_no！");
        }else if($params["openid"] == null){
            throw new WxPayException("企业付款申请接口中，缺少必填参数openid！");
        }else if($params["check_name"] == null){
            throw new WxPayException("企业付款申请接口中，缺少必填参数check_name！");
        }else if($params["amount"] == null){
            throw new WxPayException("企业付款申请接口中，缺少必填参数amount！");
        }else if($params["desc"] == null){
            throw new WxPayException("企业付款申请接口中，缺少必填参数desc！");
        }
        $params["mch_appid"] = WxPayConfig::APPID();//公众账号ID
        $params["mchid"] = WxPayConfig::MCHID();//商户号
        $params["nonce_str"] = self::getNonceStr();//随机字符串
        $params['spbill_create_ip'] = $_SERVER['REMOTE_ADDR'] == '::1' ? '192.127.1.1' : $_SERVER['REMOTE_ADDR'];//获取IP

        $obj = new \WxPayDataBase();
        $params["sign"] = $this->MakeSign($params);//签名
        $xml = $this->arrayToXml($params);
        $response = self::postXmlCurl($xml, $url, true);
        $obj->FromXml($response);
        return $obj->GetValues();
    }
```
```javascript
    //array转xml
    public function arrayToXml($arr)
    {
        $xml = "<xml>";
        foreach ($arr as $key=>$val)
        {
            if (is_numeric($val))
            {
                $xml.="<".$key.">".$val."</".$key.">";

            }
            else
                $xml.="<".$key."><![CDATA[".$val."]]></".$key.">";
        }
        $xml.="</xml>";
        return $xml;
    }
```
```javascript
    //生成签名
    public function MakeSign($params)
    {
        //签名步骤一：按字典序排序参数
        ksort($params);
        $string = $this->ToUrlParams($params);
        //签名步骤二：在string后加入KEY
        $string = $string . "&key=".WxPayConfig::KEY();
        //签名步骤三：MD5加密
        $string = md5($string);
        //签名步骤四：所有字符转为大写
        $result = strtoupper($string);
        return $result;
    }

    public function ToUrlParams($params)
    {
        $buff = "";
        foreach ($params as $k => $v)
        {
            if($k != "sign" && $v != "" && !is_array($v)){
                $buff .= $k . "=" . $v . "&";
            }
        }

        $buff = trim($buff, "&");
        return $buff;
    }
```
调用示例：
```javascript 
    $mchPay = new \WxPayApi();  
    $amount = '付款金额';//企业付款金额，单位“分”
    $desc = '企业付款';//描述
    $partner_trade_no = '订单号';
    $openid = '用户oppenid';
    $params = array(
        'partner_trade_no' => $partner_trade_no,
        'openid' => $openid,
        'check_name' => 'NO_CHECK',//不校验姓名
        'amount' => $amount,
        'desc' => $desc,
    );
    $toPay = $mchPay->mchPay($params);
    if($toPay["return_code"]=="SUCCESS"&&$toPay["result_code"]=="SUCCESS"){
        /**
        * 业务逻辑 数据库操作
        */
        echo "付款成功"; 
    }else{
        //输出错误信息
        echo $toPay['err_code'].$toPay['err_code_des'];
    }
```