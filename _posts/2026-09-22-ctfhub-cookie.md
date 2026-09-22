---
title: CTFHub HTTP请求协议 Cookie
date: 20226--09-22
categories: CTF Web HTTP协议
tags: CTFHub Web Cookie欺骗
---

# 题目： CTFHub HTTP请求协议 Cookie

## 题目描述
访问靶场，页面输出'hello guest.only admin can get falg.',提示只有admin管理员才能获取flag。
## 解题思路
服务器使用Cookie识别访问者身份。首次访问服务端下发`Set-Cookie: admin=0`，`admin=0`代表访客guest。我们修改请求头Cookie字段，将`admin`的值改为`1`，伪造管理员身份，从而获取flag。

## 解题步骤
1. 使用Burp Suite抓包，访问靶场页面，观察响应头，发现`Set-Cookie: admin=0`。
2. 将数据包右键发送到Repeater模块。
3. 在请求头末尾添加一行：`Cookie: admin=1`（Cookie后面必须带英文冒号，HTTP头格式严格）。
4. 点击Send发送修改后的请求，在响应包中得到flag。

## 知识点总结
Cookie欺骗：服务端直接信任客户端提交的Cookie内容，没有在后端做身份校验。客户端可以随意修改本地Cookie值，实现身份伪造。
> 安全启示：重要身份不能只依靠Cookie判断，后端需要保存真实身份信息，不能信任前端可控数据。

## 踩坑指南
1. HTTP请求头`Cookie`，发送后如没反应要再检查一遍请求头是否有误
2. 区分：`Set-Cookie`是**服务器返回给浏览器**；`Cookie`是**浏览器发给服务器**的请求头。 

## 题目描述
访问靶场页面，页面输出'hello guest.only admin can get flag.'，提示只有admin管理员才能获取flag。



