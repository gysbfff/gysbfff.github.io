---
title: CTFHub 备份文件下载 vim缓存
date: 2026-09-24
categories: CTFHub Web
tags: CTFHub Web 信息泄露 vim备份 swp
---

# 题目： CTFHub 备份文件下载 vim缓存

## 题目描述
页面提示flag存放在index.php源码中，需要找到源码读取方式。

## 解题思路
PHP直接访问会被服务器解析执行，看不到源码。vim编辑器异常退出会生成`.index.php.swp`交换备份文件，访问该文件下载源码。

## 解题步骤
1. 访问靶场首页，提示flag在index.php源码。
2. 构造URL访问`/.index.php.swp`，下载vim交换文件。
3. 打开swp文件，读取index.php源代码，找到flag提交。

## 踩坑指南
1. 文件名前面的'.'不能漏掉：`.index.php.swp`
2. 不能写成`.index.php.bak`

## 知识点总结
1. vim编辑文件会生成`.文件名.swp`交换文件，异常退出不会自动删除。
2. swp文件属于源码备份，暴露在web目录会造成源码泄露。
3. 当开发人员在线上环境中使用 vim 编辑器，在使用过程中会留下 vim 编辑器缓存，当vim异常退出时，缓存会一直留在服务器上，引起网站源码泄露。
4. 常见vim备份后缀：`.swp`、`.swo`、`.swn`。

#### bak 和 swp 的区别
- **.bak**：通用编辑器（Notepad++等）生成的完整备份文件。命名格式`index.php.bak`，文件名前面不带小数点；保存文件时生成，不会自动删除。
- **.swp**：Vim编辑器专属临时交换文件。命名格式`.index.php.swp`，开头带小数点，属于Linux隐藏文件；打开文件瞬间生成，正常关闭vim会自动删除，只有vim崩溃异常退出才会残留。
- 共同点：两者都属于源码备份泄露，浏览器直接访问文件，服务器不会解析PHP，直接返回原始源码。
