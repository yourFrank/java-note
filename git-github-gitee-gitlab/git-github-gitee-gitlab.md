视频地址：https://www.bilibili.com/video/BV1vy4y1s7k6?from=search&seid=17540342641792214588&spm_id_from=333.337.0.0

# git介绍

Git 是一个免费的、开源的**分布式版本**控制系统，可以快速高效地处理从小型到大型的各种项目。

Git 易于学习，占地面积小，性能极快。 它具有廉价的本地库，方便的暂存区域和多个工作流分支等特性。 其性能优于 Subversion、 CVS、 Perforce 和 ClearCase 等版本控制工具。

版本控制其实最重要的是**可以记录文件修改历史记录**，从而让用户能够查看历史版本，方便版本切换。

个人开发-> 团队协作，因此需要版本控制工具



## 分布式版本控制VS集中式版本控制

### 集中化的版本控制

集中化的版本控制系统诸如 CVS、 SVN 等，都有一个单一的集中管理的服务器，保存所有文件的修订版本，而协同工作的人们都通过客户端连到这台服务器，取出**最新的文件**或者**提交更新**。

* 优点：每个人都可以在一定程度上看到项目中的其他人正在做些什么。而管理员也可以轻松掌控每个开发者的权限，并且管理一个集中化的版本控制系统， 要远比在各个客户端上维护本地数据库来得轻松容易。
* 缺点：中央服务器的单点故障。如果服务器宕机一小时，那么在这一小时内，谁都无法提交更新，也就无法协同工作。

![image-20220311160234171](https://image.imxyu.cn/file/image-20220311160234171.png)

> 并且每个人本地都是最新的文件，没有保存历史记录，如果中央服务器数据丢失没法使用每个人本地的进行恢复所有的历史记录

### 分布式版本控制工具

像 Git 这种分布式版本控制工具，客户端提取的不是最新版本的文件快照，而是把代码仓库**完整地镜像**下来（本地库）。这样任何一处协同工作用的文件发生故障，事后都可以用其他客户端的本地仓库进行恢复。因为每个客户端的每一次文件提取操作，**实际上都是一次对整个文件仓库的完整备份**。

分布式的版本控制系统出现之后,解决了集中式版本控制系统的缺陷：

1. 服务器断网（故障）的情况下也可以进行开发（因为版本控制是在大家本地进行的），还可以提交到本地，只是没办法推送到服务器而已。并且这些服务器都是在github\码云这种大公司，基本也不会出现故障
2. 每个客户端保存的也都是整个**完整的项目**（包含历史记录， 更加安全），服务器出现问题便于恢复历史记录

![image-20220311160641656](https://image.imxyu.cn/file/image-20220311160641656.png)

> 每个人本地都有一个本地的代码仓库，代码修改之后首先会保存在每个人的本地仓库。 然后同步到中央服务器上，其他成员更新本地文件的时候首先要去中央服务器下去拉取最新的代码

## git 发展历史

![img](https://image.imxyu.cn/file/3f25cba01665ab8dcb63fcc79d05f04f.png)

> 小故事：
>
> git 也是linus 大神写的，最初是开发了linux 然后开源了，然后很多人看着不错就提出建议，lin大神打开他们的代码发现确实写的不错，然后就复制到自己的代码中，但是后来越来越多人觉得linux 太好用了就纷纷提pr ，Lin 大神就忙不过来了，一天的时间都在看处理这个问题，后来BitKeeper 软件商看不下去了，就让Lin大神免费用他们的软件Bitkeeper，但是用归用别破解就行，就这样用了3年，后来linux 社区中的一个程序员觉得这个东西有意思，就把这个东西破解了然后发了个帖子，后来被BitKeeper 发现了就收回了软件的使用权，Lin大神就很难受肯定不能回到以前自己手动的时代啊，就闭关花了2周的时间写了一个git 出来开源了（参没参考BitKeeper就不知道了），后来慢慢的有了github 上，代码托管中心包含了很多项目。
>
> Lin 大神的伟大开源精神，真的值得我们一辈子去学习。。。

## Git 工作机制和代码托管中心

![image-20220311162735074](https://image.imxyu.cn/file/image-20220311162735074.png)

举个例子：如果我们喝醉了在工作区新添加了一段骂老板的代码然后添加到了暂存区，第二天酒醒了发现了这个问题我们可以把工作区和暂存区都可以删掉并且不会留下历史记录

但是如果我们commit提交到了本地库，此时就会产生了一条历史记录，此时所有用你这个代码的同事、老板都能看到这条历史记录，就算提交了一个旧的覆盖一下还是能找到这个历史记录

如果我们提交到了远程库，不止你们公司那么全世界都能看到你新添加的代码了

> 注意这里的写代码的工作区并不是说在我们的idea上，而是在对应的磁盘上对应的是磁盘的文件目录

**代码托管中心**

代码托管中心是基于网络服务器的远程代码仓库，一般我们简单称为**远程库**。

- 局域网（如果不想让全世界的人看到的话，比如公司内部的代码，就可以搭建这个Gitlab）
  - GitLab
- 互联网
  - GitHub（外网）
  - Gitee 码云（国内网站）

# Git安装

官网地址： https://git-scm.com/
查看GNU 协议，可以直接点击下一步。

其中有几个步骤这里解释一下：

一般都是通过右键打开的，所以第一个桌面快捷方式不用选，第二个就是右键菜单打开

![image-20220311164433470](https://image.imxyu.cn/file/image-20220311164433470.png)

默认的git 编辑器，vim 最舒服用过linux 的无脑上手

![image-20220311164513163](https://image.imxyu.cn/file/image-20220311164513163.png)



这个是第一次初始化git 仓库的时候 仓库的名字，第一个默认是master ，第二个是可以自定义比如main 啥的

![image-20220311164521172](https://image.imxyu.cn/file/image-20220311164521172.png)



git 的环境变量，第二个是可以在其他软件中使用git 命令，比如说windows 自带的cmd 中使用。 一般来说我们就选第一个就行

![image-20220311164604260](https://image.imxyu.cn/file/image-20220311164604260.png)



后台连接协议

![image-20220311164648980](https://image.imxyu.cn/file/image-20220311164648980.png)

配置Git文件的行末换行符， Windows使用 CRLF Linux使用 LF，选择第一个自动转换，然后继续下一步。

![image-20220311164656455](https://image.imxyu.cn/file/image-20220311164656455.png)



终端类型：

![image-20220311164807934](https://image.imxyu.cn/file/image-20220311164807934.png)



git pull 合并的方式

![image-20220311164819495](https://image.imxyu.cn/file/image-20220311164819495.png)



![image-20220311164839752](https://image.imxyu.cn/file/image-20220311164839752.png)



实验室功能，
技术还不成熟， 有已知的 bug，不要勾选，然后点击右下角的 Install按钮，开始安装 Git。

![image-20220311164852136](https://image.imxyu.cn/file/image-20220311164852136.png)

> 一般来说上面的所有步骤选取默认的，一直next就可以了

# Git 常用命令



| 命令名称                             | 作用              |
| ------------------------------------ | ----------------- |
| git config --global user.name 用户名 | 设置用户签名      |
| git config --global user.email 邮箱  | 设置用户email地址 |
| git init                             | 设置用户email地址 |
| git status                           | 初始化本地库      |
| git add 文件名                       | 添加到暂存区      |
| git commit -m “日志信息” 文件名      | 提交到本地库      |
| git reflog                           | 查看历史记录      |
| git reset --hard 版本号              | 版本穿梭          |

## 设置用户签名

```cmd
git config --global user.name abc #有户名
git config --global user.email abc@123.com

```

说明：签名的作用是区分不同操作者身份。用户的签名信息在每一个版本的提交信息中能够看到，以此确认本次提交是谁做的。 Git 首次安装必须设置一下用户签名，否则无法提交代码。

**注意**： 这里设置用户签名和将来登录 GitHub（或其他代码托管中心）的账号没有任何关系。

查看设置过用户签名

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop
$ cat ~/.gitconfig
[user]
        name = abc
        email = abc@123.com
[core]
        quotepath = false

```

或者直接进入当前用户的文件夹下查看这个文件即可

![image-20220311173439612](https://image.imxyu.cn/file/image-20220311173439612.png)

## 初始化本地库

基本语法：`git init`

案例实操：

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop
$ mkdir HelloGit

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop (master)
$ cd HelloGit/

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ ls -al
total 4
drwxr-xr-x 1 abc 197121 0 Jul  5 20:18 ./
drwxr-xr-x 1 abc 197121 0 Jul  5 20:19 ../

# 前面是创建文件夹的操作，从下面开始
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
# 初始化本地仓库
$ git init
Initialized empty Git repository in C:/Users/abc/Desktop/HelloGit/.git/

# 创建了一个名为.git非空隐藏文件夹 ，通过ls -a 可以看到所有隐藏的文件，这里看到有一个.git文件夹
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ ls -al
total 8
drwxr-xr-x 1 abc 197121 0 Jul  5 20:19 ./
drwxr-xr-x 1 abc 197121 0 Jul  5 20:19 ../
drwxr-xr-x 1 abc 197121 0 Jul  5 20:19 .git/

```

## 查看本地库状态

基本语法：`git status`

案例实操：

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop
$ cd HelloGit/

# 首次查看（工作区没有任何文件）
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git status
On branch master

No commits yet

nothing to commit (create/copy files and use "git add" to track)

# 新增一个文件（hello.txt）
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ vim hello.txt

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ cat hello.txt
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!

# 再次查看（检测到未追踪的文件）
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git status
On branch master

# 这句是说没有提交历史
No commits yet

# 显示有一个未追踪的文件hello.txt
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        hello.txt

nothing added to commit but untracked files present (use "git add" to track)

```

## 添加暂存区

基本语法：`git add 文件名`

案例实操：

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
# 首先查看一下状态，显示有一个未追踪的文件hello.txt
$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        hello.txt

nothing added to commit but untracked files present (use "git add" to track)

#添加至暂存区
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git add hello.txt
warning: LF will be replaced by CRLF in hello.txt.
The file will have its original line endings in your working directory

# 再查看状态
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git status
On branch master

No commits yet
# 这里提示缓存区的文件，并且可以通过这个命令去移除缓存区中的文件
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   hello.txt

#移除暂存区的hello.txt
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git rm --cached hello.txt
rm 'hello.txt'

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        hello.txt

nothing added to commit but untracked files present (use "git add" to track)

# 再次添加
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git add hello.txt
warning: LF will be replaced by CRLF in hello.txt.
The file will have its original line endings in your working directory

```

## 提交本地库

基本语法：`git commit -m "日志信息" 文件名`

案例实操：

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
# 首先看一下状态，然后git add先添加到暂存区
$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        hello.txt

nothing added to commit but untracked files present (use "git add" to track)

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git add hello.txt
warning: LF will be replaced by CRLF in hello.txt.
The file will have its original line endings in your working directory


#接下来提交本地库
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git commit -m "first commit"
[master (root-commit) b0006bc] first commit
 1 file changed, 19 insertions(+)
 create mode 100644 hello.txt

# 此时再看一下状态
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git status
On branch master
nothing to commit, working tree clean

```

查看提交记录

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
# 这个是查看简要的信息 ，第一个是版本号的前几位，然后指针指向
$ git reflog
b0006bc (HEAD -> master) HEAD@{0}: commit (initial): first commit

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
# 这个是查看详细的历史记录，包括全版本号，提交的作者信息（这个就是我们一开始git config 设置的）
$ git log
commit b0006bc538b98b7eb77b4b4aaa17b6e333c4289e (HEAD -> master)
Author: abc <abc@123.com>
Date:   Tue Jul 6 00:38:24 2021 +0800

    first commit

```

> 这个提交记录的命令后面我们会详细说

## 修改文件

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)

$ vim hello.txt

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ cat hello.txt
hello, git!222
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!


abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
# 查看当前状态
$ git status
#提示 hello.txt 被修改过（modified）
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   hello.txt

no changes added to commit (use "git add" and/or "git commit -a")

# 将修改的文件再次添加暂存区
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git add hello.txt
warning: LF will be replaced by CRLF in hello.txt.
The file will have its original line endings in your working directory

# 第2次提交
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git commit -m "second commit"
[master 6967bf0] second commit
 1 file changed, 1 insertion(+)

# 查看提交日志
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git reflog
6967bf0 (HEAD -> master) HEAD@{0}: commit: second commit
b0006bc HEAD@{1}: commit (initial): first commit

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git status
On branch master
nothing to commit, working tree clean

```

## 版本穿梭

### 查看历史版本

基本语法：

- `git reflog` 查看版本信息,
- `git log` 查看版本详细信息，包含了完整的版本号，还有提交作者的信息

git log 命令可以显示所有提交过的版本信息，如果感觉太繁琐，可以加上参数 --pretty=oneline，只会显示版本号和提交时的备注信息。

git reflog 可以查看所有分支的所有操作记录（包括已经被删除的 commit 记录和 reset 的操作）。例如，执行 git reset --hard HEAD~1，退回到上一个版本，用git log则是看不出来被删除的commitid，用git reflog则可以看到被删除的commitid，我们就可以买后悔药，恢复到被删除的那个版本。link



```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git reflog
41f776b (HEAD -> master) HEAD@{0}: commit: third commit
6967bf0 HEAD@{1}: commit: second commit
b0006bc HEAD@{2}: commit (initial): first commit

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git log
commit 41f776ba69ea06c42bd449098d818ab73608d4dd (HEAD -> master)
Author: abc <abc@123.com>
Date:   Tue Jul 6 01:18:21 2021 +0800

    third commit

commit 6967bf01bdcbc8e326f1b8c45d45db8bd4c0c868
Author: abc <abc@123.com>
Date:   Tue Jul 6 01:02:10 2021 +0800

    second commit

commit b0006bc538b98b7eb77b4b4aaa17b6e333c4289e
Author: abc <abc@123.com>
Date:   Tue Jul 6 00:38:24 2021 +0800

    first commit

```

### 版本穿梭

基本语法：`git reset --hard 版本号`

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
# 首先查看当前的历史记录
$ git reflog
41f776b (HEAD -> master) HEAD@{0}: commit: third commit
6967bf0 HEAD@{1}: commit: second commit
b0006bc HEAD@{2}: commit (initial): first commit

# 切换到 b0006bc 版本，也就是第一次提交的版本
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git reset --hard b0006bc
HEAD is now at b0006bc first commit

# 切换完毕之后再查看历史记录，当前成功切换到了 b0006bc 版本
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git reflog

# 这里的第一条还会记录我们的穿梭的日志
b0006bc (HEAD -> master) HEAD@{0}: reset: moving to b0006bc
41f776b HEAD@{1}: commit: third commit
6967bf0 HEAD@{2}: commit: second commit
# 此时head指针指向的我们第一次的记录
b0006bc (HEAD -> master) HEAD@{3}: commit (initial): first commit

# 然后查看文件 hello.txt，发现文件内容回到第一版本
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ cat hello.txt
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!


```

> 注意上面使用git reflog可以查看所有的退回的记录，如果我们使用git log 查看的话就只有当前指针之前的历史记录，退回前的那几条历史记录就没了
>
> 对于上面这种回退后使用git log查看的话，就只会有这一条了。如果我们还想回到后面的记录的话还是需要使用git reflog查看当前指针后面的版本号进行回退
>
> b0006bc (HEAD -> master) HEAD@{3}: commit (initial): first commit

## 底层实现

注意这些所有的操作底层都是通过一个head指针的移动和版本号来实现的

我们打开这个.git 文件夹，里面有一个head文件，这个就是保存了我们当前指针所指向的分支，我们点进去看看

![image-20220311184236048](https://image.imxyu.cn/file/image-20220311184236048.png)

这就是我们当前所在的分支，然后我们点进去这个分支文件夹中的这个master文件

![image-20220311184332594](https://image.imxyu.cn/file/image-20220311184332594.png)



这就是我们所在的版本号

![image-20220311184405272](https://image.imxyu.cn/file/image-20220311184405272.png)



因此底层就是通过head指针的移动确定所在的分支，分支中有指向我们当前所在的版本号



# 分支

## 分支的概述和优点

![img](https://image.imxyu.cn/file/bcad650a512a72097b3391e00ecb8bbe.png)

我们首先来看一下这张图，比如说我们现在的线上代码master 目前是用户正在使用的分支，如果我们要开发一个新功能，我们不能直接在这个线上分支上进行修改，而应该拉出一个分支，修改后测试没有问题后再合并到master分支。不能影响用户的使用（master）

## 什么是分支

在版本控制过程中，同时推进多个任务，为每个任务，我们就可以创建每个任务的单独分支。使用分支意味着程序员可以把自己的工作从开发主线上分离开来， 开发自己分支的时候，不会影响主线分支的运行。对于初学者而言，分支可以简单理解为副本，一个分支就是一个单独的副本。（分支底层其实也是指针的引用）


![某项目有四条分支](https://image.imxyu.cn/file/f1d0659ed000e9dfa295fc696a58cf74.png)

* （A程序员）master是我们的主干分支，此时A程序员接到一个需求就是加个功能修改个颜色，此时我们就拉出一个分支feature-blue，然后进行修改没问题之后再合并到master, 合并之后用着用着发现有问题了，此时我们就又拉出一个分支hot-fix 然后修改测试，没问题了再合并到master
* （B程序员） 同时呢架构师给B程序员接到一个需求，就是加一个游戏的功能，B就在master上拉了一个进行游戏开发，然后这个功能可能比较难，就一直测试迭代，最后没问题完成之后合并到了master分支

### 分支的好处

同时并行推进多个功能开发，提高开发效率。

各个分支在开发过程中，如果某一个分支开发失败，不会对其他分支有任何影响。失败的分支删除重新开始即可

## 分支的查看&创建&切换

| **命令名称**        | **作用**                     |
| ------------------- | ---------------------------- |
| git branch 分支名   | 创建分支                     |
| git branch -v       | 查看分支                     |
| git checkout 分支名 | 切换分支                     |
| git merge 分支名    | 把指定的分支合并到当前分支上 |

### 查看分支

基本语法：`git branch -v`

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git branch -v
* master b0006bc first commit

```

### 创建分支

基本语法：`git branch 分支名`

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git branch hot-fix

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git branch -v
  hot-fix b0006bc first commit
* master  b0006bc first commit

```

**刚创建的新的分支，并将主分支master的内容复制了一份到创建的hot-fix分支**。

### 切换分支

基本语法：`git checkout 分支名`

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git checkout hot-fix
Switched to branch 'hot-fix'

# 切换到刚创建的分支上
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (hot-fix)
$ git branch -v
* hot-fix b0006bc first commit
  master  b0006bc first commit

```

切换分支后，在新分支修改文件：

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (hot-fix)
$ vim hello.txt

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (hot-fix)
$ cat hello.txt
hello, git!hot-fix
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (hot-fix)
$ git status
On branch hot-fix
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   hello.txt

no changes added to commit (use "git add" and/or "git commit -a")

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (hot-fix)
$ git add hello.txt

#在hot-fix提交
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (hot-fix)
$ git commit -m "hot-fix first commit"
[hot-fix 25f62d6] hot-fix first commit
 1 file changed, 1 insertion(+), 1 deletion(-)

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (hot-fix)
$ git reflog
25f62d6 (HEAD -> hot-fix) HEAD@{0}: commit: hot-fix first commit
b0006bc (master) HEAD@{1}: checkout: moving from master to hot-fix
b0006bc (master) HEAD@{2}: reset: moving to b0006bc
5f8dbf6 HEAD@{3}: commit: forth commit
b0006bc (master) HEAD@{4}: reset: moving to b0006bc
41f776b HEAD@{5}: commit: third commit
6967bf0 HEAD@{6}: commit: second commit
b0006bc (master) HEAD@{7}: commit (initial): first commit


```

## 合并分支(正常合并)

基本语法：`git merge 分支名` (意思是将`分支名` 合并到当前的分支上，注意这个意思)

在 master 分支上合并 hot-fix 分支（将hot-fix的合并到master）。

```cmd
# 首先要切换到master分支上
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (hot-fix)
$ git checkout master
Switched to branch 'master'

#将hot-fix的合并到master
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git merge hot-fix
Updating b0006bc..25f62d6
Fast-forward
 hello.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

#合并后，可以在master分支上看到hot-fix上分支对文件的修改
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ cat hello.txt
hello, git!hot-fix
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!

#合并后，hot-fix分支依然存在，并未消失
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git branch -v
  hot-fix 25f62d6 hot-fix first commit
* master  25f62d6 hot-fix first commit


abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git checkout hot-fix
Switched to branch 'hot-fix'

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (hot-fix)
$

```

因为在master 分支上这个文件并没有进行修改过，只有在hot-fix分支上做过修改。所以此时合并不会有冲突问题，正常合并

## 合并分支(冲突合并)

### 冲突产生的原因

并分支时，两个分支在同一个文件的同一个位置有两套完全不同的修改。 Git 无法替我们决定使用哪一个，因此，必须**人为决定**新代码内容。

### 产生冲突

首先，在master修改文件hello.txt最后一行内容，并提交：

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ vim hello.txt

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ cat hello.txt
hello, git!hot-fix
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!master test

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   hello.txt

no changes added to commit (use "git add" and/or "git commit -a")

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git add hello.txt

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git commit -m "master test"
[master fb0e30b] master test
 1 file changed, 2 insertions(+), 2 deletions(-)


```

然后，在hot-fix修改文件hello.txt最后一行内容，并提交：

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git checkout hot-fix
Switched to branch 'hot-fix'

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (hot-fix)
$ vim hello.txt

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (hot-fix)
$ cat hello.txt
hello, git!hot-fix
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!hot-fix test

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (hot-fix)
$ git add hello.txt

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (hot-fix)
$ git commit -m "hot-fix test"
[hot-fix 47d2d8f] hot-fix test
 1 file changed, 1 insertion(+), 1 deletion(-)


```

切换到master分支，然后将hot-fix分支的合并到master，冲突产生：

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (hot-fix)
$ git checkout master
Switched to branch 'master'


abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
# 合并操作， 冲突产生
$ git merge hot-fix
Auto-merging hello.txt
CONFLICT (content): Merge conflict in hello.txt
Automatic merge failed; fix conflicts and then commit the result.

# MERGING 出现，表示有冲突待解决
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master|MERGING)
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   hello.txt

no changes added to commit (use "git add" and/or "git commit -a")

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master|MERGING)
$ cat hello.txt
hello, git!hot-fix
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
<<<<<<< HEAD
hello, git!hot-fix test
hello, git!master test
=======
hello, git!
hello, git!hot-fix test
>>>>>>> hot-fix


```

### 解决冲突

编辑有冲突的文件，**删除特殊符号**，决定要使用的内容

特殊符号：

```cmd
<<<<<<< HEAD
当前分支的代码 
======= 
合并过来的代码 
>>>>>>> hot-fix

```

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master|MERGING)
$ cat hello.txt
hello, git!hot-fix
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
<<<<<<< HEAD
hello, git!hot-fix test
hello, git!master test
=======
hello, git!
hello, git!hot-fix test
>>>>>>> hot-fix

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master|MERGING)
$ vim hello.txt

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master|MERGING)
$ cat hello.txt
hello, git!hot-fix
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!hot-fix test
hello, git!master test


abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master|MERGING)
$ git add hello.txt

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master|MERGING)
# 这里要注意，解决完冲突后的commit 后面不能加文件名，直接commit 就可以了，否则会报错
$ git commit -m "conflict solved"
[master 785ab46] conflict solved

# (master|MERGING)的|MERGING消失了，冲突解决
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git reflog
785ab46 (HEAD -> master) HEAD@{0}: commit (merge): conflict solved
fb0e30b HEAD@{1}: checkout: moving from hot-fix to master
47d2d8f (hot-fix) HEAD@{2}: commit: hot-fix test
25f62d6 HEAD@{3}: checkout: moving from master to hot-fix
fb0e30b HEAD@{4}: commit: master test
25f62d6 HEAD@{5}: checkout: moving from hot-fix to master
25f62d6 HEAD@{6}: checkout: moving from master to hot-fix
25f62d6 HEAD@{7}: checkout: moving from hot-fix to master
25f62d6 HEAD@{8}: checkout: moving from master to hot-fix
25f62d6 HEAD@{9}: merge hot-fix: Fast-forward
b0006bc HEAD@{10}: checkout: moving from hot-fix to master
25f62d6 HEAD@{11}: commit: hot-fix first commit
b0006bc HEAD@{12}: checkout: moving from master to hot-fix
b0006bc HEAD@{13}: reset: moving to b0006bc
5f8dbf6 HEAD@{14}: commit: forth commit
b0006bc HEAD@{15}: reset: moving to b0006bc
41f776b HEAD@{16}: commit: third commit
6967bf0 HEAD@{17}: commit: second commit
b0006bc HEAD@{18}: commit (initial): first commit

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$

```

> 注意：
>
> 1、冲突是以文件为单位的，虽然这里都是改的最后一行，但是如果我们其中一个改的是其他行（非最后一行），也就是说改的不是一个行上的话也会出现冲突需要我们去解决
>
> 2、我们虽然解决了master上的冲突，但是hot-fix 还是之前的

### 创建分支和切换分支原理

master、 hot-fix 其实都是指向具体版本记录的指针。当前所在的分支，其实是由 HEAD决定的。所以创建分支的本质就是多创建一个指针。

HEAD 如果指向 master，那么我们现在就在 master 分支上。
HEAD 如果执行 hot-fix，那么我们现在就在 hot-fix 分支上。
所以切换分支的本质就是移动 HEAD 指针。

# 团队内协作和跨团队协作

##  团队内协作

![img](https://image.imxyu.cn/file/c397bde00d728c4e41eca79f578d25c3.png)

## 跨团队协作

![img](https://image.imxyu.cn/file/e3069f865cc2d9760801b7a06c9d213b.png)

团队内协作就不说了，这里说一下这个跨团队协作。比如现在令狐冲和岳不群是一个团队，他们已经把 代码放到了远程库，但是他们觉得代码可能不够好，就让另一个团队的（东方不败）帮忙看看，但是东方不败肯定不会加入岳不群的团队。此时东方不败就clone 下来，然后在本地修改了一下推送到了自己的远程库，然后发送了一个pull request 请求给岳不群，说你可以拉了。 岳不群审核了一下觉得没问题就merge到了远程库， 然后令狐冲和岳不群通过pull 拉取远程库的代码就能在本地继续操作了

# Github

## 创建远程库

登陆后，点击在网页右上角的“+” --> “New repository”，创建远程库。

![img](https://image.imxyu.cn/file/3dfd6ad9419bfcef2635e08a5c02e86c.png)

![img](https://image.imxyu.cn/file/fba2c7ad888d2aeeea3a6f4a28c7fb03.png)

## 远程仓库操作

| 命令名称           | 作用 |
| ------------------ | ---- |
| git remote -v      | 查看当前所有远程地址别名 |
| git remote add  别名 远程地址 | 起别名 |
| git push 别名 分支 | 推送本地分支上的内容到远程仓库 |
| git clone 远程地址 | 将远程仓库的内容克隆到本地 |
| git pull 远程库地址别名 远程分支名 | 将远程仓库对于分支最新内容拉下来后与 当前本地分支直接合并 |

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git remote -v

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git remote add hellogit https://github.com/abc/HelloGit.git

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git remote -v
hellogit        https://github.com/abc/HelloGit.git (fetch)
hellogit        https://github.com/abc/HelloGit.git (push)

```

添加完别名后 查看会有两个别名，这里的意思是一个是通过拉取的别名，一个通过推送的别名

## 创建别名

**基本语法**：

- `git remote -v` 查看当前所有远程地址别名
- `git remote add 别名 远程地址`

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git remote -v

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git remote add hellogit https://github.com/abc/HelloGit.git

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git remote -v
hellogit        https://github.com/abc/HelloGit.git (fetch)
hellogit        https://github.com/abc/HelloGit.git (push)


```

https://github.com/abc/HelloGit.git，这个地址在创建完远程仓库后生成的连接 ，如图所示

![img](https://image.imxyu.cn/file/d9ee95e5b293a176efbfe8afefc3e632.png)

## GitHub推送本地库到远程库

基本语法：`git push 别名 分支`

> 这里是把`本地的分支`推送到别名这个仓库地址中

```cmd
# 将master分支推送到别名为hellogit远程地址，
# 也就推送到https://github.com/abc/HelloGit.git（具体看前一节）
# 这里需要授权认证操作（输入账号密码）
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git push hellogit master
fatal: unable to access 'https://github.com/abc/HelloGit.git/': OpenSSL SSL_read: Connection was reset, errno 10054

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git push hellogit master
Enumerating objects: 13, done.
Counting objects: 100% (13/13), done.
Delta compression using up to 8 threads
Compressing objects: 100% (9/9), done.
Writing objects: 100% (13/13), 1.02 KiB | 523.00 KiB/s, done.
Total 13 (delta 4), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (4/4), done.
To https://github.com/abc/HelloGit.git
 * [new branch]      master -> master


```

推送成功后，刷新https://github.com/abc/HelloGit：

![img](https://image.imxyu.cn/file/84e90e7cd9e5cda112179e28facb2c2e.png)

## GitHub_拉取远程库到本地库

基本语法：`git pull 别名 分支`

> 这里是将远程端（别名）的分支，拉取到本地

在Github上修改hello.txt文件，并提交。

![img](https://image.imxyu.cn/file/bfe5dcc2d0362f94437bf09b16986ee5.png)

```cmd
#拉取hellogit 的master分支 ，这里注意是拉取远端的`分支` 名， 而不是把远端的拉取到本地的`分支上`。这里需要注意这个分支的意思
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git pull hellogit master
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), 672 bytes | 168.00 KiB/s, done.
From https://github.com/JallenKwong/HelloGit
 * branch            master     -> FETCH_HEAD
   785ab46..47e257f  master     -> hellogit/master
Updating 785ab46..47e257f
Fast-forward
 hello.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

# 可看到从Github上修改后痕迹
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ cat hello.txt
hello, git!hot-fix
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!modify from Github editor
hello, git!hot-fix test
hello, git!master test

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git status
On branch master
nothing to commit, working tree clean

#查看一下历史记录
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ git reflog
47e257f (HEAD -> master, hellogit/master) HEAD@{0}: pull hellogit master: Fast-forward
785ab46 HEAD@{1}: commit (merge): conflict solved
fb0e30b HEAD@{2}: checkout: moving from hot-fix to master
47d2d8f (hot-fix) HEAD@{3}: commit: hot-fix test
25f62d6 HEAD@{4}: checkout: moving from master to hot-fix
fb0e30b HEAD@{5}: commit: master test
25f62d6 HEAD@{6}: checkout: moving from hot-fix to master
25f62d6 HEAD@{7}: checkout: moving from master to hot-fix
25f62d6 HEAD@{8}: checkout: moving from hot-fix to master
25f62d6 HEAD@{9}: checkout: moving from master to hot-fix
25f62d6 HEAD@{10}: merge hot-fix: Fast-forward
b0006bc HEAD@{11}: checkout: moving from hot-fix to master
25f62d6 HEAD@{12}: commit: hot-fix first commit
b0006bc HEAD@{13}: checkout: moving from master to hot-fix
b0006bc HEAD@{14}: reset: moving to b0006bc
5f8dbf6 HEAD@{15}: commit: forth commit
b0006bc HEAD@{16}: reset: moving to b0006bc
41f776b HEAD@{17}: commit: third commit
6967bf0 HEAD@{18}: commit: second commit
b0006bc HEAD@{19}: commit (initial): first commit

```

## GitHub_克隆远程库到本地

基本语法：`git clone 远程地址`

在远程库获取地址URL

![img](https://image.imxyu.cn/file/ff23330d5eab8f2a681c6f91727a37ca.png)

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit (master)
$ cd ..

# 创建一个新文件夹
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop
$ mkdir HelloGit-clone

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop
$ cd HelloGit-clone/

# 在新的文件夹内，克隆远程库到本地
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit-clone
$ git clone https://github.com/abc/HelloGit.git
Cloning into 'HelloGit'...
remote: Enumerating objects: 16, done.
remote: Counting objects: 100% (16/16), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 16 (delta 5), reused 12 (delta 4), pack-reused 0
Receiving objects: 100% (16/16), done.
Resolving deltas: 100% (5/5), done.
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit-clone

$ git remote -v
fatal: not a git repository (or any of the parent directories): .git

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit-clone
$ ls
HelloGit/

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit-clone
$ cd HelloGit/

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit-clone/HelloGit (master)
$ git remote -v
origin  https://github.com/abc/HelloGit.git (fetch)
origin  https://github.com/abc/HelloGit.git (push)

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit-clone/HelloGit (master)
$ git reflog
47e257f (HEAD -> master, origin/master, origin/HEAD) HEAD@{0}: clone: from https://github.com/JallenKwong/HelloGit.git

```

clone 会做如下操作：

1. 拉取代码。
2. 初始化本地仓库。
3. 创建别名( 默认会创建一个别名叫origin)

## GitHub_团队内协作

一、选择邀请合作者。（在仓库设置里操作）

![img](https://image.imxyu.cn/file/945f1ba6e29fb725ee0d852ff59c3851.png)

二、填入目标合作者。

![img](https://image.imxyu.cn/file/ee1b6a6656efe2adbb740b38954529b9.png)

三、复制网址发送给你目标合作者 ， 复制内容如下：https://github.com/atguiguyueyue/git-shTest/invitations。

![img](https://image.imxyu.cn/file/0f5ec0155e64421d315594aa537fd187.png)

四、目标合作者接收到网址，用浏览器打开它，点击接受邀请。

![img](https://image.imxyu.cn/file/295a2398e0a3150d50530d5db21103aa.png)

五、接受邀请成功之后，可以在目标合作者Github账号上看到将来共同开发远程仓库。

![img](https://image.imxyu.cn/file/ddca26e11277016e10eb610195567dac.png)

六、目标合作者可以修改内容并 push 到远程仓库。

```cmd
# 编辑 clone 下来的文件
Layne@LAPTOP-Layne MINGW64 /d/Git-Space/pro-linghuchong/git-shTest(master)
$ vim hello.txt
Layne@LAPTOP-Layne MINGW64 /d/Git-Space/pro-linghuchong/git-shTest(master)
$ cat hello.txt
hello git! hello atguigu! 2222222222222
hello git! hello atguigu! 33333333333333
hello git! hello atguigu!
hello git! hello atguigu!
hello git! hello atguigu! 我是最帅的， 比岳不群还帅
hello git! hello atguigu!
hello git! hello atguigu!
hello git! hello atguigu!
hello git! hello atguigu!
hello git! hello atguigu!
hello git! hello atguigu!
hello git! hello atguigu!
hello git! hello atguigu!
hello git! hello atguigu!
hello git! hello atguigu! master test
hello git! hello atguigu! hot-fix test

# 将编辑好的文件添加到暂存区
Layne@LAPTOP-Layne MINGW64 /d/Git-Space/pro-linghuchong/git-shTest(master)
$ git add hello.txt

# 将暂存区的文件上传到本地库
Layne@LAPTOP-Layne MINGW64 /d/Git-Space/pro-linghuchong/git-shTest(master)
$ git commit -m "lhc commit" hello.txt
[master 5dabe6b] lhc commit
1 file changed, 1 insertion(+), 1 deletion(-)

# 将本地库的内容 push 到远程仓库
Layne@LAPTOP-Layne MINGW64 /d/Git-Space/pro-linghuchong/git-shTest
(master)
$ git push origin master
Logon failed, use ctrl+c to cancel basic credential prompt.
Username for 'https://github.com': atguigulinghuchong
Counting objects: 3, done.
Delta compression using up to 12 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 309 bytes | 309.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/atguiguyueyue/git-shTest.git
7cb4d02..5dabe6b master -> master

```

七、回到发送合作邀请者的 GitHub 远程仓库中可以看到，最后一次是目标合作者提交的。

![img](https://image.imxyu.cn/file/b1a36bdb929e5dcb8cc03744e8806221.png)

## GitHub_跨团队协作

![跨团队协作](https://image.imxyu.cn/file/e3069f865cc2d9760801b7a06c9d213b.png)

一、将远程仓库的地址复制发给邀请跨团队协作的人，比如东方不败。

二、在东方不败的 GitHub 账号里的地址栏复制收到的链接，然后点击 网页右上方的Fork按钮，将项目叉到自己的本地仓库。

![img](https://image.imxyu.cn/file/4856e4845a7f0dbb54c79bd804892f5e.png)

![img](https://image.imxyu.cn/file/c641d9ba65f20ba58d3f98ec792ae0e5.png)

叉成功后可以看到当前仓库信息。

![img](https://image.imxyu.cn/file/3078d75badb2fd393dbe172327dc094c.png)

三、东方不败就可以在线编辑叉取过来的文件。

![img](https://image.imxyu.cn/file/40aa522895eb04a6c203a9bcbca25005.png)

![img](https://image.imxyu.cn/file/eb19249416069d158e2b4280a679063f.png)

四、编辑完毕后，填写描述信息并点击左下角绿色按钮提交。

![img](https://image.imxyu.cn/file/c87379a8a91eb65e2961475129362da4.png)

五、接下来点击上方的 Pull 请求，并创建一个新的请求。

![img](https://image.imxyu.cn/file/8bdb52dc24df07d8d846a4fe19985908.png)

![img](https://image.imxyu.cn/file/9c2f07c7ba5586e3923ba870a37c856d.png)

![img](https://image.imxyu.cn/file/996007e8e9fee91ef37af6818e164139.png)

六、回到岳岳 GitHub 账号可以看到有一个 Pull request 请求。

![img](https://image.imxyu.cn/file/c634b139396001cb2fcb64b8e2a078e1.png)

![img](https://image.imxyu.cn/file/d666e1f3544c07821e85b602d0beffc5.png)

进入到聊天室，可以讨论代码相关内容。

![img](https://image.imxyu.cn/file/3d97452ea50fffca42ec29308c842692.png)

![img](https://image.imxyu.cn/file/9c477d95ea98448b966264bdae235b64.png)

七、如果代码没有问题，可以点击 Merge pull reque 合并代码。

![img](https://image.imxyu.cn/file/894bdb75678d7793e92f1099e5c1d080.png)

![img](https://image.imxyu.cn/file/2964db9c2239859ee59c4bfb9fb25513.png)

## GitHub_SSH免密登录

我们可以看到远程仓库中还有一个 SSH 的地址，因此我们也可以使用 SSH 进行访问。

![img](https://image.imxyu.cn/file/197d6964ccfb06f1eaf22f795061826d.png)

这里如果我们github上还没有配置SSH免密登录的话会有提醒说不能用

接下来我们在本地生成一个SSH公钥，配置到github上

先到用户的主页目录，删除.ssh文件夹（如果没有.ssh文件夹，忽略此步）：

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~
$ cd ~

abc@DESKTOP-R85C9HV MINGW64 ~
$ pwd
/c/Users/abc

abc@DESKTOP-R85C9HV MINGW64 ~
$ ls -a .ssh
./  ../  id_rsa  id_rsa.pub

abc@DESKTOP-R85C9HV MINGW64 ~
$ rm -rf .ssh

```

运行命令ssh-keygen生成.ssh目录：

ssh-keygen -t rsa -C abc@123.com  

> 这里后面的-C 参数是将什么进行加密，这里可以使用自己的一个账号

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit-clone/HelloGit (master)
$ ssh-keygen -t rsa -C abc@123.com
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/abc/.ssh/id_rsa):
Created directory '/c/Users/abc/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/abc/.ssh/id_rsa
Your public key has been saved in /c/Users/abc/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:aeNMB/hP2yiH/Dka2jK9BJciSgA8yKKLlKXX8oei7J0 jallenkwong@163.com
The key's randomart image is:
+---[RSA 3072]----+
|=                |
|++ .   .         |
|+ = . . .        |
|.= o . . +       |
|o.o + + S o      |
|o. o + @ * +     |
|. o . ..O = .    |
| o. . o+.=..     |
|.. E  .o+oo.     |
+----[SHA256]-----+

abc@DESKTOP-R85C9HV MINGW64 ~
$ ls -a .ssh
./  ../  id_rsa  id_rsa.pub

# 生成公钥
abc@DESKTOP-R85C9HV MINGW64 ~
$ cat .ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQChXy8I20br9nu4GCNeZSDkozfHvlRFpXiImYnVlHVvyvFgjct1/zMeJgot1J6+yArSJbA4TMlS9nG8owCE6C9yqhPceDlKtQbARKS2pW7IyP5OhIbcqVmWmvvd+IMmsWrWgK9S6jqp0xSqv3Z3mlcHWOAK18oOe6wF6b3SyGgCP/EcwwUGX4NG7jukhK+In9joSuAxchEg/Ba2/LVjqtfBn3hXZx/SEt+rJ0UVPIT/eEe32HflrzokNcO7l0IgyLntv5QEAsSC2hiGxrM6vF5tQpb12MVZnt1/01ytP0ruQn2TVTI96vsOAa3Cj98dAH2Z0JdqZUSVBw+o3AqXP5oeF1JWkDHZzHQjLgu741wnUZn+vVXFBu1xQyApbvH7y7cNbq8PaxU+SyZbVXbq3RwTywJsyFQvsIOM5l0tG7jUD0QAd6dP3rcNODjFTaafJaBsR9aMwvKQd/d7H+BdwFPYOFp8HB2JAzhRpvlS4Av9MCIe0474wZ0T2QOJmcs7mns= abc@123.com

```

然后，将生成的公钥添加至Github账号SSH设置

![img](https://image.imxyu.cn/file/2d213036d44d57f07ad75b23d20871ea.png)

![img](https://image.imxyu.cn/file/0a6a75ce73adad73a535947dce7fa525.png)

![img](https://image.imxyu.cn/file/0c2f4dd9ef30bdc8c47ae59e50b8851b.png)

![img](https://image.imxyu.cn/file/1f54c4dccd3d8a17e909042c28181fb6.png)

![img](https://image.imxyu.cn/file/0ad893fd8447ac90ed0ee7ceafdf582e.png)

添加公钥后，可不用输入Github账号密码便可推送。

接下来通过SSH方式提交hello.txt。

![img](https://image.imxyu.cn/file/b2391f3377179188499d33c3ee517d53.png)

```cmd
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit-clone/HelloGit (master)
$ cat hello.txt
hello, git!hot-fix
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!modify from Github editor
hello, git!hot-fix test
hello, git!master test

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit-clone/HelloGit (master)
$ vim hello.txt

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit-clone/HelloGit (master)
$ cat hello.txt
hello, git!hot-fix
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!ssh test
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!
hello, git!modify from Github editor
hello, git!hot-fix test
hello, git!master test

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit-clone/HelloGit (master)
$ git add .

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit-clone/HelloGit (master)
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   hello.txt

abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit-clone/HelloGit (master)
$ git commit -m "ssh test"
[master 9602a37] ssh test
 1 file changed, 1 insertion(+), 1 deletion(-)

# 通过SSH推送
abc@DESKTOP-R85C9HV MINGW64 ~/Desktop/HelloGit-clone/HelloGit (master)
$ git push git@github.com:abc/HelloGit.git master
The authenticity of host 'github.com (13.250.177.223)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com,13.250.177.223' (RSA) to the list of known hosts.
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 283 bytes | 283.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To github.com:JallenKwong/HelloGit.git
   47e257f..9602a37  master -> master


```

推送成功

![img](https://image.imxyu.cn/file/92d174f42df3162c15b5d2824dc06e28.png)

# IDEA集成Git_环境准备

实操的IDEA版本为ultimate 2020.1。

## 配置 Git 忽略文件

与项目的实际功能无关，不参与服务器上部署运行。把它们忽略掉能够屏蔽 IDE 工具之间的差异。例如，Maven工程根据src生成的target。

创建忽略规则文件 xxxx.ignore（前缀名随便起，建议是 git.ignore），**这个文件的存放位置原则上在哪里都可以**，为了便于让~/.gitconfig 文件引用，建议也放在用户家目录下。

git.ignore 文件模版内容如下：

```txt
# Compiled class file
*.class

# Log file
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*
.classpath
.project
.settings
target
.idea
*.iml

```

在.gitconfig 文件中引用忽略配置文件（此文件在 Windows 的家目录中）

```txt
[user]
    name = Layne
    email = Layne@atguigu.com
[core]
	excludesfile = C:/Users/asus/git.ignore

```

注意：这里要使用“正斜线（/）”，不要使用“反斜线（\）”

## 在IDEA配置Git程序

在菜单栏File->Setting->搜索栏搜Git，配置Git的安装路径。

![img](https://image.imxyu.cn/file/d86ae18c01b9a08bc73f02aa6c1b3708.png)

## IDEA集成Git_初始化&添加&提交

先创建一个名叫HelloGit的Maven工程。

### 初始化Git

在菜单栏VCS -> Import into Version Control -> Create Git Repository

![img](https://image.imxyu.cn/file/4bbfb1e76fb25655b3fe6900bb29ea47.png)

选择要创建 Git 本地仓库的工程，也就是HelloGit工程，然后添加OK。

![img](https://image.imxyu.cn/file/5dc609978787f3e5a83dbdf954a3e039.png)

### 添加到暂存区

创建一个HelloGit类，将其添加Git暂存区。

右键点击HelloGit类，选择Git->Add。可以右键点击HelloGit，更大范围地添加文件到暂存区。

![img](https://image.imxyu.cn/file/42d316f8c5058311afc83779b0c13551.png)

添加成功后，文件名会从红色变成绿色。

![img](https://image.imxyu.cn/file/ec95c2b0b99d1f3a0ac8ad3291fdde50.png)

### 提交至本地库

右键点击HelloGit，选择Git->Commit Directory。

![img](https://image.imxyu.cn/file/9a73d7bbac024bfe61951662f5bf6ded.png)

添加注释后提交：

![img](https://image.imxyu.cn/file/281c7f26319b59e7584103c2a3ee88dd.png)

添加成功后，后台打印相关信息。

![img](https://image.imxyu.cn/file/3bdf7a9135d3ae80b9aad4d678626491.png)

## IDEA集成Git_切换版本

在 IDEA 的左下角，点击 Git，然后点击 Log 查看版本

![img](https://image.imxyu.cn/file/69e6670ea5681781c173f1c86864ae1e.png)

右键选择要切换的版本，然后在菜单里点击 Checkout Revision。

![img](https://image.imxyu.cn/file/5530ac3d829954cebd23ed15a681769f.png)

## IDEA集成Git_创建分支&切换分支

### 创建分支

右键点击HelloGit，Git -> Repository -> Branches，或者点击IDEA的右下角，如图红圈所示部位：

![img](https://image.imxyu.cn/file/3e31e84cd2f7b5b95bb2639abcb1804f.png)

![img](https://image.imxyu.cn/file/c7544c1bade118b3177907ad903a8082.png)

选择点击New Branch：

![img](https://image.imxyu.cn/file/b9c18ec9924788adfa432b7b924308ce.png)

创建新分支：

![img](https://image.imxyu.cn/file/8a4a5e7cf7511d086ddac0be704e850f.png)

### 切换分支

跟**创建分支**步骤相似，如点击IDEA的右下角（它显示项目正处在那条分支），如图红圈所示部位，选择你想要切换的分支，然后checkout：

![img](https://image.imxyu.cn/file/c7544c1bade118b3177907ad903a8082.png)

![img](https://image.imxyu.cn/file/afbbe9a835629f522d0b02024fe2c11b.png)

或者在log窗口，右键点击分支，选择checkout：

![img](https://image.imxyu.cn/file/7ab6ab48e9b5a42009757b8b17b901f0.png)

## IDEA集成Git_合并分支(正常合并)

先在hot-fix分支修改HelloGit类，并将其提交：

![img](https://image.imxyu.cn/file/41667203b7e067b59d1310cce4d92b15.png)

然后切换到master分支，右下角的hot-fix会变为master：

![img](https://image.imxyu.cn/file/f183b86164b0e00e9d6e8c8c9a4a17da.png)

然后，点击IDEA 窗口的右下角的master，将 hot-fix 分支合并到当前 master 分支。选择hot-fix->Merge into Current

![img](https://image.imxyu.cn/file/fff1d4e014223aa2cc70f0fdc237f350.png)

如果代码没有冲突， 分支直接合并成功，分支合并成功以后，代码自动提交，无需手动
提交本地库。

![img](https://image.imxyu.cn/file/271421c28750e86e69accd6ac687490c.png)

## IDEA集成Git_合并分支(冲突合并)

分别在master，hot-fix分支修改HelloGit类同一行，并提交，故意制作冲突：

![img](https://image.imxyu.cn/file/53daad680bc796069dc1ce61682d4abc.png)

![img](https://image.imxyu.cn/file/f8fcf275169cdec742d31ce85ce20d7f.png)

切换到master分支，将hot-fix的合并到master分支：

![img](https://image.imxyu.cn/file/fff1d4e014223aa2cc70f0fdc237f350.png)

冲突产生，需要人工解决：

![img](https://image.imxyu.cn/file/eb3804e00dccfa2658aa33c972d8996e.png)

其中左边的意思就是master分支上的，中间是最后的result， 右边是hot-fix 分支的

![img](https://image.imxyu.cn/file/a73d642f285f171eeff7ae0c9bb6111b.png)

![img](https://img-blog.csdnimg.cn/img_convert/eafebea94008b4f1f0ae15f8b2092919.png)

代码冲突解决，将代码提交本地库后，如图所示：

![img](https://image.imxyu.cn/file/15a9058f7f35112b8605cd69aaf42e35.png)

## IDEA集成GitHub_设置GitHub账号

在菜单栏File->Setting->搜索栏搜GitHub，添加GitHub账号：

![img](https://image.imxyu.cn/file/00572764f1ed257dbc2d0f668434e6a0.png)

由于网络问题，会时常登陆不了：

![img](https://image.imxyu.cn/file/3084475acd640b1adf621688462a1504.png)

解决方法：可通过Token登陆。

![img](https://image.imxyu.cn/file/f3a64a809fa56624a997073836139140.png)

登陆Github网站，**获取Token**，操作步骤看下图：

![img](https://image.imxyu.cn/file/2d213036d44d57f07ad75b23d20871ea.png)

![img](https://image.imxyu.cn/file/9b2489e068b004bee03a227760248edb.png)

![img](https://image.imxyu.cn/file/200c556f6f0b6d44c442b58d6e8bb7ea.png)

![img](https://image.imxyu.cn/file/81a82fb47421c0a802cd1cfad7297e43.png)

![img](https://image.imxyu.cn/file/3f596f2f68d50d277eefe1a4e6035d2d.png)

将生成的token用来IDEA登录。

![img](https://image.imxyu.cn/file/704eafd9157658a0be35b081c3530ced.png)

## IDEA集成GitHub_分享项目到GitHub

![img](https://image.imxyu.cn/file/e057d2e660c2033ef9eae0c638aee2bc.png)

![img](https://image.imxyu.cn/file/a5ba3e09a890113b94420c3939dac239.png)

![img](https://image.imxyu.cn/file/44b2419dd2eebc2c053fb642188e8909.png)

## IDEA集成GitHub_推送代码到远程库

![img](https://image.imxyu.cn/file/84baeaa175c6faa3ff538e0313187eff.png)

![img](https://image.imxyu.cn/file/1ee51e1ed781404a93655f1ad10bd9ca.png)

![img](https://image.imxyu.cn/file/bb9689e1fc2e1167e68ef39f73f28af7.png)

注意： push 是将本地库代码推送到远程库，如果本地库代码跟远程库代码版本不一致，
push 的操作是会被拒绝的。也就是说， 要想 push 成功，一定要保证本地库的版本要比远程库的版本高！ 因此一个成熟的程序员在动手改本地代码之前，一定会先检查下远程库跟本地代码的区别！如果本地的代码版本已经落后，切记要先 pull 拉取一下远程库的代码，将本地代码更新到最新以后，然后再修改，提交，推送！

## IDEA集成GitHub_拉取远程库代码合并本地库

右键点击项目，可以将远程仓库的内容 pull 到本地仓库。


![img](https://image.imxyu.cn/file/193d00830bc1636e2ae2640b749b9899.png)

注意： pull 是拉取远端仓库代码到本地，如果远程库代码和本地库代码不一致，会自动
合并，如果自动合并失败，还会涉及到手动解决冲突的问题。

## IDEA集成GitHub_克隆代码到本地

在菜单栏的File->Close Project->Get from Version Control。

![img](https://image.imxyu.cn/file/f436b8f9156e5e5ce8c0e2b3b5fe3639.png)

或者在菜单栏VCS->Get from Version Control。

![img](https://image.imxyu.cn/file/63f9691dd0a7627bd44a125942be7f31.png)

# Gitee

38_码云_账号注册登录&创建远程库
码云简介
众所周知， GitHub 服务器在国外， 使用 GitHub 作为项目托管网站，如果网速不好的话，严重影响使用体验，甚至会出现登录不上的情况。针对这个情况， 大家也可以使用国内的项目托管网站-码云。

码云是开源中国推出的基于 Git 的代码托管服务中心， 网址是 https://gitee.com/ ，使用
方式跟 GitHub 一样，而且它还是一个中文网站，如果你英文不是很好，它是最好的选择。

账号注册登录
略

创建远程库
跟Github的类似。

![img](https://image.imxyu.cn/file/c8c01b7813e8578423ee2d790a580ee4.png)

![img](https://image.imxyu.cn/file/16341a72fa521e904f7b5a5489d4c693.png)

另外，可以从GitHub与GitLab中导入仓库。

![img](https://image.imxyu.cn/file/c5daf945afdd2f2836c1e9e20e8e389b.png)

## IDEA集成Gitee码云

首先，要在IDEA安装Gitee插件。

在菜单栏选File->Settings->Plugins，搜Gite

![img](https://image.imxyu.cn/file/0241b8536ebed6fb08dcf04804c62cb0.png)

安装插件成功后，重启IDEA。

功能跟在IDEA的Github插件，功能类似，如添加Gitee账号等，可参考前文IDEA的Github插件，触类旁通。

![img](https://image.imxyu.cn/file/87e35b8ee0fd1fd4136f2b2727cbf02a.png)

![img](https://image.imxyu.cn/file/89e7174170b4ecc1c4ba896e3f1c9ad9.png)

# GitLab

## GitLab_简介和安装环境准备

### GitLab简介

GitLab 是由 GitLab Inc.开发，使用 MIT 许可证的基于网络的 Git 仓库管理工具，且具有wiki 和 issue 跟踪功能。使用 Git 作为代码管理工具，并在此基础上搭建起来的 web 服务。（可搭建局域网Git仓库）。

GitLab 由乌克兰程序员 DmitriyZaporozhets 和 ValerySizov 开发，它使用 Ruby 语言写成。后来，一些部分用 Go 语言重写。截止 2018 年 5 月，该公司约有 290 名团队成员，以及 2000 多名开源贡献者。 GitLab 被 IBM， Sony， JülichResearchCenter， NASA， Alibaba，Invincea， O’ReillyMedia， Leibniz-Rechenzentrum(LRZ)， CERN， SpaceX 等组织使用。

### GitLab官网地址

[官网地址](https://about.gitlab.com/)

[安装说明](https://about.gitlab.com/installation/)

## GitLab安装准备

1. 准备一个系统为 CentOS7 以上版本的服务器， 要求内存 4G，磁盘 50G。

2. 关闭防火墙， 并且配置好主机名和 IP，保证服务器可以上网。
3. 此教程使用虚拟机：主机名： gitlab-server IP 地址： 192.168.6.200
4. Yum 在线安装 gitlab- ce 时，需要下载几百 M 的安装文件，非常耗时，所以最好提前把所需 RPM 包下载到本地，然后使用离线 rpm 的方式安装。下载地址。注：资料里提供了此 rpm 包，直接将此包上传到服务器/opt/module 目录下即可。

## GitLab_安装&初始化服务&启动服务

### 编写安装脚本

安装 gitlab 步骤比较繁琐，因此我们可以参考官网给的命令自己编写一个脚本

```cmd

[root@gitlab-server module]# vim gitlab-install.sh
sudo rpm -ivh /opt/module/gitlab-ce-13.10.2-ce.0.el7.x86_64.rpm

sudo yum install -y curl policycoreutils-python openssh-server cronie

sudo lokkit -s http -s ssh

sudo yum install -y postfix

sudo service postfix start

sudo chkconfig postfix on

curl https://packages.gitlab.com/install/repositories/gitlab/gitlabce/script.rpm.sh | sudo bash

sudo EXTERNAL_URL="http://gitlab.example.com" yum -y install gitlabce

```

给脚本增加执行权限

```cmd

[root@gitlab-server module]# chmod +x gitlab-install.sh
[root@gitlab-server module]# ll
总用量 403104
-rw-r--r--. 1 root root 412774002 4 月 7 15:47 gitlab-ce-13.10.2-
ce.0.el7.x86_64.rpm
-rwxr-xr-x. 1 root root 416 4 月 7 15:49 gitlab-install.sh


```

然后执行该脚本，开始安装 gitlab-ce。注意一定要保证服务器可以上网。

```cmd

[root@gitlab-server module]# ./gitlab-install.sh
警告： /opt/module/gitlab-ce-13.10.2-ce.0.el7.x86_64.rpm: 头 V4
RSA/SHA1 Signature, 密钥 ID f27eab47: NOKEY
准备中... #################################
[100%]
正在升级/安装...
1:gitlab-ce-13.10.2-ce.0.el7
################################# [100%]
。 。 。 。 。 。


```

### 初始化GitLab服务

执行以下命令初始化 GitLab 服务，过程大概需要几分钟，耐心等待…

```cmd

[root@gitlab-server module]# gitlab-ctl reconfigure
。 。 。 。 。 。
Running handlers:
Running handlers complete
Chef Client finished, 425/608 resources updated in 03 minutes 08
seconds
gitlab Reconfigured!


```

### 启动GitLab服务

执行以下命令启动 GitLab 服务，如需停止，执行 gitlab-ctl stop

```cmd

[root@gitlab-server module]# gitlab-ctl start
ok: run: alertmanager: (pid 6812) 134s
ok: run: gitaly: (pid 6740) 135s
ok: run: gitlab-monitor: (pid 6765) 135s
ok: run: gitlab-workhorse: (pid 6722) 136s
ok: run: logrotate: (pid 5994) 197s
ok: run: nginx: (pid 5930) 203s
ok: run: node-exporter: (pid 6234) 185s
ok: run: postgres-exporter: (pid 6834) 133s
ok: run: postgresql: (pid 5456) 257s
ok: run: prometheus: (pid 6777) 134s
ok: run: redis: (pid 5327) 263s
ok: run: redis-exporter: (pid 6391) 173s
ok: run: sidekiq: (pid 5797) 215s
ok: run: unicorn: (pid 5728) 221s


```

## GitLab_登录GitLab并创建远程库

### 登录GitLab

使用主机名或者 IP 地址即可访问 GitLab 服务。可配一下 windows 的 hosts 文件（C:\Windows\System32\drivers\etc）

![img](https://image.imxyu.cn/file/048956890f1e644fb77e8d58092a8b6d.png)

![img](https://image.imxyu.cn/file/1870c658dcabcf08fbc3b5b9fa6b2243.png)

首次登陆之前，需要修改下 GitLab 提供的 root 账户的密码，要求 8 位以上，包含大小写子母和特殊符号。

然后使用修改后的密码登录 GitLab。

![img](https://image.imxyu.cn/file/5608650ec5e913d5ab549f30fbb477d3.png)

GitLab 登录成功。

![img](https://image.imxyu.cn/file/3231f6dd1a07f90326ec0506eaae747f.png)

### 创建远程库

![img](https://image.imxyu.cn/file/2ab639dfaa57cd499133c2c4cde1222a.png)

![img](https://image.imxyu.cn/file/f135a7b76c745c3aeef9034a82c8afaf.png)

![img](https://image.imxyu.cn/file/5ff82d173fe047244945c8cd255a4b33.png)

## GitLab_IDEA集成GitLab

![img](https://image.imxyu.cn/file/1f34175126922c56c158f466dd4d665c.png)

接下来插件配置，Git操作等与Github、Gitee的IDEA插件大同小异，不再赘述，自己触类旁通吧！

