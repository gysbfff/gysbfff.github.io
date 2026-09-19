---
title: Bugku CTF 变量1
date: 2026-09-13
categories: CTF,Bugku,Web
tags: Bugku,PHP,可变变量，eval,$GLOBALS
---

# 题目：变量1

## 题目描述
Web题型，页面直接输出PHP源码，提示flag in the variable，flag包含在flag.php文件的变量内，需要利用代码漏洞读出变量的值。

## 解题思路
 核心考点：**PHP可变变量 $$** + **$GLOBALS超全局数组**
1. 正则`/^\w+$/`限制参数只能是字母下划线，不能直接写函数、括号
2. `$$args`可变变量：$args的内容会被当成变量名
3. PHP内置`$GLOBALS`是特殊超全局变量，保存当前脚本全部全局变量，flag变量就在其中
4. 传入args=GLOBALS，$$args就等价于$GLOBALS，var_dump打印全部全局变量，拿到flag
--代码考点拆解
1. $_GET['args']  GET传参
2. 正则  /^\w+$/ ：只允许字母、数字、下划线，不能写括号、引号等特殊符号
3. $$args  → PHP可变变量！重点！
如果  $args = "GLOBALS" ，那么  $$args  等价于  $GLOBALS 
4. $GLOBALS  是PHP超全局数组，里面存放了当前页面所有全局变量！flag变量就保存在这里面！

## 解题步骤
1. 分析源码，发现eval里面存在`$$args`可变变量。
2. 构造GET参数 payload：`?args=GLOBALS`
3. 浏览器访问 `http://靶机地址/?args=GLOBALS`
4. 页面输出庞大数组，从中提取flag。

## 知识点总结
1. PHP可变变量：`$$a`，将$a的值作为新的变量名称。
2. `$GLOBALS`：PHP预定义超全局数组，存放所有全局变量。
3. `\w`正则元字符：代表字母、数字、下划线。
4. eval会把字符串当做PHP代码执行，是高危函数。

### 高危函数eval知识点
eval属于PHP经典高危函数，功能是将字符串作为PHP代码执行。
一旦用户输入可以流入eval内部，就有可能触发任意代码执行漏洞。
本题中eval受到正则`/^\w+$/`的限制，仅允许字母数字下划线，无法直接传入函数与分号，因此不能直接调用system等命令执行函数，只能利用PHP可变变量与$GLOBALS超全局变量获取flag。
