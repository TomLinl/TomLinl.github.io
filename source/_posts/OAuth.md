---
title: PHP第三方登录—OAuth2.0协议
date: 2018-05-01 22:32:36
tags: 
    - PHP
    - OAuth2.0
---
OAuth 是一个开放标准，允许用户让第三方应用访问该用户在某一网站上存储的私密的资源（如照片，视频，联系人列表），而无需将用户名和密码提供给第三方应用。目前，OAuth 的最新版本为 2.0
<!--more-->

OAuth 允许用户提供一个令牌，而不是用户名和密码来访问他们存放在特定服务提供者的数据。每一个令牌授权一个特定的网站（例如，视频编辑网站)在特定的时段（例如，接下来的2小时内）内访问特定的资源（例如仅仅是某一相册中的视频）。这样，OAuth 允许用户授权第三方网站访问他们存储在另外的服务提供者上的信息，而不需要分享他们的访问许可或他们数据的所有内容。


OAuth授权流程
1.请求OAuth登陆页
Request Token URL - 未授权的令牌请求服务地址
2.用户使用第三方账号登陆并授权
3.返回登陆结果
User Authorization URL - 用户授权的令牌请求服务地址
AccessToken - 用户通过第三方应用访问OAuth接口令牌

新浪微博登陆 (http://open.weibo.com)
1.申请AppID和AppKey
2.下载PHP版本SDK
3.接入开发

QQ登陆 (http://connect.qq.com)
1.申请AppID和AppKey
2.下载PHP版本SDK
3.SDK配置
4.接入开发

以微博登陆为例：
config.php
```
define('WB_KEY','获取的AppKey');
define('WB_SRC','获取的AppSecret');
define('CALLBACK','回调地址(callback.php)');
```
wblogin.php
```
<?php
require_once 'config.php';
require_once 'saetv2.ex.class.php';
$o = new SaeTOAuthV2('WB_KEY','WB_SRC');
$url = CALLBACK；
$oauth = $o->getAuthorizeUrl($url);
header('Location: '.$oauth);
```
callback.php
```
<?php
require_once 'config.php';
require_once 'saetv2.ex.class.php';
$code = $_GET('code');
$keys['code'] = $code;
$keys['redirect_uri'] = CALLBACK;
$o = new SaeTOAuthV2('WB_KEY','WB_SRC');
$auth = $o->getAccessToken($keys);
setcookie('accesstoken',$auth['access_token'],time()+86400);
header('location: index.php');
```
index.php
```
<?php
     equire_once 'config.php';
     require_once 'saetv2.ex.class.php';
    $isLogin=isset($_COOKIE['accesstoken']);
?>
<!DOCTYPE html>
<html>
     <head>
          <meta charset="UTF-8">
          <title>微博测试</title>
     </head>
     <body>
          <?php if(!$isLogin){ ?>
               <a href='wblogin.php'><img src='./weibo_login.png'></a>
          <?php }else{ ?>
              您已成功登陆微博账号；
          <?php
             $o = new SaeTClientV2('WB_KEY','WB_SRC',$_COOKIE['accesstoken']);     //调用接口发布微博
             $o->update('发布微博');
          } ?>
     </body>
</html>
```
debug调试方法
```
function debug($val,$dump = false,$exit = true) {
    //自动获取函数名称$func
    if($dump){
          $func = 'var_dump';
     } else {
          $func = (is_array($val) || is_object($val) )? 'print_r' : 'printf';
     }
    header("Content-type: text/html; charset = utf-8");
    echo '<pre>debug output:<hr />';
    $func($val);
    echo '</pre>';
    if($exit); exit;
}
```