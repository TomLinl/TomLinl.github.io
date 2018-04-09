---
title: Vue+tp5简单列表操作
date: 2018-03-06 23:58:36
tags: 
    - Vue
    - PHP
---
前段时间，初次使用前端框架Vue和ThinkPHP5.1，于是就简单结合起来，用Vue+axios发送请求，TP5处理接口数据。
这里Vue、axios和TP5等等的安装就不多说了，按照官方文档来就ok了。
<!--more-->
Vue部分，在main.js中引入axios,qs来帮助处理数据，接口请求的时候会遇到跨域问题
```cpp
import axios from 'axios'
import Qs from 'qs'
Vue.prototype.$axios = axios;
Vue.config.productionTip = false

var axios_instance = axios.create({
transformRequest: [function (data) {
data = Qs.stringify(data);
return data;
}],
//Post请求传递参数时，需要在请求头加上item.ContentType = "application/x-www-form-urlencoded";
headers:{'Content-Type':'application/x-www-form-urlencoded'}
})
Vue.prototype.$axios = axios_instance;
```
PHP部分后端响应头进行设置
```cpp
// 指定允许其他域名访问
header("Access-Control-Allow-Origin:*");
// 响应类型
header("Access-Control-Allow-Methods:GET, POST, PATCH, PUT, DELETE");
// 响应头设置
header("Access-Control-Allow-Headers:x-requested-with,content-type");
```
解决了跨域问题后，接下来就能通过接口请求和数据处理了。

在Vue的/config/dev.env.js 设置 API_HOST
```cpp
'use strict'
const merge = require('webpack-merge')
const prodEnv = require('./prod.env')

module.exports = merge(prodEnv, {
NODE_ENV: '"development"',
API_HOST:'"http://127.0.0.1/tp5/"'
})
```

下面是一个简单的列表，包含简单的增删改查，查看单条数据等功能
{% asset_img list.png %}
列表部分代码
```cpp
<template>
    <div class="list-table">
        <h1>TODO-LIST</h1>
        <form>
            姓名：<input type="text" v-model="nickname"/> {{nickname}}
            <br/>
            年龄：<input type="text" v-model="age"/>  {{age}}
            <br/>
            <span style="color:red" v-if="!isAge">年龄必须是数字</span>
            <br/>
            <br/>
            <button type="button" @click="addList">添加</button>
            <button type="button" @click="init()">重置</button>
        </form>
        <br/>
        <table border="1" cellspacing="0">
            <thead>
                <th>序号</th>
                <th>姓名</th>
                <th>年龄</th>
                <th>操作</th>
            </thead>
            <tr v-for="(data,index) in  list">
                <td>{{index+1}}</td>
                <td>{{data.nickname}}</td>
                <td>{{data.age}}</td>
                <td>
                    <button @click="detail(index)">查看详情</button>
                    <button @click="delList(index)">删除</button>
                </td>
            </tr>
            <tr>
                <td colspan="2" style="text-align: center">总年龄：</td>
                <td>{{sumAge}}</td>
                <td><button @click="controlList">清空/还原</button></td>
            </tr>
        </table>
    </div>
</template>

<script>
export default {
    name:"TODOList"
    data () {
        return {
            //姓名
            nickname:"",
            //年龄
            age:"",
            //age是数字
            isAge:true,
            list: [],
        }
    },
    created(){
        this.getList()
    },
    methods:{
    /**
    * 获取列表数据
    */
    getList: function () {
        let $this = this;
        this.$axios.get(process.env.API_HOST)
        .then(function(response){
        $this.list = response.data;
        console.log(response.data);
        })
        .catch(function (error) {
        console.log(error);
        });
    },
    /**
    * 新增列表数据
    */
    addList:function(){
        let $this = this;
        if(!this.isAge){
            return false;
        }
        if($this.nickname && $this.age){
            $this.$axios.get(process.env.API_HOST + 'index/index/addList',{
                params: {
                    nickname: $this.nickname,
                    age: $this.age
                }
            }).then(function (response) {
                console.log(response.data);
                $this.list.push({
                    nickname:response.data.nickname,
                    age:response.data.age
                });
            }).catch(function (error) {
                console.log(error);
            });
        }
    },
    /**
    * 删除列表数据
    * @param index  list键值
    */
    delList:function(index){
        let $this = this;
        if(confirm('确定要删除吗')){
            if($this.list[index] != undefined){
                $this.$axios.post(process.env.API_HOST + 'index/index/delList',{
                    nickname: $this.list[index].nickname
                    }).then(function (response) {
                        if(response.data=='操作成功'){
                            $this.list.splice(index,1);
                        }
                }).catch(function (error) {
                    console.log(error);
                });
            }
        }
    },
    /**
    * 清空或还原列表数据
    */
    controlList:function(){
        if(confirm('确定要执行此操作吗')){
            this.$axios.post(process.env.API_HOST + 'index/index/controlList').then(response=>{
                if(response.data==''){
                    this.list.splice(0,this.list.length);
                }else{
                    for (var i in response.data) {
                        this.list.push({
                        nickname:response.data[i].nickname,
                        age:response.data[i].age
                    });
                    }
                }
            }).catch(function (error) {
                console.log(error);
            });;
        }
    },
    detail:function(index){
        this.$router.push({path:'/detail',query: {nickname: this.list[index].nickname}})
    },
    /**
    * 初始化输入
    */
    init:function(){
        this.nickname = "";
        this.age = "";
        this.isAge = true;
    },
    /**
    * 检测数字
    * @param theObj       要检测的字符串
    * @returns {boolean}  是数字放回true  不是返回false
    */
    checkNumber:function(theObj) {
        var reg = /^[0-9]+$/;
        if(reg.test(theObj)){
            return true;
        };
            return false;
        }
    },
    computed:{
        sumAge:function(){
            var ages = 0;
            for (var i in this.list) {
                ages += Number(this.list[i].age);
            }
            return ages;
        }
    },
    watch:{
        age:{
            /**
            * 监听age的变化
            * @param newValue
            * @param oldValue
            * @returns {boolean}
            */
            handler(newValue, oldValue){
                if(newValue == ""){
                    this.isAge = true;
                    return true;
                }
                var res = this.checkNumber(newValue);
                if(res){
                    this.isAge = true;
                }else{
                    this.isAge = false;
                }
            }
        }
    }
}
</script>

<style>
    .list-table{
        width: 100%;
    }
    .list-table table{
        margin: 0 auto;
        width: 500px;
    }
    .list-table table th{
        font-size: 28px;
    }
    .list-table table td{
        padding: 5px 0;
        font-size: 20px;
    }
</style>
```
{% asset_img detail.png %}
详情部分代码
```cpp
<template>
<div class="list-table">
    <h1>{{msg}}</h1>
    <table border="1" cellspacing="0">
    <thead>
        <th>用户UID</th>
        <th>姓名</th>
        <th>年龄</th>
    </thead>
    <tr>
        <td>{{uid}}</td>
        <td>{{nickname}}</td>
        <td>{{age}}</td>
    </tr>
    <tr>
        <td colspan="2">信息介绍</td>
        <td>
            <button type="button" @click="OpenModal">添加/编辑信息</button>
        </td>
    </tr>
    <tr>
        <td colspan="3" v-if="info">{{info}}</td>
    </tr>
    </table>
    <router-link to="/list">返回</router-link>
        <div class="open-box" v-if="OpenShow">
            <div class="open-box-content">
                <h3>编辑信息</h3>
                <p>请输入您的个人信息</p>
                <textarea v-model="info">{{info}}</textarea>
                <div class="open-box-btn">
                    <div class="open-box-btn-left">
                        <button @click="addinfo">保存</button>
                    </div>
                    <div class="open-box-btn-right">
                        <button @click="ColseModal">关闭</button>
                    </div>
                </div>
            </div>
         </div>
</div>
</template>

<script>
export default {
    name: 'Detail',
    data () {
        return {
        msg: '信息详情',
        uid:'',
        //姓名
        nickname:"",
        //年龄
        age:"",
        OpenShow: false,
        info:''
        }
    },
    created(){
        this.getinfo()
    },
    methods:{
    /**
    * 获取信息
    */    
    getinfo: function () {
        let $this = this;
        this.$axios.get(process.env.API_HOST+'index/index/getInfo',{
        params:{nickname:this.$route.query.nickname}
    })
    .then(function(response){
        $this.nickname = response.data.nickname
        $this.age = response.data.age
        $this.uid = response.data.uid
        $this.info = response.data.info
        console.log(response.data);
    })
    .catch(function (error) {
        console.log(error);
    });
    },
     /**
     * 编辑信息
     */  
    addinfo: function(){
        this.$axios.post(process.env.API_HOST+'index/index/addInfo',{
        info:this.info,
        uid:this.uid
    })
    .then(response=>{
        alert(response.data)
        this.OpenShow = !this.OpenShow
    })
    .catch(function(error){
        console.log(error);
    });

    },
    OpenModal() {
        this.OpenShow = !this.OpenShow
    },
    ColseModal() {
        this.OpenShow = !this.OpenShow
    }
    }
}
</script>

<style>
.list-table{
width: 100%;
}
.list-table table{
margin: 0 auto;
...
/*样式部分省略*/

/*弹窗样式*/
.open-box {
position: fixed;
background: rgba(0, 0, 0, 0.4);
width: 100%;
height: 100%;
left: 0;
top: 0;
z-index: 100;
color: #fff;
...
/*样式部分省略*/
</style>
```

TP5中相应的代码做数据库处理
```cpp
    /**
    * 获取列表数据
    */
    public function index()
    {
        $res = model('Member')->getUserList();
        return $res;
    }

    /**
    * 新增列表数据
    */
    public function addList(){
        $nickname=Input('nickname');
        $age=Input('age',0,'floatval');
        $res = model('Member')->save([
            'nickname'=>$nickname,
            'age'=>$age,
            'status'=>1
        ]);
        if($res){
            $data=['nickname'=>$nickname,'age'=>$age];
            $data = json_encode($data);
            return $data;
        }
    }

    /**
    * 删除列表数据
    */
    public function delList(){
        $nickname=Input('nickname');
        model('Member')->save(['status'=>-1],['nickname'=>$nickname]);
        return '操作成功';
    }

    /**
    * 清空还原列表数据
    */
    public function controlList(){
        if(!model('Member')->where(['status'=>1])->count()){
            model('Member')->save(['status'=>1],['status'=>-1]);
            $res = model('Member')->getUserList();
            return $res;
        }else{
            model('Member')->save(['status'=>-1],['status'=>1]);
            return '';
        }
    }

    /**
    * 获取信息
    */
    public function getInfo(){
        $nickname=Input('nickname');
        $info = model('Member')->where(['nickname'=>$nickname])->find();
        return $info;
    }

    /**
    * 编辑信息
    */
    public function addInfo(){
        $info = Input('info');
        $uid = Input('uid');
        $res = model('Member')->save(['info'=>$info],['uid'=>$uid]);
        if($res){
            return '编辑成功';
        }
    }

```
数据表结构
```cpp
--
-- 表的结构 `list_member`
--

CREATE TABLE IF NOT EXISTS `list_member` (
  `uid` int(11) NOT NULL AUTO_INCREMENT,
  `nickname` varchar(10) NOT NULL,
  `age` int(11) NOT NULL,
  `status` tinyint(4) NOT NULL,
  `info` varchar(200) NOT NULL,
  PRIMARY KEY (`uid`)
) ENGINE=MyISAM  DEFAULT CHARSET=utf8 AUTO_INCREMENT=1 ;
```