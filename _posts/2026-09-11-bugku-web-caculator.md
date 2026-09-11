---
title: "Bugku CTF 计算器 Writeup"
date: 2026-09-11
categories: [CTF, Web]
tags: [Bugku, 前端绕过, maxlength, Web入门]
description: Bugku入门Web题计算器，考点为前端maxlength输入长度限制绕过
---

# Bugku CTF｜计算器 Writeup
## 一、题目信息
- 题目平台：Bugku CTF
- 题目类型：Web 基础
- 题目难度：入门
- 考点：前端限制绕过、maxlength 属性修改

## 二、题目描述
页面提供一个计算器，需要计算出算式的正确结果并提交，即可获得 Flag。

## 三、解题思路
进入题目页面后可以发现：
输入答案的输入框**只能输入一个数字**，无法输入多位数字。

查看网页源码可知：
输入框设置了前端限制：`maxlength="1"`

`maxlength` **只属于前端浏览器限制**，仅限制用户输入，**后端不会校验该参数**。
因此我们可以直接修改前端代码，解除长度限制，输入正确答案拿到 Flag。

## 四、详细解题步骤
1. 启动题目环境，页面出现随机数学算式。
2. 按下 `F12` 打开开发者工具，选择元素选择器点击输入框。
3. 找到输入框源码：
```html
<input type="text" class="input" maxlength="1">
