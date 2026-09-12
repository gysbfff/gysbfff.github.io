---
title:"Bugku CTF web 基础$_POST Writeup
date:2026-09-11
categories:Bugku CTF
tags:
 -Bugku
 -CTF
 -Web
 -PHP
 -POST传参
 ---

 # 题目：web基础$_POST

 ## 题目描述
 Web，分值10分。
 页面给出PHP源码：
 ```php
$what=$_POST['what'];
echo $what;
if($what=='flag')
echo 'flag{c6526287b29ce38b5837cf7d79a5d497}';

## 解题思路
PHP  $_POST['what']  用于接收POST请求body中的what参数。
GET参数放在URL里，POST参数放在请求主体，无法直接在地址栏构造。
需要发送POST请求，提交  what=flag ，满足判断条件即可输出flag。

## 解题步骤
1. 访问靶机页面，查看源代码。
2. 打开cmd，使用curl发送POST请求：
3. 执行命令，命令行输出flag。

## 解题方法
1. 按下键盘  Win + R ，输入  cmd ，回车，打开命令提示符（黑窗口）
2. 复制下面这一行命令
curl -X POST -d "what=flag" http://160.202.254.160:11707
3. 在cmd窗口粘贴，按下回车发送请求即可出现flag

## 总结
1. $_POST  获取POST请求body提交的数据，参数不会出现在URL。
2. GET：参数附加在URL；POST：参数放在请求主体。
3. CTF Web基础，掌握两种不同HTTP传参方式。

##知识点
1.GET 和 POST 区分
- GET：参数写在网址URL上
- POST：参数藏在请求体Body里，地址栏看不见
2.PHP 是服务器端脚本语言，专门用来做网页后端开发。
-浏览器看到的网页文字是前端（HTML），PHP跑在网站服务器上，负责接收你的参数、做判断、返回内容，Bugku这些Web题基本都是PHP写的。

##容易踩坑的点
1. POST不是加密！抓包工具（Burp）照样能看见参数内容，只是不显示在网址。
2. 不要记混： $_GET  对应GET请求； $_POST 对应POST请求，不匹配拿不到数据。

