---
layout: post
date: 2019-06-05 10:21
comments: true
toc: true
title: Git命令技巧
category: Linux & 运维
tags:
- Shell命令
---


# Git命令技巧

## git命令中文显示问题

![image](https://user-images.githubusercontent.com/17871962/62198397-205d9580-b3b4-11e9-9625-603d9b895284.png)

<!-- more -->

解决方法:
- [x] 去除引号 
`git config --global core.quotepath false`

## git branch
### 1. 删除远程分支

~~这个删除得是：远程追踪。也就是你本机的git不会再去监控这个分支了！~~
 `git branch -r -d origin/分支名`

真正正确的命令
`git push origin -d 分支名` 或者 `git push origin --delete 分支名`

### 2. 删除本地分支
 `git branch -D 分支名`

### 3. 查看远程分支
`git branch -r`

### 4. 查看本地分支 
`git branch`

### 5. 查看本地和远程分支
 `git branch -a`

### 6. 更新本地远程已删除的分支 
我已将自己的远程分支删除了，同事使用git branch -a依然可以看到那些删除的远程分支； 这块情况下，需要使用： 
`git remote prune origin`

### 7. 查看已删除分支情况
 `git remote show origin`

## git clean
### git 删除未跟踪文件
```bash
# 删除 untracked files
git clean -f
# 连 untracked 的目录也一起删掉
git clean -fd
# 连 gitignore 的untrack 文件/目录也一起删掉 （慎用，一般这个是用来删掉编译出来的 .o之类的文件用的）
git clean -xfd
# 在用上述 git clean 前，墙裂建议加上 -n 参数来先看看会删掉哪些文件，防止重要文件被误删
git clean -nxfd
git clean -nf
git clean -nfd
```
## git diff
### git 使用vimdiff解决冲突
首先配置git difftool为vimdiff
```
git config --global diff.tool vimdiff 
git config --global difftool.prompt false 
git config --global alias.d difftool
```

使用 `git d HEAD 文件名` 比对差异
左边窗口为HEAD版本  右边为merge结果

```
Ctrl+W 两次可切换左右窗口
]c     跳到下一个差异处  
[c     跳到上一个差异处
zo     打开折叠(open)
zc     关闭折叠(close)
dp     将此文件差异处的改动应用到另外一个文件(do put)
do     将另外一个文件差异处的改动应用到此文件(do get)
```

## git reset
### 撤销commit
在git push的时候，有时候我们会想办法撤销git commit的内容 
 git log找到之前提交的git commit的id 
```bash
# 完成Commit命令的撤销，但是不对代码修改进行撤销，可以直接通过git commit 重新提交对本地代码的修改
git reset id    
# 完成撤销,同时将代码恢复到前一commit_id 对应的版本 
git reset --hard id  
```
## git merge
### 分支合并到主分支时，去掉分支的冗余提交
git 分支合并到主分支时，去掉分支的冗余提交。即，将分支的多次提交一次性合并到主分支上。
```bash
# 切换到主分支
git checkout master 
# 一次性合并分支的多次提交
git merge --squash dev
# 将刚‘合并的提交’提交到主分支master 
git commit -m 'xxx版' 
```
- `git merge --no-ff`  指的是强行关闭fast-forward方式。
fast-forward方式就是当条件允许的时候，git直接把HEAD指针指向合并分支的头，完成合并。属于“快进方式”，不过这种情况如果删除分支，则会丢失分支信息。因为在这个过程中没有创建commit

- `git merge --squash` 是用来把一些不必要commit进行压缩，
比如说，你的feature在开发的时候写的commit很乱，那么我们合并的时候不希望把这些历史commit带过来，于是使用--squash进行合并，此时文件已经同合并后一样了，但不移动HEAD，不提交。需要进行一次额外的commit来“总结”一下，然后完成最终的合并。

总结：
```
--no-ff： # 不使用fast-forward方式合并，保留源分支的commit历史
--squash：# 使用squash方式合并，把多次分支commit历史压缩为一次
```

## git拉取代码
### `git fetch` 和 `git pull`

1.`git fetch` 相当于是从远程获取最新到本地，不会自动merge，如下指令：

```bash
# 将远程仓库的master分支下载到本地当前branch中
git fetch orgin master 
# 比较本地的master分支和origin/master分支的差别
git log -p master  ..origin/master
# 进行合并
git merge origin/master
```

也可以用以下指令：

```bash
# 从远程仓库master分支获取最新，在本地建立tmp分支
git fetch origin master:tmp
# 將當前分支和tmp進行對比
git diff tmp
# 合并tmp分支到当前分支
git merge tmp
```

2.`git pull`：相当于是从远程获取最新版本并merge到本地
 `git pull origin master`
`git pull` 相当于从远程获取最新版本并merge到本地
在实际使用中，`git fetch`更安全一些

