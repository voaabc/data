---
layout: post
title: 如何用 Python 模糊搜索文件
categories: Python
tags: Python Script
---

* content
{:toc}


### 1

```python
import os
path = '/home/ubuntu01/test/script_project1_files/images' 

files = os.listdir(path)
# os.listdir() 方法用于返回指定的文件夹包含的文件或文件夹的名字的列表。这个列表以字母顺序。 它不包括 '.' 和'..' 即使它在文件夹中。

只支持在 Unix, Windows 下使用。

print(files)

for f in files:
    if f.endswith('.png') and 'fish' in f:
        print('Look! I found this' + f)
```
