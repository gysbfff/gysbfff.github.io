---
title: Pikachu File Inclusion - 本地文件包含 (Local)
date: 2026-10-30
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, 文件包含, LFI, 目录穿越]
---

## 题目: Pikachu File Inclusion - File Inclusion(local)

## 题目描述
页面提供一个下拉框选择 NBA 球员。要求利用本地文件包含漏洞，读取服务器上的任意文件。

## 解题思路
后端代码直接使用了 `include()` 函数，并将用户提交的文件名拼接到路径中。由于未对输入进行过滤，攻击者可以利用 `../`（目录穿越）跳出当前目录，读取系统文件或敏感配置。

## 解题步骤
**1. 发现注入点**
* 选择任意选项点击提交，观察 URL 发现参数为 `?filename=file1.php`。由于是 GET 请求，无需借助抓包工具，直接在地址栏修改即可。

**2. 目录穿越读取系统文件**
* 将 URL 参数修改为：`?filename=../../../../Windows/win.ini`。
* 按回车后，页面顶部成功回显了 `[fonts]`、`[extensions]` 等 Windows 系统配置文件内容，证明目录穿越成功。

**3. 读取靶场源码（进阶）**
* 构造路径：`?filename=../../../../phpStudy_64/phpstudy_pro/WWW/pikachu/vul/fileinclude/fi_local.php`。
* 成功读取到后端 PHP 源码，证明本地文件包含漏洞可利用，且危害极大（可读取任意敏感文件）。

## 知识点总结
1. **本地文件包含（LFI）**：通过控制 `include()` 函数的参数，读取并执行服务器本地的文件。
2. **目录穿越**：利用 `../` 返回上级目录，突破网站根目录的限制。
3. **防御策略**：
   * 使用白名单限制允许包含的文件名，避免用户直接控制路径。
   * 对用户输入中的 `../` 进行严格过滤或替换。
   * 关闭 `allow_url_include`，防止远程文件包含。
