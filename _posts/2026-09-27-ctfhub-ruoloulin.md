---
title: CTFHub 弱口令
date: 2026-09-27
categories: CTFHub Web
tags: CTFHub Web 密码口令 弱口令
---
# 题目: CTFHub 弱口令

## 题目描述
登录页面，需要爆破账号密码获取flag

## 解题思路
页面POST提交name（用户名）和passward（密码）参数，无验证码，存在弱口令。登录失败与登录成功页面结构相似，不能依靠返回文字判断，依靠响应包长度差异区分是否登录成功。
使用Burp Intruder的 Cluster bomb 集束炸弹模式，对用户名、密码同时字典爆破。

## 解题步骤
1. 打开Burp Suite浏览器，输入靶场地址
2. 在登录页面账户处输入a，密码处输入b
3. 回到burt查看history，将post请求发送到intruder
4. 打开intruder，点击clear，选中最后一行a和b分别点击add
5. 在payload中选择1，分别添加admin，root，ctfhub，test等
6. 在paylonad选择2，分别添加123，123456，12345678，admin，admin888等
7. 修改攻击模式为Cluster bomb
8. 点击Start attack
9. 寻找长度与其他不同的一行（258）点开
10. 发现正确账户和密码为admin，123456
11. 返回靶场页面登录，获取flag

## 踩坑指南
1. 不要把完整请求粘贴进payload列表，payload只能放账号密码。
2 .本题成功页面依旧残留部分HTML，不能靠搜索报错字符串判断，必须对比响应长度。
3. Cluster bomb模式才是账号密码两两组合，不要误用Sniper。
 
## 知识点总结
- Sniper：单参数爆破

- Battering ram：两个参数使用同一套字典

- Cluster bomb：两套字典全部两两组合，适合账号密码爆破

- Pitchfork：账号密码按行一一对应
