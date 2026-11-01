---
layout: post
title: Pikachu SSRF - SSRF(file_get_contents)
date: 2026-11-01 
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, SSRF, 文件读取]
---

## 题目: Pikachu SSRF - SSRF(file_get_contents)

## 题目描述
页面提供了读取内容的功能。要求利用 SSRF 漏洞，让服务器通过 `file_get_contents()` 函数读取本地系统文件。

## 解题思路
后端代码直接接收了 `file` 参数作为 URL，并调用 `file_get_contents()` 请求该地址。由于没有做任何过滤，我们可以利用 `file://` 伪协议，让服务器直接读取它本机的任意文件。

## 解题步骤
**1. 观察参数与回显**
* 页面 URL 参数为 `?file=`，输入内网地址可以看到页面回显内容。

**2. 利用 file:// 协议读取系统文件**
* 构造 Payload，使用 `file://` 伪协议直接读取 Windows 系统文件：
  `?file=file:///C:/Windows/win.ini`

**3. 验证漏洞**
* 提交后，页面顶部成功回显了 `; for 16-bit app support [fonts]...` 等 Windows 系统配置内容。
* 证明服务器成功读取了本地文件，SSRF 漏洞（file_get_contents 版本）利用成功。

## 知识点总结
1. **SSRF 的利用**：利用 `file://` 协议可以实现任意文件读取。
2. **防御策略**：严格限制 `file_get_contents()` 的参数，禁止传入用户可控的 URL。
