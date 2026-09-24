---
title: CTFHub 备份文件下载 bak
date: 2026-09-24
categories: CTFHub Web
tags: CTFHub Web 信息泄露 bak备份
---

# 题目：CTFHub 备份文件下载 bak

## 题目描述
页面提示flag在index.php源码中，需要找到方式读取PHP源码。

## 解题思路
PHP文件访问会被服务器解析执行，无法直接看到源码。编辑器会生成'.bak'备份文件，访问index.php.bak下载源码。

## 解题步骤
1. 打开靶场页面，提示'Flag in index.php source code.'
2. 结合题目标题得知存在备份文件，访问'/inidex.php.bak'
3. 在浏览器下载bak文件，打开查看PHP源码 ，找到flag。

## 踩坑指南
1. 直接访问index.php，PHP代码会被执行，看不到源码。

## 知识点总结
1. '.bak'是编辑器自动生成的备份文件，上线前需要删除
2. php文件直接访问会被服务器解析，'.bak'不会被解析，直接返回原始源码
3. 
