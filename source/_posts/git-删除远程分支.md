---
title: git 删除远程分支
date: 2019-02-20 17:38:51
tags: Git
categories: 
- Git
---

`git branch -r` 查看远程分支
`git branch -r -d origin/AirportProducts` 删除本地远程分支 `AirportProducts` branch-name
`git push origin :AirportProducts` 删除远程仓库的分支 