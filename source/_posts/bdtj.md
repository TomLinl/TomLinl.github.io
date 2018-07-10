---
title: 百度统计API
date: 2018-06-22 20:24:36
tags: 百度统计
---
百度统计开放平台是百度统计全新推出的，业务数据收集（获取网站的PV UV 等数据）、导出以及应用为一体的开放体系，由JS API和tongji API两部分组成。
<!--more-->

一、JS API通过在页面上部署js代码的方式，收集网站的各类业务数据。

在部署百度统计开放平台JS API之前，先注册一个百度统计账号，并安装了百度统计的访问分析代码。

二、Tongji API支持将百度统计中的数据以通用格式导出，帮助您将报表中的数据导出至自身系统中，以便更加灵活地展现、分析、挖掘百度统计中的数据价值。

登录百度统计，在后台的管理—其它设置——数据导出服务，开通服务。

为了我们自己的网站后台看到数据，我们在自己的网站集成Tongji API

下载Tongji API Demo

Config.inc.php 配置用户名密码TOKEN等信息
```php
<?php
.
.
.
//USERNAME
define('USERNAME', 'YOUR USERNAME');

//PASSWORD
define('PASSWORD', 'YOUR PASSWORD');

//TOKEN
define('TOKEN', 'YOUR TOKEN');
.
.
.
```
demo.php
```php
//头部添加：
header(‘Content-type:text/json’);
'
'
'
//尾部添加：
$res = json_decode($ret['raw'], true);
print_r($res);
```
```php
//打印结果如下：
Array
(
    [header] => Array
        (
            [desc] => success
            [failures] => Array
                (
                )

            [oprs] => 1
            [succ] => 1
            [oprtime] => 0
            [quota] => 1
            [rquota] => 49974
            [status] => 0
        )

    [body] => Array
        (
            [data] => Array
                (
                    [0] => Array
                        (
                            [result] => Array
                                (
                                    [total] => 1
                                    [items] => Array
                                        (
                                            [0] => Array
                                                (
                                                    [0] => Array
                                                        (
                                                            [0] => 2018/06/22
                                                        )

                                                )

                                            [1] => Array
                                                (
                                                    [0] => Array
                                                        (
                                                            [0] => 11201
                                                            [1] => 745
                                                            [2] => 46.15
                                                            [3] => 426
                                                            [4] => 12.34
                                                        )

                                                )

                                            [2] => Array
                                                (
                                                )

                                            [3] => Array
                                                (
                                                )

                                        )

                                    [timeSpan] => Array
                                        (
                                            [0] => 2018/06/22
                                        )

                                    [sum] => Array
                                        (
                                            [0] => Array
                                                (
                                                    [0] => 11201
                                                    [1] => 745
                                                    [2] => 46.15
                                                    [3] => 426
                                                    [4] => 12.34
                                                )

                                            [1] => Array
                                                (
                                                )

                                        )

                                    [offset] => 0
                                    [pageSum] => Array
                                        (
                                            [0] => Array
                                                (
                                                    [0] => 11201
                                                    [1] => 745
                                                    [2] => 46.15
                                                    [3] => 426
                                                    [4] => 12.34
                                                )

                                            [1] => Array
                                                (
                                                )

                                            [2] => Array
                                                (
                                                )

                                        )

                                    [fields] => Array
                                        (
                                            [0] => simple_date_title
                                            [1] => pv_count
                                            [2] => visitor_count
                                            [3] => bounce_ratio
                                            [4] => avg_visit_time
                                            [5] => avg_visit_pages
                                        )

                                )

                        )

                )

        )

)
```
下面新建GetData.php写一个方法获取数据并处理
```php
<?php

function getBdData($start_date, $end_date, $gran = '', $metrics = 'pv_count,visitor_count',$clientDevice = '', $method = 'trend/time/a', $max_results = '0')
{
    require_once('Config.inc.php');
    require_once('LoginService.inc.php');
    require_once('ReportService.inc.php');
    header('Content-type:text/json');
    $loginService = new LoginService(LOGIN_URL, UUID);

// preLogin
    if (!$loginService->preLogin(USERNAME, TOKEN)) {
        exit();
    }

// doLogin
    $ret = $loginService->doLogin(USERNAME, PASSWORD, TOKEN);
    if ($ret) {
        $ucid = $ret['ucid'];
        $st = $ret['st'];
    } else {
        exit();
    }

    $reportService = new ReportService(API_URL, USERNAME, TOKEN, $ucid, $st);
    
    $siteId = 'your siteId';
    
    $ret = $reportService->getData(array(
        'site_id' => $siteId,                   //站点ID
        'method' => $method,             //通常对应要查询的报告
        'start_date' => $start_date,             //所查询数据的起始日期
        'end_date' => $end_date,               //所查询数据的结束日期
        'metrics' => $metrics,  //自定义指标选择，多个指标用逗号分隔
        'max_results' => $max_results,                     //返回所有条数
        'gran' => $gran,                        //时间粒度(只支持有该参数的报告): day/hour/week/month
    ));
   //将获取的数据进行处理
    $res = json_decode($ret['raw'], true);
    $res = $res["body"]["data"]['0']["result"];
    array_shift($res["fields"]);
    $fields = $res["fields"];
    foreach ($res["items"][1] as $key => &$val) {
        $val = array_combine($fields, $val);
        $time = $res["items"][0][$key][0];
        $val['time'] = $time;
    }
    unset($val);

// doLogout
    $loginService->doLogout(USERNAME, TOKEN, $ucid, $st);
    return $res["items"][1];
}
```
```php
//结果如下:[items][1][0]就是处理后想得到的形式
Array
(
    [total] => 1
    [items] => Array
        (
           .
           .
           .
            [1] => Array
                (
                    [0] => Array
                        (
                            [pv_count] => 11201
                            [visitor_count] => 745
                            [bounce_ratio] => 46.15
                            [avg_visit_time] => 426
                            [avg_visit_pages] => 12.34
                            [time] => 2018/06/22
                        )

                )
            .
            .
            .
)            
```
最后在控制器中调用getBdData方法获取数据并存入数据库
```php
 public function getBdTongJi(){
        require_once(APP_PATH.'Admin/Lib/bdtongji/GetData.php');
        $start_date = I('get.start','','strval');
        $end_date = I('get.end','','strval');
        if(!$start_date||!$end_date){
            $time = date('Ymd',time())-1;
            $start_date = $time;
            $end_date = $time;
        }
        $bdCount = M('bd_count');
        $res = getBdData($start_date,$end_date,'day','pv_count,bounce_ratio,visitor_count,avg_visit_time,avg_visit_pages');
        foreach ($res as &$data){
            $data['count_time']=strtotime($data['time']);
            if(!$bdCount->where(array('count_time'=>$data['count_time']))->count()){
                $bdCount->add($data);
            }
        }
        unset($data);
    }
```