---
title: layui-Ajax分页
date: 2018-04-10 00:20:28
tags: 
    - layui
    - Ajax
---
layPage 致力于提供极致的分页逻辑，既可轻松胜任异步分页，也可作为页面刷新式分页。下面是使用layui分页控件实现异步请求分页操作的实现方法。
<!--more-->
```javascript
//引入组件js和css
<script src="__STATIC__/layui/layui.js"></script>
<link rel="stylesheet" href="__STATIC__/layui/css/layui.css">
$(function () {
        var url = '请求地址'
        var data = {};
        data.page = 1;
        ajax(data);
        function ajax(data) {
            $.ajax({
                type: "post",
                url: url,
                data: data,
                success: function (result) {
                    var res = result.data;
                    var count = result.count;
                    var page = result.page;
                    var limit = result.limit;
                    console.log(res); //查看服务器返回的数据
                    if(count>limit){
                        lay_page(count,limit,page);
                    }
                }
            })
        }
        function lay_page(count,limit,page) {
            layui.use(['laypage', 'layer'], function() {
                var laypage = layui.laypage
                        , layer = layui.layer;
                laypage.render({
                    elem: '指向存放分页的容器，值可以是容器ID、DOM对象'
                    , count: count //数据总数，从服务端得到
                    , limit: limit //每页显示的条数
                    , curr: page
                    , jump: function (obj, first) {
                        //obj包含了当前分页的所有参数，比如：
                        //console.log(obj.curr); //得到当前页，以便向服务端请求对应页的数据。
                        //console.log(obj.limit); //得到每页显示的条数
                        //首次不执行
                        if (!first) {
                            data.page=obj.curr;
                            ajax(data);
                        }
                    }
                });
            });
        }
    })
```