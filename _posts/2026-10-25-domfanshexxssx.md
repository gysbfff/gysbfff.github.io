---
title: "Pikachu Cross-Site Scripting - DOM型XSS-x"
date: "2026-10-25
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, XSS, DOM型XSS]
---

## 题目: Pikachu Cross-Site Scripting - DOM型XSS-x

## 题目描述
页面提供了一个输入框（“请说出你的伤心往事”），要求利用 DOM 型 XSS 漏洞，让页面弹窗。本关的界面与上一关不同，且底层 JavaScript 逻辑有所变化。

## 解题思路
DOM 型 XSS 完全发生在客户端。本关的 JS 代码在处理 URL 参数时，可能使用了 `decodeURI` 进行解码，或者在拼接时对 `javascript:` 伪协议进行了特定处理。
虽然直接输入 `<script>alert(1)</script>` 会被当做普通字符串，但只要注入点最终是输出在 `<a>` 标签的 `href` 属性中，我们就可以尝试使用 JavaScript 伪协议来触发。

## 解题步骤

**1. 尝试常规标签注入**
* 在输入框直接输入 `<script>alert(1)</script>`，点击按钮。
* 页面虽然回显了文字，但没有弹窗。说明 `<script>` 标签在此上下文中失效。

**2. 构造伪协议 Payload（通关步骤）**
* 清空输入框，输入：`javascript:alert(1)`
* 点击按钮，此时页面下方生成了一段可点击的文本（或链接）。
* **点击该文本**，浏览器成功执行了 `javascript:` 伪协议，弹出了警告框，通关！

**3. 备选思路（如果伪协议失效）**
* 如果遇到前端对 `javascript:` 做了替换，可以尝试闭合标签的 Payload：
  `'><img src=x onerror=alert(1)>`
* 或使用 SVG 标签：`<svg onload=alert(1)>`

## 知识点总结
1. **上下文决定 Payload**：在 `innerHTML` 操作中，`<script>` 标签往往不会执行。此时需要根据输出位置（如 `href` 属性、`text` 文本），选择 `javascript:` 伪协议或带有 `onerror` 等事件属性的 HTML 标签。
2. **DOM型XSS的利用难度**：相比于反射型，DOM 型 XSS 的 Payload 构造更加灵活多变。开发者即使过滤了 `<script>`，如果没有过滤 `javascript:` 或 `onerror` 等事件属性，依然会导致漏洞。
3. **防御策略**：
   * 使用 `textContent` 代替 `innerHTML` 来输出用户输入的内容。
   * 对 URL 参数进行严格的过滤，禁止 `javascript:` 等危险协议。
