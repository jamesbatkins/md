﻿---
title: checkout 检出和切换
date: 2016-9-6 14:00:02
tags: [java,git,checkout]
categories: java
---

这个命令是个多功能的命令用法很灵活。
切换分支、撤消修改
下面文中是 `--` 两个杠框是连在一起，中间没有空格，因为字体太小，所说中间给个空格看着明显一些。

### 1.切换分支
```
git checkout <name>
```
### 2.创建并切换分支
```
git checkout -b <name>
这其实可以拆解成两步操作 -b 应该就是branch
```

### 3.撤销工作区修改
实际就是“以旧换新”的操作
有两种情况：  
1.如果未添加到暂存区，则把版本库中的最新版本覆盖  
2.如果已添加到暂存区，则把暂存区中的修改拿出覆盖

撤销工作区修改:
```
git checkout - - <file>
```
清除全部 - - 不能丢，不然就成了上面的切换切支命令了:
```
git checkout - - .
```

### 4.连招

假如有一个文件，做了修改，但是不确定后面的修改是不是想要的。先添加到暂存区中，过了一会这个修改是不想要的，想要把工作区的文件从暂存区撤回覆盖。
```
git add                   //放一份当前写到一半觉得没问题的放到暂存区中
git checkout - - file     //将 版本库 中的修改替换到 工作区中
```
没有 add 的情况下，直接拿版本来覆盖本地，这样搞的话，之前工作区的文件的修改就没了。用这招就看之前修改要不要了。
```
git checkout - - file
```
