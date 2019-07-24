---
layout: post
date: 2019-06-05 10:21
comments: true
toc: true
title: shell输出重定向问题
tags: 
- Shell
---


预期打印java版本信息
```bash
# str=$(java -version)
# 将java -version的输出结果赋值给str
str=`java -version`
echo $str
```

<!-- more -->

结果输出为空
> 原因:
> 屏幕上看到的输出来自stdout或stderr，命令结果赋值只会捕获来自stdout的内容，可以用`java -version 1>out 2>err`来验证一下: java -version输出是被重定向到stderr了
- [x] 解决方法:
```bash
# 将java -version的输出结果重定向到stdout, 然后赋值给str
str=`java -version 2>&1`
echo $str
```

