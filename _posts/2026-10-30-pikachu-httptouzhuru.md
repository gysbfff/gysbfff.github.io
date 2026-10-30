---
title: Pikachu SQL Inject - HTTP Header 注入
date: 2026-10-30
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, SQL注入, Header注入, Python自动化]
---

## 题目: Pikachu SQL Inject - HTTP Header 注入

## 题目描述
页面是一个登录框。登录成功后，后端会将客户端的 IP、User-Agent 等信息记录到数据库。要求利用 Header 注入漏洞，获取数据库敏感信息。

## 解题思路
由于本地 MySQL 8.0+ 环境限制，直接在 Burp 里使用 `updatexml` 容易失败。为了提高效率，我编写了 Python 脚本，自动携带 `PHPSESSID` 发送请求，并在 `User-Agent` 头部注入 `extractvalue` 报错 Payload，通过正则表达式直接从报错回显中提取敏感数据。

## 解题步骤
**1. 抓包与提取凭证**
* 成功登录进入页面，开启 F12 抓包。
* 在 `sqli_header.php` 的请求头中，提取出有效的 `PHPSESSID`（如 `khh51dubk1op29mcvfv8ptgrl8`）。

**2. 编写 Python 自动化注入脚本**
* 构造 `User-Agent` 注入 Payload（优先使用 `extractvalue`）：
  `' and extractvalue(1,concat(0x7e,(select group_concat(username,0x3a,pw) from member),0x7e)) and '1'='1`
* 使用 `requests` 库发送带 Cookie 的 GET 请求。
* 利用正则表达式 `r"~([^~]+)~"` 提取报错信息中的内容。

**3. 执行结果**
* 在终端运行 `python pikachu_header_sqli.py`。
* 脚本输出：`[+] 报错注入成功！正在提取数据...`，成功回显了数据库中的账号密码哈希，证明 Header 注入利用成功。

## 知识点总结
1. **HTTP Header 注入的隐蔽性**：注入点往往在 `User-Agent`、`X-Forwarded-For` 等请求头中，需要抓包才能发现。
2. **自动化脚本的优势**：面对严格的 WAF 或版本限制，使用 Python 脚本能灵活控制请求头、Cookie 和正则匹配，比手动抓包更高效。
3. **防御策略**：使用预编译语句（PDO）处理日志记录的 INSERT 语句；对所有 HTTP Header 进行严格过滤。
