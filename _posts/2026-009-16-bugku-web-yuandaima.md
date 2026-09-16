---
title: Bugku CTF 源代码
date: 2026-09-16
categories: CTF
tags: Bugku, Web, JS审计, escape编码, CyberChef
---

# 题目: 源代码

## 题目描述
考点：JS前端源码审计
提示：看看源代码？
给到靶机链接，页面存在密码输入框，密码校验逻辑写在前端JS代码中。

## 解题思路
1. 使用快捷键 `Ctrl+U` 查看网页源代码，发现页面的JS代码使用`unescape()`进行编码执行，核心代码：`eval(unescape(p1) + unescape('%35%34%61%61%32' + p2));`
2. 一开始尝试浏览器控制台直接运行代码，出现DOM报错：`Cannot set properties of null`，是因为页面还没有加载完表单元素，eval执行失败。
3. 改用CyberChef静态解码方案，不需要执行JS，直接还原完整JS源码。
4. CyberChef操作：搜索`URL Decode`配方拖入Recipe，将编码字符串放入Input自动解码，得到明文JS代码。
5. 分析if判断语句，提取页面输入框需要提交的密码，填入靶机页面提交，服务器返回flag。

## 解题步骤
1. 打开CyberChef官网 https://cyberchef.org/
2. 在左侧Operations搜索框输入`URL Decode`，将配方拖动到中间Recipe区域
3. 将p1、`%35%34%61%61%32`、p2三段拼接后的百分号编码串放入Input
4. AutoBake自动解码，Output输出完整JS明文，读取密码校验逻辑
5. 将正确密码填入靶机页面输入框提交，页面回显flag:flag{54aa267d709b2b54aa2aa648cf6e87a7114f1}


## 知识点总结
1. `unescape()`是JS旧版编码函数，编码后生成`%xx`格式，没有unicode`%u`部分时，可以使用URL Decode静态解码。
2. 前端JS校验的Web题，优先Ctrl+U查看源代码。
3. eval执行报错不一定代表思路错误，可以不用运行JS，直接静态解码审计字符串。

## 踩坑记录
1. 一开始误以为JS拼接出来的完整字符串就是flag，自行拼接flag{}直接提交到Bugku平台，提交错误。
-正确流程：解码拿到**靶机页面的登录密码** → 在靶机页面提交密码 → 服务器返回flag → 将服务器返回flag复制提交到Bugku。
2. 混淆「表单密码」和「最终flag」，二者不是同一个字符串！
