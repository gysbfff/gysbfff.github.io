---
title: Bugku CTF Simple_SSTI_2
date: 2026-09-18
categories: CTF
tags: CTF,Web,SSTI,Jinja2,模板注入，Flask，命令执行
---

# 题目：Bugku CTF Simple_SSTI_2

## 题目描述
Bugku Simple_SSTI_2，Web SSTI模板注入题目。页面接收GET参数flag，后端使用Jinja2模板渲染传入的参数，存在服务器端模板注入漏洞。

## 解题思路
1. 先用最简payload验证SSTI漏洞；
2. 尝试`__subclasses__`相关payload，发现被过滤，直接500报错；
3. 改用Flask内置lipsum函数，通过`lipsum.__globals__`拿到全局命名空间，成功取出os模块；
4. 使用os.popen执行系统命令ls，查看当前目录文件，发现目录内存在flag文件；
5. 执行`cat flag`读取文件，拿到flag。

## 解题步骤
1. 漏洞验证：访问`?flag={{7*7}}`，页面返回49，确认Jinja2 SSTI漏洞存在。
2. 尝试利用`__subclasses__`读取文件，靶场对该字符串做了过滤，访问直接500，放弃该思路。
3. 使用payload `?flag={{lipsum.__globals__}}` 获取全局变量，成功拿到os模块。
4. 执行系统命令ls：`?flag={{lipsum.__globals__['os'].popen('ls').read()}}`，列出当前目录文件，发现flag文件。
5. 执行`cat flag`读取flag文件内容，得到最终flag：`flag{1c0f40017df2840159c1fd2d4a243644}`。

## 踩坑指南
1. `__subclasses__`被过滤，访问直接报500内部错误，不要死磕这一类payload。
2. 一开始尝试`cat /flag`读取根目录flag，没有回显，flag不在根目录，需要先ls查看当前目录文件。
3. 使用PowerShell发包，避免在线发包工具自动HTML转义，导致payload失效。
4. 命令执行无回显时，优先ls查看目录，判断flag文件位置，不要盲目猜文件路径。

## 知识点总结
1. SSTI（服务器端模板注入）：Jinja2模板引擎会解析`{{}}`中的Python表达式，在服务端执行。
2. Flask内置函数lipsum：可以通过`lipsum.__globals__`获取全局命名空间，绕过`__subclasses__`过滤。
3. os.popen()：Python中调用系统命令，读取命令输出结果。
4. 渗透思路：漏洞验证 → 探测过滤字符 → 寻找可用全局变量 → 列出目录 → 读取目标文件。
