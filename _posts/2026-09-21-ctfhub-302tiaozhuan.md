---
title: 302跳转
date: 2026-09-21
categories: CTFHub
tags: CTFHub Web HTTP协议 302跳转
---

# 题目: 302跳转

## 题目描述
页面提示'No Flag here!'，点击'Give me falg'后页面自动跳转，无法直接看到flag。考察HTTP 302临时重定向原理。

## 解题思路
302是临时重定向状态码。浏览器收到302响应，会自动读取'Location'响应头并跳转至新页面，会丢弃本次响应的正文内容，而flag就写在302这一次的响应正文里面。
使用Burp Suite抓包，将请求发送到Repeater,关闭自动跟随跳转，查看原始响应拿到falg。

## 解题步骤
1. 使用Burp Suite自带浏览器访问靶机页面，点击'Give me falg'。
2. Proxy的HTTP history捕获到`index.php`的GET请求，状态码302。
3. 将数据包右键`Send to Repeater`。
4. Repeater设置`Follow redirects`为`Never`，禁止自动跳转。
5. 点击Send发送请求，查看右侧Response原始报文，在响应底部获取flag。

## 踩坑指南
1. 不要用普通浏览器直接访问，浏览器自动跟随跳转，看不到原始响应的flag。
2. Repeater必须关闭自动跟随跳转，否则会直接跳转到index.html，丢失flag。
3. 区分：302临时重定向；301是永久重定向。

## 知识点总结
- `302 Moved Temporarily`：HTTP临时重定向。
- `Location`响应头：指定跳转的目标URL。
- 浏览器会自动跟随3xx跳转；Burp Repeater可以手动控制是否跟随跳转，查看原始HTTP响应。
