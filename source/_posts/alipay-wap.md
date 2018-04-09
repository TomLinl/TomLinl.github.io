---
title: 支付宝手机网站支付(PHP)
date: 2018-04-07 22:29:02
tags: 
    - 支付宝支付
    - PHP
---
关于接入的步骤可以参考上一篇《支付宝APP支付服务端接入(PHP)》，详细可以看[官方文档](https://docs.open.alipay.com/203/105285/)
下面就不多说了，接入开发，首先还是下载官方的sdk,然后进行调用。
<!--more-->
实现方法
手机网站支付请求示例
```javascript
public function alipay(){
        require_once("alipay/wappay/service/AlipayTradeService.php");
        require_once("alipay/wappay/buildermodel/AlipayTradeWapPayContentBuilder.php");
        require_once("alipay/config.php");
        
        //商户订单号，商户网站订单系统中唯一订单号，必填
        $out_trade_no = '商户订单号';

        //订单名称，必填
        $subject = '订单名称';

        //付款金额，必填
        $total_amount = '付款金额';

        //商品描述，可空
        $body = '商品描述';

        //超时时间
        $timeout_express="1m";

        $payRequestBuilder = new \AlipayTradeWapPayContentBuilder();
        $payRequestBuilder->setBody($body);
        $payRequestBuilder->setSubject($subject);
        $payRequestBuilder->setOutTradeNo($out_trade_no);
        $payRequestBuilder->setTotalAmount($total_amount);
        $payRequestBuilder->setTimeExpress($timeout_express);

        $payResponse = new \AlipayTradeService($config);
        $payResponse->wapPay($payRequestBuilder,$config['return_url'],$config['notify_url']);

    }
```
调起支付宝app支付示例
```javascript
//由于微信浏览器屏蔽支付宝链接 官方给出跳出微信环境支付的解决方案 引入ap.js（将ap.js和pay.htm放在网站根目录）
<script type="text/javascript" src="ap.js"></script>
    <script>
        $(document).on('click','[data-role="alipay"]',function (e) {
            var order_id = $(this).attr('data-id');
            var ua = window.navigator.userAgent.toLowerCase();
            $.post("{:U('xxx/xxx/alipay')}",{order_id:order_id},function (res) {
                if(res.status==false){
                    $.toast(res.info);
                }else {
                    if(ua.match(/MicroMessenger/i) == 'micromessenger'){//判断微信浏览器
                        e.preventDefault();
                        e.stopPropagation();
                        e.stopImmediatePropagation();
                        _AP.pay(res);
                        return false;
                    }else{
                        window.location.href = res;
                    }
                }
            });
        })
    </script>
```
回调示例
```javascript
public function alipay_notify()
    {
        require_once("alipay/wappay/service/AlipayTradeService.php");
        require_once("alipay/config.php");
        $arr = $_POST;
        $alipaySevice = new \AlipayTradeService($config);
        $alipaySevice->writeLog(var_export($_POST, true));
        $result = $alipaySevice->check($arr);
        if ($result) {//验证成功
            //商户订单号

            $out_trade_no = $_POST['out_trade_no'];

            //支付宝交易号

            $trade_no = $_POST['trade_no'];

            //交易状态
            $trade_status = $_POST['trade_status'];


            if ($_POST['trade_status'] == 'TRADE_FINISHED') {

                //判断该笔订单是否在商户网站中已经做过处理
                //如果没有做过处理，根据订单号（out_trade_no）在商户网站的订单系统中查到该笔订单的详细，并执行商户的业务程序
                //请务必判断请求时的total_amount与通知时获取的total_fee为一致的
                //如果有做过处理，不执行商户的业务程序

                //注意：
                //退款日期超过可退款期限后（如三个月可退款），支付宝系统发送该交易状态通知
            } else if ($_POST['trade_status'] == 'TRADE_SUCCESS') {
                
                //支付成功
                /**
                * 业务逻辑
                */

                echo "success";        //请不要修改或删除

            } else {
                //验证失败
                echo "fail";    //请不要修改或删除

            }
        }
    }
```