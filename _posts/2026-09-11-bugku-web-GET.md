---
title: Bugku CTF web 基础$_GET Writeup
date: 2026-09-11
categories: Bugku CTF
tags: Bugku CTF Web PHP GET传参
---
# 题目 web基础$_GET

## 题目描述
Web，分值10分。
页面直接给出PHP源码：
```php```
$what=$_GET['what'];
echo $what;
if($what=="flag")
echo 'flag{****}';

## 解题思路
PHP的 $_GET['what'] 用来读取URL GET参数what。
代码逻辑：URL传入what=flag，满足 $what=="flag" 条件，页面输出flag。
GET参数语法： url?参数名=值 
 
## 解题步骤
1. 启动靶机，访问页面，看到源代码。
2. 在靶机URL后面拼接  ?what=flag 
3. 访问拼接后的链接，页面输出flag

## 知识点总结
1. $_GET 接收URL查询参数，属于PHP基础。
2. GET传参格式： url?key=value 
3. 简单代码审计，根据if判断构造输入。
