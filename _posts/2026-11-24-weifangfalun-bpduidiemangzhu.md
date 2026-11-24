---
title: BurpSuite SQL盲注微方法论
date: 2026-11-24
categories: [Web安全, 微方法论]
tags: [micro,BurpSuite, SQL盲注, 布尔盲注, 时间盲注, Intruder, Repeater]
---

# BurpSuite SQL盲注微方法论
> 微方法论定位：总结一套可复用操作流程，用于在无直接数据回显场景，借助Burp完成**布尔盲注、时间盲注**手工+半自动化测试；仅用于授权靶场学习。
> 适用场景：不存在Union回显，无数据库报错输出；依靠**页面内容差异**或**HTTP响应时间**间接推断数据。

## 前置基础概念
1. **布尔盲注**：SQL条件真假，页面返回文本、长度存在明显差异，以此判断条件结果。
2. **时间盲注**：真假条件页面返回完全一致；利用`sleep()`延时函数，通过请求响应耗时判断条件是否成立。
3. MySQL核心函数
- `substr(str,offset,len)`：截取字符串，offset从1开始
- `ascii(char)`：将字符转为ASCII数字，盲注优先数字比较，规避引号转义问题
- `if(expr,true_res,false_res)`：条件判断，时间盲注核心

> Burp模块分工
- **Repeater**：漏洞确认、手工调试Payload、确认基线响应/基线耗时
- **Intruder**：半自动化猜解字符；社区版仅单线程，速度受限
- **Grep‑Match**：布尔盲注用来匹配页面成功特征；**时间盲注不能使用Grep，依靠Elapsed耗时列**

## 1 整体执行流程（标准化步骤）
> 流程顺序：抓包复制原始请求 → Repeater验证漏洞与基线 → 确定闭合与注入位置 → 选择盲注类型 → Intruder配置自动化猜解 → 结果整理拼接

### 步骤1：抓包获取原始请求
1. Burp开启代理，浏览器访问业务，触发注入点请求；
2. 右键请求包，`Send to Repeater`。
> 注入点可以是GET参数、POST表单参数、Cookie参数。

### 步骤2：Repeater确认漏洞、采集基线
#### 2.1 布尔盲注基线验证
构造真假条件对比，观察页面返回内容。
```http
# 条件为真
xxx?param=1' and 1=1 -- -
# 条件为假
xxx?param=1' and 1=2 -- -
```
- 真：出现业务成功特征（登录成功、特定关键词、内容长度大）
- 假：页面空白、报错、无对应关键词
> 如果两个请求返回页面存在稳定差异 → **布尔盲注可行**。

#### 2.2 时间盲注基线验证（页面无任何差异时使用）
先发送原始请求，记录 Repeater 右下角`Elapsed`基线耗时（网络正常一般几十 ms）。
构造时间 Payload：条件成立延时 3 秒
```
param=1' and if(ascii(substr(database(),1,1))>1,sleep(3),1) -- -
```
- Elapsed >3000ms → 条件成立
- Elapsed 接近基线耗时 → 条件不成立 
> sleep 建议设置 3‑5 秒，规避网络抖动造成误判。

### 步骤 3：确定闭合符号
常见闭合：`'`、`")`、`')`、数字型无闭合。
> 
> 坑：闭合错误会直接造成 SQL 语法错误，盲注无报错输出，表现为请求全部为假；必须先在 Repeater 调试正确闭合。

### 步骤 4：Payload 模板
#### 布尔盲注模板
```
# 判断数据库名第pos位ASCII是否大于num
param=1' and ascii(substr(database(),§pos§,1)) > §ascii_num§ -- -
```

#### 时间盲注模板
```
param=1' and if(ascii(substr(database(),§pos§,1)) > §ascii_num§,sleep(3),1) -- -
```
> 
> 可替换：`database()` → 查询表、查询字段子查询语句。

### 步骤 5：Intruder 自动化配置
1. Repeater 数据包右键 `Send to Intruder`
2. Positions：`Clear §`清除自动标记，手动添加两个标记位`§pos§`、`§ascii_num§`；攻击类型选择`Cluster bomb（集束炸弹）`
3. Payloads
   - Payload set1（字符串位置 pos）：Numbers，从 1~20，步长 1
   - Payload set2（ASCII 码）：Numbers，33‑126（全部可见打印字符）
4. Options
   - **布尔盲注**：开启`Grep‑Match`，填入页面成功特征字符串（如`YOU ARE LOGGED IN AS`），命中代表条件 True
   - **时间盲注**：关闭 Grep 匹配；攻击完成人工查看`Elapsed`耗时列，大于 sleep 设定值即为真
5. `Start attack`启动攻击，筛选 True 结果，拼接字符得到数据。

## 2 关键踩坑点清单
1. **时间盲注网络抖动误判**：必须先测基线耗时，sleep 不要设置过小；同一个 payload 多次测试确认。
2. **编码问题**：GET 请求`'`、`#`需要 URL 编码；POST 表单 raw 模式直接写原始 payload，Burp 会自动编码。
3. **闭合丢失**：盲注环境无报错，闭合写错全部请求返回 False，很难排查，务必 Repeater 先调通。
4. Intruder 社区版单线程，猜解速度慢，适合原理学习；实战可以使用 SQLmap 或 Python 脚本。
5. 堆叠注入场景时间盲注：需要注意多语句分隔符`;`，Payload 格式：`param=1;select if(...,sleep(3),1)-- -`。

## 3 盲注选型决策树
```
有无页面内容稳定差异？
├─有 → 优先【布尔盲注】，使用Grep‑Match匹配特征
└─无 → 使用【时间盲注】，依靠Elapsed响应时间判断
```

## 4 微方法论总结
1. 盲注不要直接上 Intruder 自动化；**Repeater 手工验证漏洞、闭合、基线是前提**；
2. 布尔盲注看页面文本特征，时间盲注看请求耗时；二者不要混用配置；
3. 使用`ascii()+substr`数字比较，减少引号逃逸带来的麻烦；
4. 无报错盲注环境，语法错误不会提示，真假响应对比是判断的核心依据。
