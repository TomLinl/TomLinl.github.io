---
title: PHP实现文件上传与下载
date: 2018-05-01 22:18:17
tags: PHP
---
文件上传原理：将客户端的文件上传到服务器端，再将服务器端的临时文件移动到指定目录即可。
<!--more-->

#### 客户端配置：
表单页面
表单发送方式为 post
添加 enctype="multipart/form-data"

通过使用 PHP 的全局数组 $_FILES，你可以从客户计算机向远程服务器上传文件。

第一个参数是表单的 input name，第二个下标可以是 "name", "type", "size", "tmp_name" 或 "error"。就像这样：

$_FILES["file"]["name"] - 被上传文件的名称
$_FILES["file"]["type"] - 被上传文件的类型
$_FILES["file"]["size"] - 被上传文件的大小，以字节计
$_FILES["file"]["tmp_name"] - 存储在服务器的文件的临时副本的名称
$_FILES["file"]["error"] - 由文件上传导致的错误代码

move_uploaded_file() 函数将上传的文件移动到新位置。

若成功，则返回 true，否则返回 false。

move_uploaded_file(file,newloc)

将文件从 source 拷贝到 destination。如果成功则返回 TRUE，否则返回 FALSE。
copy(source,destination)

#### 文件上传配置    
file_uploads=on,支持HTTP上传
upload_tmp_dir=,临时文件保存的目录
upload_max_filesize=2M,允许上传文件的最大值
max_file_uploads=20,允许文件上传的最大文件数
post_max_size=8M,POST方式发送数据的最大值
max_execution_time=-1,设置了脚本被解析器终止之前允许的最大执行时间，单位为秒，防止程序写的不好而占尽服务器资源
max_input_time=60,脚本解析输入数据允许的最大时间，单位为秒
max_input_nesting_level=64,设置输入变量的嵌套深度
max_input_vars=1000,接受多少输入的变量（限制分别应用于$_GET、$_POST和$_COOKIE超全局变量）指令的使用减轻了以哈希碰撞来进行拒绝服务器攻击的可能性。如果有超过指令指定数量的变量，将会导致$_WARNING的产生，更多的输入变量将会从请求中截断。
memory_limit=128M,最大单线程的独立内存使用量。也就是一个web请求，给予线程最大使用量的定义。

#### 错误信息说明
1、UPLOAD_ERR_OK

其值为 0，没有错误发生，文件上传成功。
 

2、UPLOAD_ERR_INI_SIZE

其值为 1，上传的文件超过了 php.ini 中 upload_max_filesize选项限制的值。
 

3、UPLOAD_ERR_FORM_SIZE

其值为 2，上传文件的大小超过了 HTML 表单中 MAX_FILE_SIZE 选项指定的值。
 

4、UPLOAD_ERR_PARTIAL

其值为 3，文件只有部分被上传。
 

5、UPLOAD_ERR_NO_FILE

其值为 4，没有文件被上传。
 

6、UPLOAD_ERR_NO_TMP_DIR

其值为 6，找不到临时文件夹。PHP 4.3.10 和 PHP 5.0.3 引进。
 

7、UPLOAD_ERR_CANT_WRITE

其值为 7，文件写入失败。PHP 5.1.0 引进。


8、UPLOAD_ERR_EXTENSION

其值为 8，上传的文件被PHP扩展程序中断



#### 上传文件限制

通过表单隐藏域限制文件上传的最大值
```html
<input type="hidden" name='MAX_FILE_SIZE' value='字节数' />
```

通过accept属性限制上传文件类型
```html
<input type="file" name="file" accept="文件的MIME类型" />
```
（ps:表单限制不可靠）
服务器限制
限制文件上传的大小、类型、检测类型、检测是否为HTTP POST方式上传



#### 文件下载


```php
$filename=$_GET['filename'];
header('Content-Disposition:attachment;filename='.$filename);
header("Content-length:".filesize($filename));
readfile($filename);
```