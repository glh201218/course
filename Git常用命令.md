# Git常用命令

在使用命令之前我们先进行文件和文件夹的创建,创建文件夹`glh`进入文件夹创建文件`glh.txt`

## 初始化版本库

> git init

```bash
$ git init
```

## 添加到本地仓库

> git add 文件路径/文件夹

```bash
$ git add glh.txt
```

## 查看状态

> git status

```bash
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   glh/glh.txt

no changes added to commit (use "git add" and/or "git commit -a")

```

## 提交到本地仓库

> git commit -m "本次提交说明"

```bash
$ git commit -m glh/glh.txt
[master 69213b0] glh/glh.txt
 1 file changed, 2 insertions(+), 1 deletion(-)
```

## 查看修改内容

> git diff 文件名

```bash
$ git diff glh/glh.txt
diff --git a/glh/glh.txt b/glh/glh.txt
index f542bc1..32d968b 100644
--- a/glh/glh.txt
+++ b/glh/glh.txt
@@ -1,3 +1,4 @@
 你真帅
 热得快炸了
-过来哈
\ No newline at end of file
+过来哈
+发斯蒂芬
\ No newline at end of file
```

## 查看提交日志

> git log

```bash
$ git log
commit 69213b0636b4db7e73249840f39bf8f975a60577 (HEAD -> master)
Author: Your Name <you@example.com>
Date:   Fri Aug 7 20:57:50 2020 +0800

    glh/glh.txt

commit d8ec9e166f3c1f62740ee49bb57f1ac97ce2c223
Author: Your Name <you@example.com>
Date:   Fri Aug 7 20:32:27 2020 +0800

    第三次提交333333

commit 2ac4aaf8f11a6381f43cb7f7bf9018b07190bd17
Author: Your Name <you@example.com>
Date:   Fri Aug 7 20:29:16 2020 +0800

    第二次修改

commit 16314ddea47937733cf3590e6fe60bfc14b0f641
Author: Your Name <you@example.com>
Date:   Fri Aug 7 20:27:15 2020 +0800

    第一次提交
```

## 日志格式化

> git log --pretty=oneline --abbrev-commit

```bash
$ git log --pretty=oneline --abbrev-commit
1604983 (HEAD -> master, tag: v1.0, origin2/master, origin/master, dev) gasfdsd
6cbecb9 remove text.txt
efbcd5d text.txt
69213b0 glh/glh.txt
d8ec9e1 第三次提交333333
2ac4aaf 第二次修改
16314dd 第一次提交
```



## 查看日志时加入参数

> git log --pretty=oneline

```BASH
$ git log --pretty=oneline
69213b0636b4db7e73249840f39bf8f975a60577 (HEAD -> master) glh/glh.txt ##HEAD为当前版本
d8ec9e166f3c1f62740ee49bb57f1ac97ce2c223 第三次提交333333
2ac4aaf8f11a6381f43cb7f7bf9018b07190bd17 第二次修改
16314ddea47937733cf3590e6fe60bfc14b0f641 第一次提交
```

## 回到上个版本

> git reset --hard HEAD^  ##上个版本
>
> git reset --hard HEAD^^  ##上上个版本
>
> git reset --hard HEAD^^^  ##上上上个版本
>
> ##如果太多了可以写成
>
> git reset --hard HEAD~100 ##前100个版本

```bash
$ git reset --hard HEAD^
HEAD is now at d8ec9e1 绗笁娆℃彁浜?33333
```

## 返回到指定版本

   > git reset --hard 版本号

   ```bash
   $ git reset --hard 69213b0
   HEAD is now at 69213b0 glh/glh.txt
   ```

## 查看历史命令和结果

> $ git reflog

```bash
$ git reflog
69213b0 (HEAD -> master) HEAD@{0}: reset: moving to 69213b0
d8ec9e1 HEAD@{1}: reset: moving to HEAD^
69213b0 (HEAD -> master) HEAD@{2}: commit: glh/glh.txt
d8ec9e1 HEAD@{3}: commit: 第三次提交333333
2ac4aaf HEAD@{4}: commit: 第二次修改
16314dd HEAD@{5}: commit (initial): 第一次提交
```

## 查看工作区的文件和版本库里的区别

> git diff HEAD -- 文件名

```bash
$ git diff HEAD -- glh/glh.txt
diff --git a/glh/glh.txt b/glh/glh.txt
index f542bc1..91b79ba 100644
--- a/glh/glh.txt
+++ b/glh/glh.txt
@@ -1,3 +1,4 @@
 你真帅
 热得快炸了
-过来哈
\ No newline at end of file
+过来哈
+发顺丰
\ No newline at end of file
```

## 撤销本地修改,修改为暂存区或版本库内容

> git checkout -- 文件名

```bash
$ git checkout -- glh/glh.txt
```

## 撤销暂存区内容

> git reset HEAD 文件名

```bash
$ git reset HEAD glh/glh.txt
Unstaged changes after reset:
M       glh/glh.txt
```

## 删除版本库中文件

> git rm 文件名

```bash
$ git rm text.txt
rm 'text.txt'
# 然后提交
$ git commit -m "remove text.txt"
```

## 本地项目连接远程仓库

> git remote add origin 远程仓库地址

```bash
$ git remote add origin2 git@github.com:glh201218/test1.git
```

## 提交本地项目到远程

> git push -u origin2 master #第一次提交可以带上-u参数,将本地master和远程进行关联 ,以后就可以用__`git push origin master`__

```basj
$ git push -u origin2 master
Enumerating objects: 24, done.
Counting objects: 100% (24/24), done.
Delta compression using up to 4 threads
Compressing objects: 100% (8/8), done.
Writing objects: 100% (24/24), 1.73 KiB | 252.00 KiB/s, done.
Total 24 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), done.
To github.com:glh201218/test1.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin2'.
```

## 克隆远程仓库

> git clone 远程仓库地址

```bash
$ git clone git@github.com:glh201218/test1.git
Cloning into 'test1'...
Warning: Permanently added the RSA host key for IP address '52.74.223.119' to the list of known hosts.
remote: Enumerating objects: 24, done.
remote: Counting objects: 100% (24/24), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 24 (delta 1), reused 24 (delta 1), pack-reused 0
Receiving objects: 100% (24/24), done.
Resolving deltas: 100% (1/1), done.
```

## 创建分支

> git checkout -b 分支名称  # 参数 -b :创建并切换分支,相当于__`git branch 分支名称`__和__`git checkout 分支名称`__的结合

```bash
$ git checkout -b dev
Switched to a new branch 'dev'
```

## 查看分支

> git branch

```bash
$ git branch
* dev
  master
```

## 切换分支

> git checkout 分支名称 #新版本可用__`git switch 分成名称`__

```bash
$ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin2/master'.
```

## 创建并切换分支(新版)

> git switch -c 分支名称

## 合并分支

> git merge 要合并的分支

```bash
$ git merge dev
Already up to date.
```

## 删除分支

> git branch -d 分支名称

```bash
$ git branch -d dev
Deleted branch dev (was 1604983).
```

## 禁用快速合并 ##快速合并后如果删除分支后就无法找回分支了,log中没有记录

> git merge --no-ff -m "提交说明" 要合并的分支

```bash
$ git merge --no-ff -m "merge with no-ff" dev
Merge made by the 'recursive' strategy.
 readme.txt | 1 +
 1 file changed, 1 insertion(+)
```

## 储存当前分支未提交的工作内容

> git stash

```bash
$ git stash
Saved working directory and index state WIP on dev: f52c633 add merge
```

## 查看当前分支存储的信息

> git stash list

```bash
$ git stash list
stash@{0}: WIP on dev: f52c633 add merge
```

## 恢复储存的信息

> git stash apply

```bash
$ git stash apply
On branch master
Your branch is up to date with 'origin2/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   glh/glh.txt
```

## 恢复储存信息并删除

> git stash pop

```bash
$ git stash pop
On branch master
Your branch is up to date with 'origin2/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   glh/glh.txt

Dropped refs/stash@{0} (2ff1871a185db251c9b12ae37fdfb48d0c22d184)
```

## 指定恢复的信息

> git stash apply 信息key

```bash
$ git stash apply stash@{0}
Auto-merging glh/glh.txt
CONFLICT (content): Merge conflict in glh/glh.txt
```

## 复制指定提交内容到当前分支

> git cherry-pick 提交key

```bash
$ git cherry-pick 4c805e2
[master 1d4b803] fix bug 101
 1 file changed, 1 insertion(+), 1 deletion(-)
```

## 强制删除未被合并的分支

> git branch -D 分支名称

```bash
$ git branch -D feature-vulcan
Deleted branch feature-vulcan (was 287773e).
```

## 查看远程库信息

> git remote -v #参数 -v :详细信息

```bash
$ git remote
origin
origin2
```

## 查看远程分支

> git branch -r

```bash
$ git branch -r
  origin/master
  origin2/master
```

## 本地分支与远程分支进行关联

> git push --set-upstream 仓库名称 分支名称

## 推送分支到远程仓库

> git push 仓库名称(默认origin) 要推动的分支

## 远程有一个本地没有的分支,可以用

> ```bash
> git checkout -b 本地分支名称 origin/远程分支名  #会自动进行关联并切换分支
> git fetch origin 远程分支名:本地分支名 #不会会自动进行关联
> ```

## 拉去远程仓库信息到本地

> git pull

```bash
$ git pull
Already up to date.
```

## 将本地未push的分支变成一条直线

> git rebase

## 给分支打上标签

> git tag 标签内容

```bash
git tag v1.0
```

## 查看所有标签

> git tag

```bash
$ git tag
v1.0
```

## 对指定提交打上标签

> git tag 标签内容 提交id

```bash
$ git tag v2.0 d8ec9e1
```

## 对指定提交打上标签并加入提交信息

> git tag -a 标签名称 -m "提示信息" 提交id

```bash
$ git tag -a v2.2 -m "hahaha" 16314dd
```



## 查看标签信息

> git show 标签名称

```bash
$ git show v2.0
commit d8ec9e166f3c1f62740ee49bb57f1ac97ce2c223 (tag: v2.0)
Author: Your Name <you@example.com>
Date:   Fri Aug 7 20:32:27 2020 +0800

    第三次提交333333

diff --git a/glh/glh.txt b/glh/glh.txt
index 4062605..94a2b07 100644
--- a/glh/glh.txt
+++ b/glh/glh.txt
@@ -1,2 +1,2 @@
 你真帅
-帅炸了
\ No newline at end of file
+热得快炸了
\ No newline at end of file
```

## 删除标签

> git tag -d 标签名称

```bash
$ git tag -d v2.2
Deleted tag 'v2.2' (was 41ff510)
```

## 推送标签到远程

> git push origin 本地标签名

```bash
$ git push origin v1.0
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:glh201218/first-project.git
 * [new tag]         v1.0 -> v1.0
```

## 一次性推送所有标签

> git push origin --tags

```bash
$ git push origin --tags
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:glh201218/first-project.git
 * [new tag]         v2.0 -> v2.0
```

## 删除远程标签

> git push origin :refs/tags/标签名称

```bash
$ git push origin :refs/tags/v2.0
To github.com:glh201218/first-project.git
 - [deleted]         v2.0
```

