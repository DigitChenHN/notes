---
id: "2013580359443456919"
title: "git备忘"
author: "lambda"
type: zhihu-article
source: "https://zhuanlan.zhihu.com/p/2013580359443456919"
created: "2026-05-14 15:53"
updated: "2026-05-14 15:53"
downloaded: "2026-05-14"
---
## 引言

git是著名的版本管理工具，主要用来记录项目文件夹下的文件发生的变动，一般用于软件工程领域，可以方便多人协作，配合GitHub可以让开发者们有序地了解其他人的对代码的改动，从而在这个基础上进行改动防止冲突。

这篇文章主要用来记录常用的git命令和对于git的一些理解。

## git的设计

在git的这个文件版本管理系统中，主要分为工作区、暂存区、版本库和远程仓库。

-   工作区就是当前本地存放代码的位置。
-   暂存区（index/stage）是临时存放对文件的更改的地方。staged就是代表文件已经存放在暂存区，这个时候文件是被git追踪的状态。使用`git add`将文件添加到暂存区中。
-   版本库，将暂存在暂存区中的所有变更使用`git commit`命令提交到版本库中作为一次版本更新。

## 常用命令

### 基础命令

-   `git init`，在当前文件夹初始化一个git仓库，本质上就是创建一个.git文件夹并且初始化一些文件，这些文件就是用来记录这个项目的文件以及发生的变动的。
-   `git clone <url>`直接从远程拉去一个项目，这个项目中含有一个.git文件夹，也即一个git仓库。
-   `git add <file>`，将文件添加到暂存区。

-   `git add .` ，将所有未追踪的文件（untracked）和所有未staged的变更存放到暂存区中。

> 在工作区的一个文件第一次被添加到暂存区之后，那么该文件就会被追踪。  
> 之后当工作区的这个文件发生变更之后，就会跟存放在暂存区的文件对比，这样做产生了一些变更（changes），此时这些变更的状态为"Changes not staged for commit"。

-   `git commit -m 'message'`，将staged的文件变更提交到版本库中，并且配上文字说明。
-   `git commit -am "message"`， 将staged和unstaged的文件变更提交到版本库中，并配上文字说明。

> 同一个文件发生改变之后都会产生【变更】，变更存在于工作区，所以需要使用【add】命令添加到暂存区中，从暂存区【commit】到版本库中就是一个新的版本。而同一个文件每次都【add】似乎比较麻烦，因此使用-a选项可以使得unstaged的变更也提交（产生了unstaged changes的文件肯定是至少使用过一次【add】命令了，因此这个操作不会将你不会将你不希望添加到版本库中的无关文件添加【add】到暂存区）。  
> 每一次commit都是一个版本。

-   `git remote add origin <url>`，将一个远程仓库取一个别名为【origin】然后跟本地仓库链接。后续可以用origin来指代这个远程仓库。
-   `git push --set-upstream origin main`，将当前本地的分支和远程仓库origin的main联系起来并推送到远程的main分支中。之后就可以直接使用`git push`和`git pull`推送和拉取内容了。

### 远程仓库

-   `git fetch`，拉取远程最新信息，但不合并，这时候使用git log可以看到远程分支的最新情况。
-   `git pull`，从另一个仓库或者本地的其他分支拉取并整合

### 查看情况

-   `git status`，打印当前项目中untracked file、未staged的变更等等信息。
-   `git diff`，查看所有未staged的变更。即应该是对比**工作区**的和**暂存区**的文件之间的区别。
-   `git diff --staged`，查看**暂存区**和最后一次**提交的版本**之间的区别。
-   `git diff HEAD` ，查看最后一次**提交的版本**和**目前工作区**之间的区别。
-   `git log` ，查看版本日志，可以搭配各种多个选项如：--graph、--oneline、-all。

一般日志中会出现一些常见的符号，其含义如下：

| 符号           | 含义                                      |
| ------------ | --------------------------------------- |
| HEAD -> main | 当前所在位置：HEAD 指向 main 分支，main 指向这个 commit |
| origin/main  | 远程分支：远程仓库 origin 上的 main 分支位置           |
| origin/HEAD  | 远程默认分支：远程仓库默认指向的分支（通常是 main）            |
当项目中文件多的时候，使用git status需要查找untracked文件，需要很多时间，可以使用`git status --untrack-files=no`命令，这样git status不会列出未追踪的文件，是最快的方式。   
### 分支相关命令

-   `git switch -c <name>`或者`git checkout -b <name>`创建分支
-   `git checkout <branch_name>` 或者`git switch <branch_name>`，切换到指定分支

### 撤销相关命令

-   `git reset`，将当前所有暂存区的变更都unstaged。
-   `git commit --amend`，将staged的变更补充到最近一次提交的版本中去。一般用于提交了一次之后发现一些遗漏。但只建议在将版本推送到远程之前使用，amend一个已经推送到远程的commit然后强制推送，会导致一些问题。
-   `git rm --cache <file>`，这个命令是用来删除文件的，不加--cache选项就会删除该文件，加入--cache则会
-   `git restore <file>`，将工作区文件还原为目前的暂存区中的文件