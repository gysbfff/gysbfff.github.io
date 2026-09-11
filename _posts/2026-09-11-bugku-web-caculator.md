---
title: Bugku CTF 计算器
date: 2026-09-11
categories: [CTF, Web]
tags: [Bugku,前端绕过,maxlength]
---

# Bugku CTF 计算器
## 题目信息
- 平台：Bugku CTF
- 题目名称：计算器
- 难度：Web入门
- 考点：前端DOM限制绕过，maxlength属性理解

## 题目描述
计算正确即可得到flag。

## 解题思路
页面存在输入框用来填写算式答案，但输入框设置了前端限制 `maxlength="1"`，只能输入单个字符。而算式结果是两位数，无法直接输入。
`maxlength` 只是浏览器前端层面的限制，后端校验不受这个属性控制。我们利用开发者工具修改网页DOM，增大输入字符上限，填入正确计算结果，拿到flag。

## 解题步骤
1. 点击「启动场景」进入题目页面，页面给出算式：`27+5=?`。
2. 按下F12打开浏览器开发者工具，使用元素选择器点击输入框，定位输入框对应的HTML代码。
3. 修改属性：双击 `maxlength="1"`，将数字`1`改为`10`，回车保存修改。
修改后代码示例：
```html
<input type="text" class="input" maxlength="10">
