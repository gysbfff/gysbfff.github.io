---
title: Pikachu XXE - XML 外部实体注入漏洞 (源码审计与实战)
date: 2026-11-01 
categories: [Web安全, 靶场Writeup]
tags: [Pikachu, XXE漏洞, 源码审计, 文件读取]
---

## 题目: Pikachu XXE - XXE漏洞

## 题目描述
页面提供了一个接收 XML 数据的 API 接口。要求利用 XXE（XML 外部实体注入）漏洞，读取服务器上的系统敏感文件。

## 解题思路
通过审计后端源码 `xxe_1.php` 发现，代码使用了 `simplexml_load_string($xml, 'SimpleXMLElement', LIBXML_NOENT)`。其中 **`LIBXML_NOENT` 参数强制开启了外部实体解析**，这就是漏洞的根源。同时，代码使用了 `@` 符号屏蔽报错，导致如果文件包含特殊字符（如 `&`）导致解析失败时，页面会毫无回显。因此，我们需要读取一个内容纯净的系统文件来验证。

## 解题步骤
**1. 源码审计（⚠️ 核心踩坑记录）**
* 打开后端源码，发现 `LIBXML_NOENT` 开启了实体解析。
* **踩坑记录**：起初尝试读取 `win.ini` 或 `php://filter`，页面一直空白。原因在于 `@` 屏蔽了错误，且 `win.ini` 中的特殊字符破坏了 XML 解析。
* **破局思路**：必须选择一个纯净、不含特殊符号的系统文件，比如 Windows 的 `hosts` 文件。

**2. 构造 Payload**
* 提交以下 XML，利用 `SYSTEM` 伪协议读取 Windows 的 hosts 文件：
```xml
<?xml version="1.0"?>
<!DOCTYPE note [
  <!ENTITY xxe SYSTEM "file:///C:/Windows/System32/drivers/etc/hosts">
]>
<x>&xxe;</x>
```
**3. 验证回显**
* 提交后，页面成功回显了 127.0.0.1 localhost 以及大量 # 号注释，证明成功读取了系统底层文件，XXE 漏洞利用成功。

## 知识点总结
* LIBXML_NOENT 的作用：开启后会使 XML 解析器解析外部实体，是导致 XXE 漏洞的直接原因。
* 实战中的“盲区”：@ 符号屏蔽错误会增加盲注难度。在实战中，如果遇到 XXE 无回显，可以考虑使用 DNSLog 等外带通道（OOB）来验证。

### 防御策略：
* 绝不开启 LIBXML_NOENT，或者使用 libxml_disable_entity_loader(true) 彻底禁用外部实体。
* 使用 JSON 格式替代 XML。
