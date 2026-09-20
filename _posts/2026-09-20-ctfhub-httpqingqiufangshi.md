---
title: CTFHub HTTP协议 请求方式
date: 2026-09-20
categories: Web
tags: CTFHub Web HTTP请求方式
---

# 题目：CTFHub HTTP协议 请求方式

## 题目描述
HTTP 请求方法, HTTP/1.1协议中共定义了八种方法（也叫动作）来以不同方式操作指定的资源。

## 解题思路
本题考察HTTP请求方式，除GET、POST外，还支持PUT/DELETE/HEAD等，服务器判断请求方法返回flag。

## 解题步骤
1. 打开靶场链接，观察页面提示
2. 使用Burp Suite抓包，修改请求方法
3. 把GET改成题目要求的请求方式，发送数据包
4. 页面返回flag

## 踩坑指南
-注意：修改请求方法大小写敏感，PUT不能小写put（部分服务器严格校验）
-抓包后不要多余修改其他请求头，容易400
-靶场超时要重新开启环境

## 知识点总结
1. HTTP常见请求方法
   GET: 从服务器获取资源
   POST: 提交数据到服务器
   PUT: 上传或修改服务器上资源
   DELETE: 删除服务器资源
   HEAD: 只获取响应头，不返回响应主体
   OPTIONS: 查询服务器支持哪些请求方法
3. 服务器可以根据请求方法做访问控制
4. Burp修改请求包是Web基础操作
5. Web题里payload = 你传给服务器的攻击数据
- SQL注入：payload是注入语句  ?id=1' or 1=1--+ 
- XSS：payload是  <script>alert(1)</script> 
- 本题：payload是修改请求方式的HTTP包


    
