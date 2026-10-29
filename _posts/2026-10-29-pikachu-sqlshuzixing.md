---
title: Pikachu SQL Inject - 数字型注入
date: 2026-10-29
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, SQL注入, 数字型注入, Python自动化]
---

## 题目: Pikachu SQL Inject - 数字型注入

## 题目描述
页面提供了一个下拉框用于选择 `userid` 查询用户信息。提示明确说明“变量类型为数字”。要求利用数字型 SQL 注入漏洞，获取数据库敏感数据。

## 解题思路
因为是 POST 请求，且下拉框限制了输入，在浏览器 F12 里修改参数经常会遇到各种兼容性问题（尤其是本地高版本 PHP 环境）。因此，我决定直接使用 **Python 的 `requests` 库** 编写自动化脚本，通过代码构造 POST 请求，这样不仅能精准控制 Payload，还能方便地根据响应内容做判断。

## 解题步骤

### 第一步：抓包分析请求
* 在页面下拉框选择 `1`，点击查询，F12 抓包。
* 发现请求为 POST，目标 URL 为 `sqli_id.php`，请求体包含 `id=1&submit=查询`。

### 第二步：验证注入是否存在（布尔测试）
* 编写 Python 脚本，分别发送 `id=1 and 1=1` 和 `id=1 and 1=2`。
* **现象**：`1=1` 时页面正常回显 `vince`，而 `1=2` 时查询无结果。这证明数字型注入点确实存在。

### 第三步：探测字段数（ORDER BY）
* 注入的下一步是确定原表的列数。我写了一个简单的循环脚本，从 `ORDER BY 1` 测到 `ORDER BY 10`。
* **结果**：`ORDER BY 2` 页面正常，`ORDER BY 3` 页面报错。由此推断后端 SQL 语句查询了 **2 个字段**。

### 第四步：UNION 联合查询爆数据
由于确定只有 2 列，我们让前面的查询失效（`id=0`），直接利用 `UNION SELECT 1, 查询语句` 把数据带到页面上。

1.  **查库名**：
    *   Payload：`0 union select 1,database()`
    *   结果：获取到当前数据库名为 `pikachu`。
2.  **查表名**：
    *   Payload：`0 union select 1,group_concat(table_name) from information_schema.tables where table_schema='pikachu'`
    *   结果：爆出了 `httpinfo, member, message, users, xssblind`。一眼锁定目标表为 `member`。
3.  **查列名**：
    *   Payload：`0 union select 1,group_concat(column_name) from information_schema.columns where table_name='member' and table_schema='pikachu'`
    *   结果：找到了 `username` 和 `pw` 两个关键字段。
4.  **爆账号密码**：
    *   Payload：`0 union select 1,group_concat(concat(username, ':', pw)) from member`
    *   结果：成功获取到账号和对应的 MD5 哈希值，例如 `vince:e10adc3949ba59abbe56e057f20f883e`。

### 第五步：哈希解密
* 拿到 MD5 值后，直接丢进 cmd5 等在线网站查询。
* 解密出明文密码为 `123456`。说明靶场未对密码加盐，且使用了弱口令。

## 踩坑
*   **⚠️ 坑1：浏览器 F12 修改 POST 参数的局限性**。与 GET 请求不同，POST 参数在 URL 中不可见。虽然 F12 的“网络”面板有“编辑并重发”功能，但在某些版本或特定环境下，修改参数后请求容易被拦截或报错。**解决方案**：直接用 Python 的 `requests.post()` 发送脚本，稳定且高效。
*   **⚠️ 坑2：本地环境兼容性导致的页面崩溃**。在浏览器手动输入 `union select` 测试时，页面经常直接白屏或报错。这是因为靶场代码基于老版本 PHP 编写，本地高版本 PHP 环境在执行某些复杂的联合查询或获取大量数据时，底层数据库函数会直接崩溃。**解决思路**：改用自动化脚本抓取返回内容，或者将长数据拆分，避免直接触发环境 Bug。

## 知识点总结
1.  **布尔盲注基础**：利用 `and 1=1` 和 `and 1=2` 的页面差异，是验证注入点最快的方式。
2.  **ORDER BY 测列数**：联合查询的前提是字段数必须一致，`ORDER BY n` 是测试列数的标准动作。
3.  **MySQL 元数据表**：`information_schema.tables` 和 `information_schema.columns` 是 SQL 注入中“顺藤摸瓜”的地图。
4.  **防御策略**：使用预编译语句（PDO），绝不拼接 SQL；密码存储必须加盐（如 bcrypt），杜绝 MD5 裸奔。
