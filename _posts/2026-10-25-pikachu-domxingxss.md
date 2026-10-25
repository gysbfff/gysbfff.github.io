---
title: Pikachu Cross-Site Scripting - DOM型XSS
date: 2026-10-25 18:00:00 +0800
categories: Web安全 靶场Writeup
tags: Pikachu XSS DOM型XSS
---

## 题目:  Pikachu Cross-Site Scripting - DOM型XSS

## 题目描述
页面提供了一个输入框，要求利用 DOM 型 XSS 漏洞，让页面弹窗。

## 解题思路
DOM 型 XSS 与反射型、存储型最大的区别在于：**它完全发生在客户端（浏览器）**。服务器不参与数据的处理，恶意代码的执行是由前端 JavaScript 代码在解析 URL 或操作 DOM 树时导致的。
我们需要查看页面源代码，寻找不安全的 DOM 操作函数（如 `innerHTML`、`document.write` 等），并根据上下文构造特定的 Payload。

## 解题步骤

**1. 尝试常规注入（⚠️ 踩坑记录）**
* 在输入框直接输入最经典的 XSS Payload：`<script>alert(1)</script>`，点击 `click me!`。
* 页面只回显了 `what do you see?`，没有任何弹窗。
* **原因分析**：通过 F12 审查前端源码发现，前端 JS 是通过 `innerHTML` 将用户输入直接拼接到了 `<a>` 标签的 `href` 属性中。由于 `<script>` 标签被包裹在属性的引号内，浏览器会将其当作 URL 字符串的一部分，而不会将其解析为可执行的脚本。

**2. 构造专属 Payload**
* 既然注入点在 `href` 属性中，最有效的方法是使用 JavaScript 伪协议。
* 清空输入框，输入：`javascript:alert(1)`
* 点击 `click me!`，此时页面下方原本的文字变成了一个超链接。

**3. 触发弹窗**
* 点击页面下方生成的 `what do you see?` 链接，浏览器成功执行 `javascript:` 伪协议，弹出了警告框，通关！

## 知识点总结
1. **DOM 型 XSS 的独特性**：它的漏洞逻辑完全在前端 JS 中，不涉及后端代码，因此传统的 WAF（Web 应用防火墙）和基于后端过滤的防御机制对它无效。
2. **`<script>` 标签的失效场景**：在 `innerHTML` 等操作中，动态插入的 `<script>` 标签通常不会执行。此时需要根据输出上下文，改用其他 Payload（如 `javascript:` 伪协议、`<img src=x onerror=alert(1)>` 等）。
3. **防御策略**：
   * 避免使用 `innerHTML`、`outerHTML`、`document.write` 等直接插入 HTML 的 API，改用 `textContent` 或 `innerText`。
   * 如果必须使用 `innerHTML`，必须对用户输入进行严格的过滤或转义处理。
