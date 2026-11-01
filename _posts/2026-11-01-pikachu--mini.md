---
title: Pikachu Unsafe File Upload - Server Check (服务端 MIME 绕过)
date: 2026-11-01 
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, 文件上传, MIME绕过, BurpSuite]
---

## 题目: Pikachu Unsafe File Upload - server check

## 题目描述
页面提示“这里只允许上传图片，不要乱搞！”。本关引入了服务端的安全校验，要求绕过服务端的检测机制，成功上传并执行 PHP 脚本。

## 解题思路
当校验逻辑转移到后端时，前端的抓包改后缀（如双写、大小写）可能会被后端的黑名单拦截。通过分析发现，后端代码只校验了 HTTP 请求头中的 `Content-Type`（MIME 类型）。我们只需要在抓包时，将原本的 `application/octet-stream` 伪造为 `image/jpeg`，即可欺骗后端，让服务器认为我们上传的是一张合法图片。

## 解题步骤
**1. 准备纯净的 Payload**
在本地新建一个文本文件，写入以下验证代码，并保存为 `shell.php`（确保文件后缀为 `.php`）：
```php
<?php echo "pikachu_upload_success"; ?>
```

**2. 抓包与修改 MIME 类型**
开启 Burp Suite 拦截（Intercept is on）。
在网页上选择 shell.php 文件，点击“开始上传”。
在 Burp 拦截到的 POST 请求中，找到文件上传的分块部分。
核心操作：将请求体中的 Content-Type: application/octet-stream 手动修改为：
```text
Content-Type: image/jpeg
```
确保 filename="shell.php" 保持不变，点击 Forward 放行请求。

**3. 访问触发与验证**
页面提示：文件上传成功，文件保存的路径为：uploads/shell.php。
在浏览器中直接访问该路径：
```http://127.0.0.1/pikachu/vul/unsafeupload/uploads/shell.php
```
页面成功执行了 PHP 代码，输出了 pikachu_upload_success，证明 MIME 类型校验被成功绕过，文件上传漏洞利用成功。

## 知识点总结
*MIME 类型校验的脆弱性：服务端如果仅信任客户端传来的 Content-Type 头，是极易被伪造的。攻击者可以通过抓包工具随意修改该字段。

### 正确的防御策略：
*服务端必须使用 finfo 等函数读取文件的真实文件头（Magic Number），而不是轻信请求头。
*结合后缀名白名单校验。
*将上传的文件重命名为随机字符串，并配置上传目录禁止执行 PHP 脚本。
