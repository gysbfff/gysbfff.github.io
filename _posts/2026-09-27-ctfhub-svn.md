---
title: CTFHub SVN泄露
date: 2026-09-27
categories: CTFHub Web
tags: CTFHub Web 信息泄露 SVN泄露
---

# 题目: CTFHub SVN泄露

## 题目描述
点开靶场地址后页面显示Flag在服务器的旧版本中。开发人员使用SVN版本控制系统，部署网站时没有删除.svn版本控制文件夹，造成源码泄露，flag存放在SVN旧版本文件夹中。

## 解题思路
 SVN泄露，文件已经被web端删除，无法直接访问。
 通过下载`.svn/wc.db` SQLite数据库，从pristine表拿到文件sha1哈希，按照SVN的pristine存储规则构造URL，直接下载原始备份文件获取flag。
 
## 解题步骤
1. 访问`http://靶场/.svn/wc.db`下载数据库文件wc.db。
2. 使用DB Browser for SQLite打开wc.db。
3. 查看`nodes`表，发现flag文件名，checksum字段为NULL，代表文件已被删除，直接访问404。
4. 切换到`pristine`表，找到size匹配flag长度(33字节)的记录，提取sha1值，删除前缀`$sha1$`。
5. 根据SVN规则构造pristine访问链接：
 `/.svn/pristine/哈希前两位/完整哈希.svn-base`
6. 浏览器访问构造完成的URL，下载`.svn‑base`文件，记事本打开得到flag。

## 踩坑指南
1. nodes表checksum为NULL → 文件已经web端删除，直接访问txt文件404。
2. pristine表里面的 $sha1$ 只是算法标记，拼接URL必须删除这个前缀。
3. 哈希取前两位作为中间一层文件夹，SVN的pristine目录就是按哈希前两位分文件夹存储备份。
4. 后缀固定加上  .svn‑base 。

##  知识点总结
1. SVN旧版本文件访问格式
   /.svn/pristine/哈希前两位/完整哈希.svn-base
2. SVN版本控制部署不当，对外泄露 .svn 目录，造成源码、已删除文件泄露。
3. wc.db 保存工作副本全部文件元数据；pristine目录存放各个版本原始文件。
4. 防护：上线部署删除 .svn 、 .git 这类版本控制文件夹。
