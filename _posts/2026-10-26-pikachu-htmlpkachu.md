---
title: Pikachu Cross-Site Scripting - XSS之htmlspecialchars
date: 2026-10-26
categories: Web安全 靶场Writeup
tags: Pikachu XSS htmlspecialchars绕过 属性注入]
---

## 题目: Pikachu Cross-Site Scripting - XSS之htmlspecialchars

## 题目描述
页面提供了一个输入框，要求利用 XSS 漏洞让页面弹窗。本关的后端代码使用了 PHP 的 `htmlspecialchars()` 函数对输入进行了处理，需要分析其配置缺陷并绕过。

## 解题思路
`htmlspecialchars()` 是 PHP 中常用的防 XSS 函数。但它有一个经典的坑：如果使用默认参数 `htmlspecialchars($string)`，它**默认不会转义单引号 `'`**（只转义双引号 `"`、小于号 `<` 和大于号 `>`）。
如果后端将用户输入拼接在 HTML 的**单引号属性**中（如 `href='输入'`），我们就可以利用单引号闭合属性，然后插入事件（如 `onmouseover`）来触发 XSS。

## 解题步骤

**1. 尝试常规 Payload（⚠️ 踩坑记录）**
* 输入 `<script>alert(1)</script>` 并提交。
* **现象**：页面回显“你的输入已经被记录：”，但代码以纯文本形式显示（`<script>alert(1)</script>`），没有弹窗。
* **原因分析**：`htmlspecialchars()` 将 `<` 和 `>` 转义成了 `&lt;` 和 `&gt;`，浏览器将其视为普通文本。但注意，页面显示的是代码本身，说明单引号 `'` 没有被转义。

**2. 利用单引号闭合属性**
* 既然单引号未转义，我们输入：`' onmouseover='alert(1)`
* 提交后，页面下方生成了一段文字。将鼠标悬停在其上方，成功弹出了警告框 `1`。
* **原理**：后端代码大概率为 `<a href='输入'>`。单引号闭合了 `href` 的属性值，并插入了 `onmouseover` 事件。由于单引号未被转义，浏览器成功解析了这个事件。

## 知识点总结
1. **`htmlspecialchars()` 的参数陷阱**：
   * 默认参数（`ENT_COMPAT`）只转义双引号，**不转义单引号**。
   * 正确的防御姿势应指定 `ENT_QUOTES` 参数：`htmlspecialchars($str, ENT_QUOTES, 'UTF-8')`，同时转义单引号和双引号。
2. **属性闭合注入**：当输出点位于 HTML 属性内部时，即使 `<` 和 `>` 被转义，只要对应的引号未被转义，攻击者就能闭合属性并注入事件（如 `onmouseover`、`onclick`）。
3. **防御策略**：
   * 使用 `htmlspecialchars($str, ENT_QUOTES, 'UTF-8')`。
   * 遵循**上下文输出编码**原则：输出到 HTML 属性、JS 代码、URL 参数时，应使用对应上下文的安全编码。
