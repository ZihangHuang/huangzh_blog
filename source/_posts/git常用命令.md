---
title: git常用命令
date: 2018-05-21 22:39:17
categories: 技术
tags: [git]
---

整理了一下 git等 的常用命令

<!--more-->

### 生成ssh key
```shell
ssh-keygen -t rsa -C "yourmail@gmail.com" 
```

### 打开ssh-agent
```shell
eval $(ssh-agent -s)
```

### 添加私钥
```shell
ssh-add ~/.ssh/id_rsa
```

### 创建或修改config文件
```shell
#github  
Host github.com #别名
HostName github.com #域名或ip
User 用户名
IdentityFile ~/.ssh/id_rsa
```

### 设置或删除git全局用户名和邮箱
```shell
git config --global --unset user.name
git config --global --unset user.email
git config --global user.name "yourname"
git config --global user.email "youremail"
```

```
ssh -T git@github.com
```

### 查看远程仓库

```shell
git remote -v
```

### 添加远程仓库

```shell
git remote add origin [地址]
```

### 拉取和提交

```shell
git fetch origin master #拉取远程仓库，但不会自动merge

git pull origin master #拉取远程仓库，并合并到当前分支
git pull origin master --allow-unrelated-histories #让本地仓库和远程仓库链接上，并拉取远程仓库

git pull = git fetch + git merge

git push origin master
```


### 版本回退

```shell
git reset HEAD^ #回退到上版本，不会重置工作区，会撤销git add .
git reset --mixed HEAD^    #默认形式，和 git reset HEAD^ 一样
git reset --hard HEAD^    #回退到上版本，会重置工作区，会撤销git add .
git reset --soft HEAD^    #回退到上版本，不会重置工作区，不会撤销git add .

git reset --hard HEAD^^    #回退到上上版本
git reset --hard [commit] #回退到某个版本号
git log   #查看提交日记

git push origin [分支名] -f 
#将回退同步到远程仓库，本地分支回滚后，版本将落后远程分支，必须使用强制推送覆盖远程分支

git reset --hard origin/分支 #用git reset撤回了提交，然后同步到了远端仓库，其他人可以用这个强制和远程版本一致，但是不建议使用
```

> 用`git reset --hard HEAD^` 回退到某一个以前版本后，再 git log 已经看不到新的版本。如果想撤销回退，就必须找到 append GPL 的 commit id。
> 当你命令行界面还没关闭时，你可以往上看那个新版本的 commit id，然后再`git reset --hard 版本号`。
> 如果已经关了界面，Git 提供了一个命令`git reflog`用来记录当前分支的提交记录，你可以找到那个新版本的 commit id。

### 撤销命令

把文件在工作区的修改全部撤销，这里有两种情况：
一种是文件自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
一种是文件已经 add 到暂存区后，又作了修改，要先`git reset [file]`，重置暂存区的文件，与上次commit保持一致，然后再`git checkout -- [file]`，撤销修改。

或者这两步可以用`git reset --hard`代替，但会撤销全部文件。

```shell
git checkout [file] #重置工作区文件
git checkout . #重置工作区全部文件
git reset [file] #重置暂存区的文件，与上一次commit保持一致，但工作区不变，是下面的简写形式
git reset --mixed HEAD [file]
```

### 分支

```shell
git branch -a #查看包括远程仓库的所有分支，如果远程仓库有了新分支，要先git pull拉下来

git checkout -b haha #创建新分支haha

git branch "本地新建分支名" "远程分支名" #本地新建分支和远程分支关联起来

git checkout master #切换到主分支

git merge dev #合并

git rebase master #变基，一般用在开发分支合并master分支的修改，和merge的区别是产生的历史记录是直线的，比较简洁

git branch -d dev #删除dev分支

git push origin -d dev # 删除远程分支

git branch -m "旧分支名" "新分支名"
```

### 查看修改内容

```shell
git diff #查看暂存区和工作区的差异
git diff --cached #查看暂存区和上一个commit的差异
git diff --name-only --diff-filter=U #查看冲突文件
```

### git stash

假如你在本地新建了一个dev分支，用于开发新功能。然后master分支有bug需要处理。你不想把还没完成的功能commit，但是不commit的话切换到master分支时依旧有dev分支的修改。这时候就可以在dev分支使用git stash，把修改内容保存到stash中，这样切换到master分支时，就会保持和原来的一致。
注意：需要`git add`后才能`git stash`

```shell
git stash #直接把代码保存到stash
git stash save "message" #保存到stash，并加上注释（不包含新文件）
git stash save -a "message" #保存到stash，并加上注释（包含新文件）

git stash list #查看所有stash

git stash pop #弹出stash里最后一个存储
git stash pop stash@{n} #弹出stash里指定的存储
git stash apply #弹出stash里最后一个存储，但stash里依旧存在
git stash apply stash@{n} #弹出stash里指定的存储，但stash里依旧存在

git stash drop stash@{n} #删除指定的stash
git stash clear #清空所有stash
```

### git tag

```shell
git tag <name> #新建一个标签，默认为HEAD，也可以指定一个commit id

git tag -a <tagname> -m "blablabla..." #可以指定标签信息

git tag #可以查看所有标签

git tag -d <name> #删除标签

git push origin <tagname> #推送tag到远程仓库

git push origin --tags #一次性推送全部尚未推送到远程的本地标签

git push origin :refs/tags/标签名  #删除远程标签(要先删除本地标签)
```

### 代码提交流程

1. 将本地修改的代码切换到一条本地新分支上（`git checkout -b dev`），注意切换前保证本地dev分支不存在
2. 在本地dev分支将代码追踪（`git add .`）
3. 在本地dev分支将代码提交（`git commit`）
4. 完成提交后，切换到本地master分支，此时的本地master分支应该是干净无任何修改的（`git checkout master`）
5. 在本地master分支上拉取远程master分支最新代码（`git pull`）
6. 切换到本地dev分支（`git checkout dev`），合并本地master分支（`git rebase master`），若有冲突就解决冲突并追踪（`git add .`），之后继续合并（`git rebase --continue`），直至代码完全合并
7. 切换到本地master分支（`git checkout master`），然后合并本地dev分支（`git merge dev`），注意合之前一定要保证本地dev分支是干净无修改的
8. 合并完成后，再推到远程master分支（`git push origin master`）
9. 清除本地dev分支（`git branch -d dev`），这一步根据自己实际情况选择是否清除


<!-- 如果post_asset_folder为false，图片统一放在source/images文件夹，用![](/images/image.jpg)引用
如果post_asset_folder为true，图片放在跟文章对应的同名文件夹的assets文件夹即可，引用语法如下,但每次新建文章都会创建对应的同名文件夹.
{% asset_img author.jpg 这是一个博客图片的说明 %} -->
