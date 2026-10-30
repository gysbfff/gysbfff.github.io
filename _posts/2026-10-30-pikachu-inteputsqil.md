---
title: Pikachu SQL Inject - insert/update 注入 
date: 2026-10-30
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, SQL注入, UPDATE注入, 源码审计]
---

## 题目: Pikachu SQL Inject - "insert/update"注入

## 题目描述
页面提供登录与修改个人信息功能。要求利用 SQL 注入漏洞，在修改信息的 UPDATE 语句中注入恶意代码，篡改数据库中的敏感信息。

## 解题思路
当后端执行 `UPDATE` 语句时，如果未对输入进行过滤，攻击者可以通过闭合单引号并注入注释符（`#`），强行截断原本的语句结构，插入自定义的修改逻辑。本关由于本地高版本 MySQL 封杀了报错注入，最终采取**审计源码** + **直接篡改**的思路突破。

## 解题步骤

**1. 登录与初步测试**
* 使用 `admin` 账号成功登录，进入“个人会员中心”。
* 点击“修改个人信息”，在输入框随意填入单引号 `'` 并提交，页面提示“修改失败，请重试”。

**2. 源码审计与破局（⚠️ 核心踩坑记录）**
* **环境修复**：之前因 PHP 高版本导致 `MYSQL_ASSOC` 报错，已通过修改 `sqli_edit.php` 源码解决。
* **报错注入失败**：尝试使用 `updatexml` 和 `extractvalue` 函数，页面始终提示 `Only constant XPATH queries are supported`，证明本地 MySQL 8.0+ 环境已限制该攻击手法。
* **源码分析**：查看 `sqli_edit.php` 源码，发现核心 SQL 语句为：
  `update member set sex='$sex', phonenum='$phonenum', address='$add', email='$email' where username='$_SESSION['sqli']['username']'`
  发现表名为 `member`，且同时更新多个字段，一旦随意闭合，会导致后续的逗号和字段语法断裂，从而引发报错。

**3. 构造最终 Payload**
* 利用多字段更新的特性，在“邮箱”输入框构造 Payload：
  `' , sex='hacker' where username='admin' #`
* **原理**：闭合 email 的单引号后，用 `,` 拼接自己的 `sex='hacker'` 赋值语句，最后用 `#` 注释掉后面所有多余的代码。
* 提交后，刷新个人中心，发现性别已被成功修改为 `hacker`，UPDATE 注入成功。

## 知识点总结
1. **多字段 UPDATE 注入的闭合**：必须注意原 SQL 语句中逗号和字段的排列，利用 `#` 注释掉多余部分是成功的关键。
2. **高版本 MySQL 的限制**：MySQL 8.0+ 封杀了报错注入的常规玩法，实战中需灵活切换为时间盲注或直接篡改。
3. **源码审计的价值**：当黑盒测试受阻时，直接查看源码能快速定位过滤逻辑和拼接结构，从而实现精准打击。
4. **防御策略**：使用预编译语句（PDO）处理 UPDATE 语句，杜绝字符串拼接。
