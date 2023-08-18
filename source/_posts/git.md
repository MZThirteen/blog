---
title: git
tags: git
categories: 工具
---

## 分支

```
<!-- 查看本地分支 -->
git branch

<!-- 查看远程分支 -->
git branch -r

<!-- 查看所有分支 -->
git branch -a

<!-- 删除不存在的远程跟踪分支 -->
git fetch -p

<!-- 删除本地分支 -->
git branch -d <branch_name>
<!-- 删除已经合并的分支 -->
git branch | xargs git branch -d
<!-- 强制删除当前分支之外的所有分支 -->
git branch | xargs git branch -D  

<!-- 删除远程分支 -->
git push origin --delete [branch_name]

<!-- 创建本地关联origin/dev的分支 -->
git checkout -b dev origin/dev
<!-- 或者 -->
git checkout -- dev origin/dev 
git checkout dev

<!-- 本地分支推送远程分支 -->
git push --set-upstream origin dev
```