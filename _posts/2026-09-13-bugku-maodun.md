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
1. 先搞懂  $_GET['num']  是什么
 -$_GET  代表URL GET传参，格式: shturl.cc/rzQBJ?参数名=参数值 
 这里参数名叫 num ，所以就是： ?num=xxx 
👉 这就是为什么要在网址后面拼接  ?num=1a 。
-只要代码出现  $_GET['xxx'] ，就代表可以在URL问号后面构造参数。
 如果是  $_POST['xxx'] ，就是POST传参，要在请求体里面传，不是URL。
2. 分析条件A： !is_numeric($num) 
   is_numeric(字符串) ：判断这个字符串是不是纯数字
-  is_numeric("123")  → true
-  is_numeric("12a3")  → false（包含字母，不是纯数字）
 !is_numeric  就是取反，必须返回false，所以 num 的值不能是纯数字。
3. 分析条件B： $num ==1  弱比较
 PHP双等号 == ：两边类型不一样，PHP会自动把字符串转数字。
字符串 "1a" 转数字的时候，读到字母就停止，只取前面的数字。
 "1a"  → 提取开头 1 ，于是  "1a" == 1  → true
4. 合并两个条件，找出payload
要求：
-1. 不是纯数字（带字母）
-2. 弱比较等于1
---想到：1后面跟字母， 1a 、 1abc  都满足。
排除错误尝试:
-  num=1 ：is_numeric("1")=true →  !true 不成立，进不去大括号
-  num=a1 ： a1==1  弱比较为false，提取不到数字1

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

