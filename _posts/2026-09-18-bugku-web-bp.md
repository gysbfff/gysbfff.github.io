---
tittle: Bugku CTF bp
date: 2026-09-18
categories: CTF
tags: Bugku web 爆破 BurpSuite 弱口令
---
# 题目：Bugku CTF bp

## 题目描述
登录页面，存在账号密码输入框，POST提交到`check.php`,提示`Wrong account or password!`，需要找到正确账号密码登录获取flag。

## 解题思路
1. 初步尝试SQL注入Payload（`admin'#` / `' or 1=1#`），页面依旧返回账号密码错误，判断**不存在SQL注入**。
2. 题目名称`bp`是BurpSuite缩写，提示考点为弱口令爆破。
3. 账号固定为`admin`，密码未知，使用Burp Intruder模块加载弱口令字典进行爆破。
4. 响应包特征：密码错误时返回`code:"bugku10000"`，页面提示账号密码错误；密码正确时会跳转`success.php`页面，返回flag。

## 解题步骤
1. **配置Burp代理**
打开BurpSuite，开启代理监听，浏览器配置代理，使数据包能被Burp捕获。
2. **抓包**
回到登录页面，账号填写`admin`，密码随便输入`123`，点击提交。Burp成功捕获POST数据包。
请求包示例：
```http`
POST /check.php HTTP/1.1
Host: 160.202.254.160:19295
Content-Type: application/x-www-form-urlencoded
Content-Length: 27
username=admin&password=123
3. 发送到Intruder爆破模块
右键数据包 →  Send to Intruder 。
进入 Positions 标签页，清除全部标记，仅在 password= 后面添加变量标记 § § ，只爆破密码字段。
4. 加载密码字典
切换到 Payloads ，选择 Payload type 为Simple list，导入top1000弱口令字典。
5. 设置匹配标记（Grep-Match）
打开 Options ，找到 Grep - Match ，添加字符串 bugku10000 。
(作用：所有返回包包含 bugku10000 代表密码错误；没有该字符串的请求就是正确密码。)
6. 启动爆破
点击 Start attack 开始爆破。
爆破完成后查看结果，找到响应包不包含bugku10000的记录，得到正确密码： zxc123 。
7. 登录拿flag
回到登录页面：
账号： admin 
密码： zxc123 
提交表单，页面跳转success.php，获取flag。
 
## 知识点总结
1. 题目名称是重要提示， bp 直接暗示BurpSuite爆破。
2. 登录题不一定是SQL注入，先简单测试注入，无效就考虑弱口令爆破。
3. 爆破判断技巧：利用页面返回的特征字符串区分正确/错误响应。
 
## 踩坑总结
1. 一开始误以为是SQL注入，反复提交注入Payload都失败，浪费时间。拿到题先看题目名字！
2. Intruder里一定要清理多余的 §§ 标记，只保留password变量，否则爆破会出错。
3. 抓包时记得开启浏览器网络面板的保留日志，防止页面刷新丢失数据包。
