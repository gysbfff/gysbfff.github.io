---
title: CTFHub Git泄露 Stash
date: 2026-09-26
categories: CTFHub Web
tags: CTFHub Web 信息泄露 Git泄露
---

# 题目: CTFHub Git泄露 Stash

## 题目描述
打开靶场，页面显示'Where is flag?',flag被git stash贮藏，本题为Git泄露。

## 解题思路
git stash可以保存工作区还没有commit提交的修改，该部分内容不会出现在git log提交日志中，普通版本回滚无法获取flag，需要操作stash堆栈读取临时贮藏。

## 解题步骤
1. 在靶场地址后面加上/.git/HEAD,出现文件下载弹窗判断为Git泄露
2. 关闭弹窗，打开powershell执行git-dumper下载整套git仓库 "http://challenge-1a7f70afe67a43c4.sandbox.ctfhub.com:10800/.git/" ./git_out2
3. 在仓库目录打开Git Bash，查看stash贮藏列表 git stash list
4. 直接打印贮藏的完整改动，获取flag git stash show -p stash@{0}

## 踩坑指南
1. 不要用git log查找flag，stash没有commit，不会出现在提交记录。
2. Windows记事本打开的文件不会自动刷新，优先在终端直接读取内容，避免被缓存误导。
 
## 知识点总结
1. git stash list ：列出全部临时贮藏记录
2. git stash show -p stash@{0} ：查看0号贮藏完整代码改动，不修改本地文件
3. git stash pop ：释放贮藏内容，同时删除这条贮藏记录
4. WIP：Work In Progress，代表未提交的工作现场
