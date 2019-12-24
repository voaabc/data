---
layout: post
title: 用 Python 实现文件查找（BIF实现及队列实现）
categories: Python 脚本 图形界面
tags: Python Script GUI
---

* content
{:toc}


## 利用内置函数实现文件查找



### 功能：返回用户输入的文件的绝对路径


### 设计思路


1. 用户输入在哪个盘进行查找
2. 遍历此盘文件，若为目标文件则输出
3. 无此文件，则输出错误


### 实验代码
```python
#查找某个目录下的目标文件
import os       #引入操作系统模块
import sys      #用于标准输入输出

def search(path,name):

    for root, dirs, files in os.walk(path):  # path 为根目录
        if name in dirs or name in files:
            flag = 1      #判断是否找到文件
            root = str(root)
            dirs = str(dirs)
            return os.path.join(root, dirs)
    return -1


path = input('请输入您要查找哪个盘中的文件（如：D:\\\）')
print('请输入您要查找的文件名：')
name = sys.stdin.readline().rstrip()  #标准输入,其中rstrip()函数把字符串结尾的空白和回车删除
answer = search(path,name)
if answer == -1:
    print("查无此文件")
else:
    print(answer)

```
