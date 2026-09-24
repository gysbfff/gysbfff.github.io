---
title: CTFHub PHPINFO
date: 2026-09-24
categories: CTFHub Web
tags: CTFHub Web 信息泄露
---
# 题目：CTFHub PHPINFO

## 题目描述
访问靶场得到phpinfo页面，得到flag。

## 解题思路
phpinfo页面会打印服务器环境变量，flag存放在环境变量里面，直接页面搜索即可找到。

## 解题步骤
1. 访问靶场链接，打开phpinfo.php
2. 使用Ctrl+F页面内搜索，搜索flag即可定位flag

## 知识点总结
1. phpinfo() 用于查看PHP环境配置，开发调试使用。
2. 生产环境禁止对外开放phpinfo页面，会泄露服务器版本、路径、环境变量等敏感信息。
