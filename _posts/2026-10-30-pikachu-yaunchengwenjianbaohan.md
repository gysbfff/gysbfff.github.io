---
title: Pikachu File Inclusion - 远程文件包含 (Remote)
date: 2026-10-30
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, 文件包含, RFI, data伪协议]
---

## 题目: Pikachu File Inclusion - File Inclusion(remote)

## 题目描述
页面提供一个下拉框。要求利用远程文件包含漏洞，读取远程服务器上的恶意脚本或利用伪协议执行代码。

## 解题思路
远程文件包含依赖于 PHP 的 `allow_url_include` 配置。在本地开启该配置后，如果直接包含外部 URL 较慢或受限，可以利用 `data://` 伪协议，将恶意 PHP 代码直接内联在 URL 中执行。

## 解题步骤
**1. 开启环境配置（⚠️ 踩坑记录）**
* 页面提示 `warning: 你的allow_url_include没有打开`。
* 在 phpStudy 中打开 `php.ini`，将 `allow_url_include = Off` 修改为 `On`，并重启 Apache 服务，警告消失。

**2. 利用 data:// 伪协议执行代码**
* 直接在地址栏构造 Payload：
  `?filename=data://text/plain,<?php phpinfo();?>`
* 按回车后，页面成功弹出了紫色的 PHP 配置信息页面（phpinfo），证明远程文件包含漏洞存在，且可以实现任意代码执行。

**3. 进阶利用（一句话木马）**
* 如果将 Payload 替换为：
  `?filename=data://text/plain,<?php @eval($_POST['cmd']);?>`
* 结合蚁剑等工具连接，即可完全控制服务器。

## 知识点总结
1. **远程文件包含（RFI）**：通过包含远程服务器上的脚本文件，攻击者可以实现代码执行。
2. **data:// 伪协议的优势**：无需在公网搭建服务器，直接将代码写在 URL 中，绕过外部网络限制。
3. **防御策略**：
   * 严格关闭 `allow_url_include`。
   * 使用白名单限制允许包含的路径。
   * 对输入中的 `http://`、`data://`、`php://` 等危险协议头进行严格过滤。
