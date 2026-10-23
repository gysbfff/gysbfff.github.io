---
title: CTFHub SSRF-伪协议读取文件
date: 2026-10-23
categories: CTFHub Web
tags: CTFHub Web
---

# 题目： CTFHub SSRF-伪协议读取文件

## 题目描述
打开题目页面，提示：
> 尝试去读取一下 Web 目录下的 flag.php 吧
URL 参数结构为：
```http://challenge-xxx.sandbox.ctfhub.com:10800/?url=
```
**分析：**
因为题目明确提示读取 Web 目录下的文件，说明服务器存在 SSRF 漏洞。但服务器环境限制了部分协议（如 `gopher`、`dict`），此时最直接有效的方式是使用 `file://` 协议。

## 解题思路

###  漏洞原理
SSRF 漏洞允许服务器发起请求，攻击者可以通过构造 `url` 参数，让服务器去读取本地文件。
在 PHP 的 cURL 环境中，`file://` 协议是默认支持的，但必须使用**绝对路径**。

###  攻击链
1. 构造 `file:///var/www/html/flag.php`。
2. 服务器读取本地文件并返回。
3. 检查源码（可能需要通过 F12 查看响应）获取 flag。

## 解题步骤
1. 构造 Payload
在浏览器地址栏输入：
```http://challenge-xxx.sandbox.ctfhub.com:10800/?url=file:///var/www/html/flag.php
```

*(注意：`file://` 后面必须是三个斜杠 `///`，代表根目录 `/`)*

2. 获取 Flag
如果页面直接回显了 `ctfhub{...}`，直接提交即可。
如果页面显示乱码或空白，按 `F12` 打开开发者工具 -> **网络 (Network)** -> 查看请求的 **响应 (Response)**，在原始代码中寻找 `ctfhub{...}`。
提交 flag，通关！

## 踩坑指南
1. 使用 `http://127.0.0.1/flag.php` 是空白
**原因：** 页面返回了 PHP 代码，被浏览器解析成了乱码。
**解决：** 使用 `file://` 协议直接读取文件源码。

2. 路径写错
**原因：** cURL 的 `file://` 协议不支持相对路径。
**解决：** 必须写绝对路径，例如 `/var/www/html/flag.php`。

## 知识点总结

| 知识点 | 说明 |
| :--- | :--- |
| SSRF | 服务端请求伪造，服务器替攻击者发起请求 |
| `file://` 协议 | 用于读取本地文件，需要绝对路径 |
| cURL | PHP 中用于请求的库，支持部分伪协议 |

### 防御建议
1. **限制协议**：只允许 `http` 和 `https` 协议。
2.  **校验路径**：禁止包含 `../` 等路径穿越字符。
