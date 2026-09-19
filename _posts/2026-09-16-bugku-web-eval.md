---
title: Bugku CTF eval
date: 2026-09-16
categories: CTF
tags: Bugku Web PHP eval注入 文件读取
---

# 题目: eval

## 题目描述
考点：PHP eval代码注入
访问靶机直接展示源码，存在eval高危函数，需要构造payload获取flag。
题目源码：
```php`
<?php
include "flag.php";
$a = @$_REQUEST['hello'];
eval( "var_dump($a);");
show_source(__FILE__);
?>

## 解题思路
1. 代码审计： $a 通过 $_REQUEST 接收hello参数，直接拼入 eval("var_dump($a);") 执行，存在PHP代码注入漏洞。
2. 第一次尝试payload： ?hello=$GLOBALS 
 $GLOBALS 可以打印内存中所有PHP变量，返回 Too Young Too Simple ，提交flag判定错误。
3. 分析失败原因：这个字符串只是变量 $flag 的伪装值，真正flag写在flag.php源码内部（可能藏在注释中）；PHP运行时注释不会解析成变量，因此 $GLOBALS 看不到。
4. 修改payload?hello=file('flag.php')
使用 file() 读取flag.php完整源码，文件里面所有内容（包括注释）都会被读取出来，拿到真正flag。

## 解题步骤
1. 复制原靶机页面网址，再加上?hello=file('flag.php')
2. 完整靶机访问链接为http://160.202.254.160:19762/?hello=file('flag.php')
3. 访问之后可以直接获得falg: flag{d1bcd58d1ede30055ad4bb02d48e69b5}

## 知识点总结
1. eval() 属于PHP高危函数，用户可控输入直接拼接进入eval会产生代码注入。
2. $GLOBALS 只能拿到运行期内存变量，无法读取PHP源码注释。
3. file() 函数读取磁盘文件原始源码，可以拿到文件中注释内隐藏内容。
4. $_REQUEST 可以接收GET、POST两种方式传参。
 
## 踩坑记录
1. 想当然认为 $GLOBALS 输出的flag变量的值就是答案，踩了伪装flag的坑，提交失败。
2. 分清两个概念：PHP运行后的变量值 ≠ 磁盘上原始php文件完整源码。如果flag藏在源码注释，就必须做文件读取。
