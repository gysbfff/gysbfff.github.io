---
title: Pikachu Cross-Site Scripting-反射型XSS(post)
date: 2026-10-25 
categories: Web 靶场Writeup
tags: Pikachu XSS POST型XSS Burp Suite
---

## 题目: Pikachu Cross-Site Scripting-反射型XSS(post)

## 题目描述
这是一个需要提交账号密码的登录表单。提交方式为 POST 请求。要求利用反射型 XSS 漏洞，让页面弹出警告框。

## 解题思路
POST 型反射 XSS 的原理与 GET 型相同，都是由于服务端未对用户输入进行过滤直接回显导致的。
区别在于，POST 请求的参数不在 URL 中，因此我们无法通过直接修改地址栏来注入 Payload。必须使用 Burp Suite 拦截请求，在请求体（Request Body）中修改参数。

## 解题步骤

**1. 寻找真正的注入点（⚠️ 踩坑记录）**
* 在登录框输入任意账号密码（如 `admin / 123`）并抓包，将 `username` 修改为 `<script>alert(1)</script>`。发现页面回显 `username or password error!`，**没有弹窗**。
* **原因分析**：这是一个带有鉴权的登录逻辑。如果账号密码错误，后端直接返回错误提示，根本不会走到“回显用户名”的代码分支。也就是说，登录框本身可能不是注入点，或者需要正确凭证才能触发回显。
* **转折点**：使用正确的凭证（如 `admin / 123456`）登录后，页面跳转到了一个带有输入框的新页面，提示“你已经登陆成功”。这个输入框（询问你喜欢哪个 NBA 球员）才是真正接收用户输入并回显的地方。

**2. 抓包与注入**
* 开启 Burp Suite 代理拦截。
* 在该输入框中随便输入内容，点击 submit，拦截到 POST 请求。
* 将请求体中的参数值替换为 Payload：`message=<script>alert(1)</script>&submit=submit`。
* 放行请求（Forward）。

**3. 成功触发**
* 浏览器成功弹出了一个写着 `1` 的警告框，反射型 XSS (POST) 注入成功！

## 知识点总结
1. **POST 型 XSS 的利用条件**：与 GET 型 XSS 不同，POST 参数在请求体中，受害者无法通过简单的点击链接触发。攻击者通常需要构造一个恶意的 HTML 页面（利用表单自动提交）来诱导受害者访问。
2. **逻辑上下文的重要性**：在渗透测试中，寻找注入点不能只看输入框，要结合业务逻辑。登录失败的页面不处理输入，所以不会触发 XSS。必须要走到成功回显的页面才能注入。
3. **防御策略**：
   * 对用户输入的数据在输出到 HTML 页面之前，必须进行严格的 HTML 实体编码（如 `htmlspecialchars()`）。
   * 避免将用户输入直接拼接进 JavaScript 代码或 HTML 标签属性中。
