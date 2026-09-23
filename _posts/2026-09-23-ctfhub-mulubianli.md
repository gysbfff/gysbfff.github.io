---
title: CTFHub 信息泄露 目录遍历
date: 2026-09-23
categories: CTFHub Web
tags: CTFHub Web 信息泄露
---

# 题目: CTFHub 目录遍历

## 题目描述
访问靶场页面，Apache服务器开启目录浏览功能，页面展示Index of /flag_in_here/ .../...,列出目录内文件找到falg。

## 解题思路
Apache有一个配置选项： Indexes 。开启之后，当访问的文件夹下没有index.html/index.php默认首页，服务器就会返回目录文件列表（Index of页面），直接展示这个目录下所有文件。
考点：了解Apache目录浏览漏洞，逐级访问目录，找到存放flag的文件。
 
## 解题步骤
1. 访问靶场链接，页面出现 Index of /flag_in_here/.../... ，说明开启目录浏览。
2. 页面能看到文件  flag.txt ，直接点击  flag.txt 。
3. 打开文件，读取里面的flag，提交。
-补充：本题目录是多层  /flag_in_here/.../.../ ，需要一层一层点开目录，直到最后一层，看到flag.txt。

## 踩坑指南
1. 漏看层级： flag_in_here  →  ... →  ...，一层一层进入目录。
2. 目录浏览是Apache配置不当造成的，属于信息泄露，不是代码漏洞
 
## 知识点总结
1. Apache配置  Options Indexes ：无默认首页时，列出目录全部文件。
2. 危害：泄露网站目录结构、敏感文件（源码、密钥、flag等）。
3. 安全修复：
- 修改Apache配置，删掉indexes(本题不需要）
  -拓展
   1. 点击左下角Windows开始菜单，在应用列表找到文件夹 XAMPP，点击里面的 XAMPP Control Pa。
   2. 打开XAMPP Control Panel，在Apache那一行，点 Config 按钮，在弹出菜单，选择  Apache (httpd.conf) ，直接打开配置文件。
   3. 找到 httpd.conf XAMPP： xampp\apache\conf\httpd.conf 
   4. 找到<Directory>标签,修改Options
      #### 修改前（危险）
      Options Indexes FollowSymLinks
      #### 修改后（安全）
      Options FollowSymLinks
  5. 保存文件，重启Apache服务。
- 放置index默认首页；
- 敏感文件不要放在web可访问目录。
