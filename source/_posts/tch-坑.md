title: nutch 坑
author: wen
tags:
  - nutch
categories: []
date: 2019-11-21 10:05:00
---

每次运行nutch都会在/tmp/hadoop-aaws/mapred/staging产生一个临时目录，若不定时清理，很容易inode(在同一个路径下，一级子目录的个数是有限制的)


# 统计当前文件夹下目录的个数:
```
ls -l |grep "^d"|wc -l
```

# 统计当前文件夹下文件的个数:
```
ls -l |grep "^-"|wc -l
```

#  统计当前文件夹下文件的个数，包括子文件夹里的 :
```
ls -lR|grep "^-"|wc -l
```

 
# 统计文件夹下目录的个数，包括子文件夹里的:
```
ls -lR|grep "^d"|wc -l
```