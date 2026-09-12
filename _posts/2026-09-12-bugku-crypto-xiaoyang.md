---
title: Bugku CTF｜聪明的小羊
date: 2026-09-12
categories: CTF, Crypto
tags: 栅栏密码, rail fence
---

# 聪明的小羊

## 题目描述
> 一只小羊翻过了2个栅栏 fa{fe13f590lg6d46d0d0}
提示“翻过2个栅栏”，代表**2栏栅栏密码**。
⚠️ 关键点：密文需要把 `fa{fe13f590lg6d46d0d0}` 完整带入解密，不能只取大括号里面的字符串。

## 解题思路
栅栏密码（Rail Fence Cipher），2栏。
加密时字符按照之字形排布，解密时逆向还原字符顺序。

## Python 解密脚本
```python```
def rail_fence_decrypt(cipher, rails=2):
    length = len(cipher)
    cycle = 2 * rails - 2
    plain = [''] * length
    ptr = 0
    for r in range(rails):
        step1 = cycle - 2 * r
        step2 = 2 * r
        pos = r
        toggle = True
        while pos < length:
            plain[pos] = cipher[ptr]
            ptr += 1
            if r == 0 or r == rails -1:
                pos += cycle
            else:
                if toggle:
                    pos += step1
                else:
                    pos += step2
                toggle = not toggle
    return ''.join(plain)

-执行后可得flag{6fde4163df05d900}

## 知识点总结
 栅栏密码属于置换类加密，不会改变字符本身，只打乱字符顺序。
“翻过n个栅栏”就是 n 栏栅栏密码，是CTF Crypto签到题高频考点。
