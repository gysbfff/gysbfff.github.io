---
layout: post
title: Pikachu Cross-Site Scripting - XSS之过滤
date: 2026-10-26 
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, XSS, 过滤绕过]
---

## 题目: Pikachu Cross-Site Scripting - XSS之过滤

## 题目描述
页面提供了一个输入框，要求利用 XSS 漏洞让页面弹窗。本关的后端代码对用户输入进行了特定字符或标签的过滤，需要想办法绕过。

## 解题思路
XSS 过滤通常采用黑名单机制，比如使用 `preg_replace` 将 `<script>` 替换为空。
既然 `<script>` 被拦截，我们就必须**更换其他能触发 JavaScript 代码的 HTML 标签或事件**。比如 `<img>` 标签的 `onerror` 事件、`<svg>` 标签的 `onload` 事件。

## 解题步骤

**1. 尝试常规 Payload（⚠️ 踩坑记录）**
* 输入 `<script>alert(1)</script>` 并提交。
* **现象**：页面没有弹窗，说明后端使用了黑名单，识别到了 `<script>` 标签并直接将其过滤/删除了。
* **尝试大小写绕过**：输入 `<ScRipT>alert(1)</ScRipT>`，依然失败。说明后端可能使用了忽略大小写的正则匹配（如 `preg_replace('/<script>/i', '', $input)`）。

**2. 更换 HTML 标签绕过**
* 既然 `<script>` 被严打，我们可以使用图片加载失败的事件。
* 清空输入框，输入 Payload：`<img src=x onerror=alert(1)>`
* 点击 submit。

**3. 触发弹窗**
* 页面成功弹出了警告框 `1`，通关！
* **现象分析**：页面下方原本的文字变成了 `别说这些小破图的话，不要怕，就是干！`，中间夹杂着一个裂开的图片图标。这说明后端只过滤了 `<script>`，并没有过滤 `<img>` 或 `onerror` 事件。当浏览器解析到这张不存在的图片时，触发了 `onerror`，从而执行了 JS 代码。

## 知识点总结
1. **黑名单过滤的局限性**：黑名单永远是不完整的。开发者往往只想到了 `<script>`，却忽略了 `<img>`、`<svg>`、`<body>` 等大量可以执行 JS 的标签和事件属性。
2. **常见 XSS 绕过姿势**：
   * **标签替换**：`<img src=x onerror=alert(1)>`、`<svg onload=alert(1)>`。
   * **大小写混淆**：`<ScRiPt>`（针对单次且未忽略大小写的正则）。
   * **双写绕过**：如果过滤规则是将 `<script>` 替换为空，可以尝试输入 `<scr<script>ipt>alert(1)</scr</script>ipt>`。
   * **编码绕过**：使用 HTML 实体编码、URL 编码或 JS 编码（如 `&#x6a;avascript:`）。
3. **防御策略**：**不要使用黑名单来防 XSS**。应当采用白名单机制，或者更有效的**输出转义**（HTML 实体编码），将 `<`、`>`、`&`、`'`、`"` 等字符转义，让浏览器将其视为纯文本。
