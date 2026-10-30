---
title: Pikachu SQL Inject - 字符型注入 (get)
date: 2026-10-30
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, SQL注入, 字符型注入, UNION联合查询]
---

## 题目: Pikachu SQL Inject - 字符型注入 (get)

## 题目描述
页面提供一个输入框，要求输入用户名进行查询。要求利用字符型 SQL 注入漏洞，获取数据库中的敏感信息。

## 解题思路
字符型注入的 SQL 语句通常为 `SELECT * FROM users WHERE username = '$name'`。因为输入被单引号包裹，我们必须先输入单引号 `'` 闭合字符串，再拼接 `UNION SELECT` 语句，最后用注释符 `-- ` 或 `#` 注释掉尾部残留的单引号。

## 解题步骤

**1. 闭合测试与寻找回显**
* 输入 `kobe'`，页面报错 `You have an error in your SQL syntax; check the manual...`，证明单引号被成功闭合，存在字符型注入。
* 构造 Payload：`kobe' union select 1,2 -- `，页面成功回显了 `kobe` 的信息，并在下方打印出 `your uid:1` 和 `your email is: 2`。
* 这说明当前查询有 2 个字段，且数字 `1` 和 `2` 的位置是我们能控制的回显位。

**2. 爆数据库名与敏感数据**
* 将回显位替换为数据库函数，查询当前库名：
  `kobe' union select 1,database() -- `
  *页面成功回显 `pikachu`。*
* 直接查询 `member` 表中的账号密码：
  `kobe' union select 1,group_concat(username,0x3a,pw) from member -- `
  *(0x3a 是冒号的十六进制，用于分隔账号与密码)*
* 页面成功打印出所有用户的账号和 MD5 哈希值（如 `vince:e10adc...`），注入彻底完成。

## 踩坑指南
1. * **报错 `near ''` 的处理**：直接输入 `kobe'` 会触发语法错误，是因为原本 SQL 语句末尾还有一个单引号没有被处理。必须在 Payload 末尾加上 `-- `（注意后面要带空格）或 `#` 来注释掉尾部代码。
2. * **高版本 PHP 环境兼容性**：在本地测试时，如果直接在浏览器地址栏输入长 Payload，由于 URL 编码和 PHP 高版本的限制，偶尔会出现白屏。建议直接使用输入框提交，或使用 Burp Suite 抓包改包，能更稳定地拿到回显。

## 知识点总结
1. **字符型注入的核心**：闭合符（`'`） + 注入语句 + 注释符（`-- `）。
2. **UNION SELECT 技巧**：利用 `group_concat()` 可以将多行数据拼接成一行输出，配合 `concat()` 或十六进制（`0x3a`）分隔，能在单次请求中“一锅端”所有账号密码。
3. **防御策略**：必须使用**预编译语句（PDO/MySQLi）**；对输入进行严格过滤；数据库密码严禁明文或简单 MD5 存储，必须加盐（如 bcrypt）。
