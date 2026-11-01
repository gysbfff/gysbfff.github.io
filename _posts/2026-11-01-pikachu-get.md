---
title: Pikachu Unsafe File Upload - getimagesize (图片马与文件包含联合利用)
date: 2026-11-01
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, 文件上传, 图片马, 文件包含]
---

## 题目: Pikachu Unsafe File Upload - getimagesize

## 题目描述
页面提示“这里只允许上传图片，不要乱搞！”。后端使用了 `getimagesize()` 函数读取文件头，要求上传真实的图片文件，并会对上传后的文件进行时间戳重命名。

## 解题思路
`getimagesize()` 会检查文件的二进制魔数，普通的伪造后缀或 MIME 无法通过。我们可以将 PHP 代码追加到正常图片的末尾，制作**“图片马”**。但直接访问 `.jpg` 不会执行代码，必须配合 **File Inclusion（文件包含）** 漏洞，让 `include()` 去解析这张图片。

## 解题步骤
**1. 制作图片马**
*   准备一张正常图片 `1.jpg` 和一个 PHP 木马 `shell.php`（内容为 `<?php echo "pikachu_upload_success"; ?>`）。
*   在 CMD 中使用二进制合并命令：
    ```cmd
    copy /b 1.jpg + shell.php shell.jpg
    ```
*   生成一个真实图片，但尾部附带了 PHP 代码的 `shell.jpg`。

**2. 上传图片马**
*   在网页选择 `shell.jpg` 正常上传。
*   页面提示上传成功，路径为 `uploads/2026/11/01/6145...273.jpg`。

**3. 结合文件包含漏洞触发执行（⚠️ 踩坑记录）**
*   进入 **File Inclusion (local)** 关卡。
*   **踩坑记录**：尝试直接在地址栏物理访问 `.jpg` 路径时，由于目录层级不对，返回了 `404 Not Found`。此外，尝试在 `fi_local.php` 界面直接输入路径，由于没有正确利用 `../` 目录穿越，未能包含成功。
*   **解决办法**：在 `fi_local.php` 的 URL 中构造 Payload，利用 `../../` 退回到根目录，再进入 `unsafeupload/uploads/` 找到图片马：
    ```text
    http://127.0.0.1/pikachu/vul/fileinclude/fi_local.php?filename=../../unsafeupload/uploads/2026/11/01/6145...273.jpg&submit=提交
    ```
*   虽然页面最后报了 PHP 语法错误（`Parse error`），但这**恰恰证明了漏洞利用成功**——服务器已经把图片里的内容当成 PHP 代码去解析了。

## 知识点总结
1. **图片马原理**：将恶意代码追加到合法文件末尾，保留合法文件头，从而绕过 `getimagesize()` 文件头检测。
2. **文件包含的配合**：单独的图片马无法执行，必须配合 `include()` 漏洞，才能让 PHP 解析器去执行图片尾部的代码。
3. **防御策略**：
   * 上传图片必须进行**二次渲染**（彻底销毁附加代码）。
   * 禁止在上传目录执行 PHP 脚本。
   * 修复文件包含漏洞，限制 `include` 的路径白名单。
