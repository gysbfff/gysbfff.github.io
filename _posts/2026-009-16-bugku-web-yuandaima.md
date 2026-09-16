---
title: "Bugku CTF - 源代码"
date: 2026-09-16
categories: CTF
tags: [Bugku, Web, JS审计, escape编码, CyberChef]
---

# 源代码
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

## 工具操作步骤
1. 打开CyberChef官网 https://cyberchef.org/
2. 在左侧Operations搜索框输入`URL Decode`，将配方拖动到中间Recipe区域
3. 将p1、`%35%34%61%61%32`、p2三段拼接后的百分号编码串放入Input
4. AutoBake自动解码，Output输出完整JS明文，读取密码校验逻辑
5. 将正确密码填入靶机页面输入框提交，页面回显flag

## Payload（靶机输入框填写）
