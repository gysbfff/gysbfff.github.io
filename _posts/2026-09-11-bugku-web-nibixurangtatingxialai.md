---
title: "Bugku CTF 你必须让他停下来 Writeup"
date: 2026-09-11
categories: Bugku CTF
tags:
  - Bugku
  - CTF
  - Web
  - 前端调试
---
# 题目：你必须让他停下
## 题目描述
页面每隔0.5秒自动刷新，提示 Stop at panda I u will get flag。
## 解题思路
页面会自动刷新，线索藏在被CSS隐藏的a标签内，只有返回熊猫图片的页面才会出现flag。
## 解题步骤
1. 访问靶机页面，F12打开开发者工具
2. 在设置按钮中找到禁用Java,页面停止自动刷新
3. 在Elements元素面板找到隐藏标签：`<a style="display:none">flag{xxx}</a >`
4. 将`display:none`中的`none`修改为`block`，回车确认
5. 页面上直接展示出完整flag
## 踩坑
一开始以为是图片轮播就尝试在控制台中输入了clearlnterval(timer);但是后面报错了，根据ai提示了解了这是页面整页刷新。
1. 选择了先禁用JavaScript,刷新页面后一直按F5，但是一直不出现熊猫图片
2. 根据ai了解到可以用python的脚本自动循环寻找，但是速度太慢了
3. 查看网页源代码，按F5刷新了靶机网页，用Ctrl+F搜索flag，但是没有抓到
## Flag
flag{536fb395da1b3166374bae04c83b664}
## 总结
1. `display:none`只是前端隐藏元素，内容仍然存在页面DOM中。禁用JS阻止页面刷新，修改CSS属性即可让隐藏内容显现。这道题考察前端基础调试能力。
2. F12可以打开控制面板，F5可以刷新靶机网页
