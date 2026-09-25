---
title: CTFHub Git泄露 Log
date: 2026-09-25
categories: CTFHub Web
tags: CTFHub Web 信息泄露 Git泄露 
---

# 题目： CTFHub Git泄露 Log

## 题目描述
点开靶场之后页面出现' Where is flag？'.网站存在git泄露，flag被开发者提交后删除，需要从git历史版本恢复flag。

## 解题思路
网站禁止直接访问/.git/目录（返回403），但可以访问/.git/HEAD，确认存在Git源码泄露。

## 解题步骤
1. 浏览器访问/.git/HEAD，出现下载任务。
2. 下载文件后选择全部解压
3. 点击文件夹进去后文件空白处shift+鼠标右键打开powershell执行pip install git-dumper直接pip安装
4. 安装完成后执行git"靶场地址/.git/"./git_out
5. 运行完后打开git_out文件夹
6. Shift+右键空白处选择Git Bash，输入git log
7. 复制添加flag的提交的commit号'git reset --hard 粘贴commit编号
8. 打开文件夹出现的flag.php,拿到flag

## 踩坑指南
1. dvcs‑ripper为Perl脚本，**无法pip install安装**，Windows环境不友好，不要使用pip安装它。
2. 靶场返回403访问`/.git/`目录，但是单个`.git/HEAD`文件可访问，确认git源码泄露。
3. git clone失败：靶场不支持git智能HTTP握手，浏览器能GET文件，但git克隆握手报错。
4. 推荐工具：git‑dumper（Python3，Windows直接pip安装）下载完整.git仓库。

## 知识点总结
1. Git源码泄露：网站把.git文件夹部署到web目录，攻击者下载仓库拿到源码与历史提交。
2. git log：查看全部版本提交日志。
3. git reset --hard 哈希值：强制回滚到指定版本，恢复已经删除的文件。
4. ### 如何挑选正确commit
git log按时间倒序展示提交，顶部为最新版本：
-`remove flag`：最新提交，执行删除flag操作，此时文件不存在。
- `add flag`：前一次提交，flag文件被新增，回滚到此版本恢复flag.php。
- init：仓库初始化，不存在flag文件。
5. 当前大量开发人员使用git进行版本控制，对站点自动部署。如果配置不当,可能会将.git文件夹直接部署到线上环境。这就引起了git泄露漏洞。
   克尝试使用BugScanTeam的GitHack
