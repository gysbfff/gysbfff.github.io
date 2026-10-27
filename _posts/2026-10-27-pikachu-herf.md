---
title: Pikachu Cross-Site Scripting - XSS之href输出
date: 2026-10-27 
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, XSS, href输出, 伪协议]
---

## 题目: Pikachu Cross-Site Scripting - XSS之href输出

## 题目描述
页面要求用户输入一个常用的网站 URL 地址。要求利用 XSS 漏洞，通过点击生成的链接触发弹窗。

## 解题思路
因为输入的内容是 URL，后端大概率会将其放入 `<a href="用户输入">` 标签中。
如果后端只过滤了 `<script>` 标签，而**没有对 `javascript:` 伪协议进行严格限制**，我们只需要输入 `javascript:alert(1)`，当生成的链接被点击时，浏览器就会执行这段 JS 代码。

## 解题步骤

**1. 探测输入点**
* 输入一个正常的网址（如 `http://www.baidu.com`），点击 submit。
* 页面下方生成了一个可点击的超链接，说明输入被拼接到了 `href` 属性中。

**2. 尝试常规 Payload（⚠️ 踩坑记录）**
* 输入 `<script>alert(1)</script>`，提交后发现没有弹窗，页面可能只显示了残缺的文字。
* **原因分析**：`<script>` 标签被后端过滤或转义了。但在 `href` 属性中，执行代码不一定非要靠标签。

**3. 利用 javascript: 伪协议（通关步骤）**
* 清空输入框，输入：
  `javascript:alert(1)`
* 点击 submit，页面下方生成了一个链接（链接文字可能是 `javascript:alert(1)`）。
* **点击这个链接**，浏览器成功执行了 JS 代码，弹出了警告框，通关！

**4. 备选绕过思路**
* 如果后端对 `javascript:` 关键字也做了过滤（比如替换为空），可以尝试大小写混合：
  `JaVaScRiPt:alert(1)`
* 或者使用 HTML 实体编码：
  `&#106;avascript:alert(1)`

## 知识点总结
1. **`href` 属性注入**：在 `<a href="...">` 中，用户输入会被当作 URL 解析。如果允许 `javascript:` 伪协议，攻击者就能在用户点击链接时执行任意 JS 代码。
2. **过滤的局限**：很多开发者只防 `<script>` 标签，却忘了 HTML 属性和伪协议也是 XSS 的重灾区。
3. **防御策略**：
   * 严格校验 URL 协议，只允许 `http://`、`https://` 等安全协议，**直接拦截 `javascript:`、`data:`、`vbscript:`**。
   * 对输出到 HTML 属性的内容进行 HTML 实体编码。
   * 对于用户提交的 URL，可以在服务器端解析并规范化后再输出。
