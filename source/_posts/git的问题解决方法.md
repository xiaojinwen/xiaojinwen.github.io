---
title: git的问题解决方法
date: 2018-11-08 10:10:36
categories: ['其他']
tags: 其他
comments: true
---
# 正常提交文件
```bash
git init (第一次需要)
git add . (./表示所有文件  可指定文件夹/文件)
git remote add origin git@github.com:xxxxgit.git (第一次需要)(ssh链接上传代码大小没有限制)
git commit -m"xxxx"
git push -u origin master /git push origin master
```
# 不正常提交文件
## git push 不上去,因为远程仓库有修改但本地没有拉取
git stash 介绍
```bash
git stash /git stash save "info"  // 将文件保存到暂存区
git stash list // 查看暂存区列表
git stash pop 将暂存区的内容弹出,并应用到当前分支对应的工作目录上.
```
--rebase 介绍
告诉Git把小红的提交移到同步了中央仓库修改后的master分支的顶部

## 解决1
```bash
git pull --rebase origin master // 拉取远程仓库文件并把本地修改的文件置分支顶部
git push origin master 
```
rebase操作过程是把本地提交一次一个地迁移到更新了的中央仓库master分支之上。 这意味着可能要解决在迁移某个提交时出现的合并冲突，而不是解决包含了所有提交的大型合并时所出现的冲突。 这样的方式让你尽可能保持每个提交的聚焦和项目历史的整洁。反过来，简化了哪里引入Bug的分析，如果有必要，回滚修改也可以做到对项目影响最小。

如果小红和小明的功能是不相关的，不大可能在rebase过程中有冲突。如果有，Git在合并有冲突的提交处暂停rebase过程，输出下面的信息并带上相关的指令：

`CONFLICT (content): Merge conflict in <some-file>`

`git status` 命令来查看哪里有问题
接着小红编辑这些文件。修改完成后，用老套路暂存这些文件，并让git rebase完成剩下的事：
```bash
git add <some-file> 
git rebase --continue

git rebase --abort // 回到你执行git pull --rebase命令前的样子：
``````
## 解决2
```bash
git stash save "info"  // 将本次修改文件提交到暂存区
git pull --rebase origin master // 拉取远程仓库文件并把本地修改的文件置分支顶部
git stash pop // 放出暂存区修改
git push origin master
```
// master
git stash save "info"  // 将本次修改文件提交到暂存区
git checkout dev
git stash pop // 放出暂存区修改
# 开发新功能
```bash
git checkout -b marys-feature master
// 相当于
git checkout -b marys-feature
git merge master
```
这个命令检出一个基于master名为marys-feature的分支，Git的-b选项表示如果分支还不存在则新建分支。 这个新分支上，小红按老套路编辑、暂存和提交修改，按需要提交以实现功能：

`git push -u origin marys-feature`

这条命令push marys-feature分支到中央仓库（origin），-u选项设置本地分支去跟踪远程对应的分支。 设置好跟踪的分支后，小红就可以使用git push命令省去指定推送分支的参数。

```bash
git checkout master
git pull
git pull origin marys-feature
git push
```
# 合并分支
```bash
git merge --no-ff dev // 不使用fast-forward方式合并，保留分支的commit历史
git merge --squash dev // 使用squash方式合并，把多次分支commit历史压缩为一次 之后要提交一次
```
