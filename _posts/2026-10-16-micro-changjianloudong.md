---
title: 新手常见漏洞
time: 2026-10-16
categories: micro
tags: micro
---
# 题目：新手常见漏洞

<table>
  <thead>
    <tr>
      <th>漏洞名</th>
      <th>通俗解释</th>
      <th>例子</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>SQL注入</td>
      <td>在输入框里输入特殊字符，骗数据库吐数据</td>
      <td><code>' or 1=1--</code></td>
    </tr>
    <tr>
      <td>文件上传</td>
      <td>网站让你传图片，你传个木马上去</td>
      <td>传 <code>.php</code> 文件</td>
    </tr>
    <tr>
      <td>文件包含</td>
      <td>网站能读文件，你让它读 flag</td>
      <td><code>?file=../../flag</code></td>
    </tr>
    <tr>
      <td>命令执行</td>
      <td>网站执行系统命令，你注入自己的命令</td>
      <td><code>; cat /flag</code></td>
    </tr>
    <tr>
      <td>XSS</td>
      <td>往网页里塞 JavaScript，让别人中招</td>
      <td><code>&lt;script&gt;alert(1)&lt;/script&gt;</code></td>
    </tr>
    <tr>
      <td>目录遍历</td>
      <td>用 <code>../</code> 跳出网站目录，读服务器敏感文件</td>
      <td><code>?file=../../../../etc/passwd</code></td>
    </tr>
  </tbody>
</table>
