---
title: Bugku CTF 隐写
date: 2026-09-13
categories: CTF,Bugku,Misc
tags: Bugku,png, IHDR,图片隐写，Python
---

## 题目：隐写

## 题目描述
下载图片打开只能看到 Bugku 字样，图片下半部分被截断，flag无法直接看到。
-考点：篡改PNG图片IHDR块高度。

 ## 解题思路
 PNG文件IHDR头部存储图片宽高。出题人手动修改IHDR里的高度值，图片渲染时按篡改后的高度截断画面，但图片像素数据完整保留在文件内。
 用Python读取图片二进制，修改IHDR块的高度字段，另存新图片，查看完整画面拿到flag。
 
 ## 解题步骤
1. 下载图片`2.png`。
2. 脚本读取IHDR，查看原始宽高：
 ```python```
 import struct
 def get_png_width_height(file_path):
     with open(file_path,"rb") as f:
         data = f.read()
     if data[:8] != b'\x89PNG\r\n\x1a\n':
         print("不是PNG图片！")
         return
     ihdr_start = 8
     ihdr_data = data[ihdr_start+8 : ihdr_start+8+13]
     width, height = struct.unpack(">II", ihdr_data[0:8])
     print(f"图片宽度 = {width}")
     print(f"图片高度 = {height}")
   运行输出：宽度500，高度420。
3. 修改IHDR高度，另存新图片：
   with open("2.png","rb") as f:
    data = bytearray(f.read())
#修改IHDR的高度字段为500，大端序写入
new_height = 500
data[20:24] = new_height.to_bytes(4, byteorder='big')
with open("new_2.png","wb") as f:
    f.write(data)
print("已生成 new_2.png")
   return width,height
 if __name__ == "__main__":
     get_png_width_height("2.png")
4. 打开 new_2.png ，图片底部出现flag： BUGKU{a1e5aSA} 
 
## 知识点总结
1. PNG由文件签名 + 多个chunk组成，IHDR是首个chunk，保存图片基础信息。
2. IHDR宽高使用4字节大端序存储。
3. 修改IHDR高度不会删除像素数据，只是改变图片渲染范围，是Misc中经典PNG隐写手法。
4. 常见图片后缀区分
-  .png ：无损、支持透明、分块存储（CTF隐写高频考点）
-  .jpg / .jpeg ：有损、文件更小、没有分块结构
-  .gif ：动图
-  .bmp ：无压缩位图，体积巨大   
  
