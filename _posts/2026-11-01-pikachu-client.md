---
title: Pikachu Unsafe File Upload - Client Check
date: 2026-11-01 
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, 文件上传, BurpSuite, 客户端校验]
---

## 题目: Pikachu Unsafe File Upload - client check

## 题目描述
页面提供了一个文件上传功能，并提示“这里只允许上传图片哦！”。要求绕过前端的安全校验机制，成功上传一个可执行的 PHP 脚本文件，并验证其被执行。

## 解题思路
题目提示“只允许上传图片”，说明前端使用了 JavaScript 对文件后缀名进行了白名单校验。前端的校验是极其脆弱的，攻击者可以轻易通过浏览器开发者工具修改、禁用 JS，或者使用 Burp Suite 抓包拦截请求，在发送给服务器之前将文件后缀修改为 `.php`，从而绕过限制。

## 解题步骤
**1. 准备 Payload**
在本地新建一个文本文件，写入最简单的验证代码，并保存为 `test.php`（确保文件后缀确实为 `.php`，而不是 `.php.txt`）。

要写入的代码为：
<?php echo "pikachu_upload_success"; ?>

**2. 绕过前端拦截与抓包改后缀（⚠️ 踩坑点）**
*   直接选中 `test.php` 点击上传，由于前端 JS 的白名单校验，浏览器会直接弹窗报错，导致 Burp Suite 根本抓不到任何请求数据。
*   **解决办法**：先在网页上选择一张正常的图片（如 `1.jpg`）点击“开始上传”，让请求顺利发出。
*   在 Burp Suite 拦截到的 POST 请求中，找到请求体里的 `filename="1.jpg"`，将其强制修改为 `filename="test.php"`，然后点击 **Forward** 放行。
*   此时页面会提示上传成功，并给出路径：`uploads/test.php`。

**3. 访问触发与环境残留排错（⚠️ 踩坑点）**
*   在浏览器中直接访问 `http://127.0.0.1/pikachu/vul/unsafeupload/uploads/test.php`。
*   **踩坑记录**：在反复测试过程中，曾尝试使用 `system('whoami')` 等 Payload，由于浏览器会自动对 URL 中的单引号进行 URL 编码（变成 `%27`），导致后端的 PHP 解析器直接报错 `Parse error`。此外，由于服务器端文件存在缓存或旧文件覆盖，即使物理删除了新文件，访问时依然持续报错 `line 179` 的脏数据错误。
*   **解决办法**：针对 URL 编码问题，对于简单的漏洞证明，只需保留最基础的 `echo` 语句即可；若需执行复杂命令，建议换用 Burp 的 Repeater 直接发送 POST 包。针对环境残留问题，直接进入靶场物理目录（`WWW/pikachu/vul/unsafeupload/uploads/`）清空所有测试的 `.php` 文件，并更换一个全新的文件名重新上传，彻底清理环境。

**4. 验证利用证明**
重新上传干净的脚本并访问，页面成功执行了 PHP 代码，输出了 `pikachu_upload_success`，证明文件上传漏洞利用成功，前端校验完全失效。

## 知识点总结
1. **前端校验不可信**：任何前端的 JS 校验（后缀名、MIME 类型）都可以通过抓包修改、禁用 JS 等方式绕过。安全校验必须在服务器端执行。
2. **防御策略**：
   * 后端强制进行后缀名白名单校验（如只允许 `.jpg`、`.png`）。
   * 使用 `finfo` 或 `mime_content_type` 校验真实的 MIME 类型。
   * 检查文件头（Magic Number）是否合法。
   * 将上传后的文件重命名为随机字符串，且禁止上传目录的脚本执行权限。
