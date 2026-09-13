---
title: Bugku CTF 矛盾
categories: CTF
tags: Web,PHP弱类型,is_numeric
---

# 题目：矛盾

## 题目描述
Web题，给出PHP源码。
```php```
$num=$_GET['num'];
if(!is_numeric($num))
{
    echo $num;
    if($num==1)
    echo 'flag{**********}';
}

## 解题思路
考点：PHP弱类型比较与 is_numeric() 函数特性。
1. is_numeric()  检测参数是否为数字字符串， !is_numeric($num) 要求传入的参数不能是纯数字，才能进入代码块。
2. 内部判断  $num ==1  是弱比较，PHP会自动提取字符串开头的数字进行对比。
3. 构造 payload： num=1a 
-  is_numeric("1a")  返回false，满足外层if条件
-  "1a" ==1  弱比较结果为true，输出flag

## 解题步骤
1. 使用GET传参，构造参数 num=1a 
2. 访问地址： http://ip:port/?num=1a 
3. 页面得到flag，提交。

## 通用做题套路
1. 先看接收参数是  $_GET  还是  $_POST 
-  $_GET  → URL问号构造参数
-  $_POST  → 请求体传参（Hackbar/Burp）
2. 把每一个 if 条件全部列出来，全部条件必须同时满足
3. 逐个分析每个函数的特性，看它限制了什么
4. 寻找矛盾点，构造同时满足所有限制的字符串（payload）

## 知识点总结
1.  is_numeric() ：判断是否为数字/数字字符串，含有字母则返回false。
2. PHP  ==  双等号：弱类型比较，两边类型不同时自动转换类型。字符串 1a 和数字1比较时，会自动提取开头数字 1 进行对比。
-注意： ===  是强比较，不会自动转换类型。

