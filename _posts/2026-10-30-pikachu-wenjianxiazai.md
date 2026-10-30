---
title: Pikachu Unsafe File Download - 不安全的文件下载
date: 2026-10-30
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, 文件下载, 目录穿越, Python自动化]
---

## 题目: Pikachu Unsafe File Download - 不安全的文件下载

## 题目描述
页面展示了一组 NBA 球员头像，点击即可下载。要求利用不安全的文件下载漏洞，下载服务器上的敏感文件。

## 解题思路
后端在接收下载参数时，未对用户提交的文件名进行过滤，直接拼接进文件路径。攻击者可以利用 `../` 进行目录穿越。
**⚠️ 环境踩坑**：本地 phpStudy 配置了 `open_basedir`，限制了 PHP 只能访问 Web 根目录（如 `WWW` 文件夹）。因此，常规的跨盘读取（如读取 `C:\Windows\win.ini`）会被系统安全策略拦截。

## 解题步骤
**1. 功能测试与请求捕获**
* 点击球员图片，抓取请求 URL：
  `http://127.0.0.1/pikachu/vul/unsafedownload/execdownload.php?filename=kb.png`
* 确认参数为 `filename`。

**2. 编写 Python 脚本批量测试**
* 既然跨盘被拦截，我们将目标转向 Web 根目录下的源码文件。手动在 Burp 中试错极慢，因此编写脚本自动遍历不同层级的 `../`。
* 脚本通过判断响应包中是否包含“你要下载的文件不存在”来确认攻击是否成功。

**3. 攻击验证与分析**
* 脚本成功运行，Payload `../../../index.php` 成功回显了 `index.php` 的源码片段（响应长度从 553 字节突变为 3955 字节）。
* **原理**：从当前脚本目录（`/pikachu/vul/unsafedownload/`）向上跳三级，刚好回到 Web 根目录（`WWW/`），从而成功读取到 `index.php`。

## 知识点总结
1. **任意文件下载的危害**：攻击者可以下载网站的配置文件、系统文件甚至源码，为进一步渗透提供情报。
2. **`open_basedir` 限制**：PHP 配置项，可将文件访问限制在指定目录内，是防止跨盘读取系统文件的有效手段。
3. **防御策略**：
   * 使用白名单限制允许下载的文件名或文件后缀。
   * 对用户输入中的 `../` 进行严格过滤。
   * 配置 `open_basedir` 限制 PHP 的访问范围。
