---
comments: true
date: 2012-06-05 09:56:35
layout: post
title: 使用git 管理你的代码库
categories:
- Linux
tags:
- git
- 版本控制
---


* TOC
{:toc}
<hr/>
Git 是 Linux Torvalds 为了帮助管理 Linux® 内核开发而开发的一个开放源码的版本控制软件。我们可以自己下载这个软件用于对内核的 hack 分析，或者用来管理自己的软件开发项目。本文将教你如何使用 Git 工具管理自己的程序代码 。

目前比较流行的版本控制系统包括：

* CVS
* Subversion
* Arch
* Bazaar
* BitKeeper

版本控制是程序开发、管理必不可少的工具，特别是在多人协作的团队中，适宜的版本控制工具可以提高开发效率，消除很多有代码版本带来的问题.git就是一个非常好的版本控制工具。

### 安装

在Ubuntu Linux 下可以通过简单的如下命令安装git:
    sudo apt-get install git-core
ArchLinux使用如下命令
    sudo pacman -S git
Fedora, Ret Hat 系列可以使用yum ,其他Linux版本可以下载源代码编译安装

### 使用



使用Git的第一件事就是设置你的名字和email,这些就是你在提交commit时的签名。
    
    $ git config --global user.name "Scott Chacon"
    $ git config --global user.email "schacon@gmail.com"
    

执行了上面的命令后,会在你的主目录(home directory)建立一个叫 ~/.gitconfig 的文件. 内容一般像下面这样:

    [user]
    name = Scott Chacon
    email = schacon@gmail.com

### 创建版本库

#### 初始化

在使用git之前，需要在你的代码所在的目录初始化git的版本库，使之能记录你对代码的修改，初始化的命令很简单
    
    $git init
    

初始化成功会出现类似以下内容

    初始化空的 Git 版本库于  /[yourpath]/.git

### 植入内容跟踪信息

#### 添加文件

初始化git的仓库之后，就要把你需要管理的文件添加到git的版本库里。
    
    $git add [yourfile]
    

### 提交内容到版本库

#### 提交内容修改

现在我们已经添加了文件信息，我们现在可以看看git的状态
    
    $git status
    

返回了类似如下结果

    位于分支 master

    # 初始提交

    ##### 要提交的变更：（使用 "git rm --cached ..." 撤出暂存区）

    ##### 新文件：    [yourfile]

    #

提示信息告诉我们，版本库里添加了新文件，我们需要提交文件修改
    
    $git commit -m "init"
    

命令中双引号所夹内容是一串自定义的字符串，用来标示这次提交，你将来如果要恢复到当前版本时，可以通过这段字符串找到具体的版本。

#### 查询未提交修改

现在我们已经把文件提交到版本控制库里面了,我们稍微修改一下我们的文件。然后我们可以使用如下命令查询当前版本和版本库版本的区别：
    
    $git diff
    

返回类似如下结果

    diff --git a/div.py b/div.py
    index fc374df..c9755c7 100644
    --- a/div.py
    +++ b/div.py
    @@ -17,5 +17,6 @@ while(math.fabs(f(a)-f(b)) < tol):
    if (f(b)*f(c) < 0):
    c = (b + c) / 2
    if (math.fabs(f(a)-f(b)) < tol):
    +
    print(c)

 如果你使用过diff 和patch  你会发现这就是diff -u 输出的格式，补丁文件也是采用这样的格式。
 接下来我们就可以继续提交我们的修改。

#### 删除文件

现在我们已经是使用git来管理我们的代码了，如果我们要删除某个文件，就不能直接rm它了，因为它在版本库里有记录，如果直接删除，版本库发现它不见了，会出现一些错误，正确的删除方式是
    
    $git rm [yourfile]
    

### 管理分支

git一个强大的地方就是他的分支管理

#### 管理分支
    
    $git branch
    

直至现在为止，我们的项目版本库一直都是只有一个分支 master。在 git 版本库中创建分支的成本几乎为零，所以，不必吝啬多创建几个分支。下面列举一些常见的分支策略，仅供大家参考：

* 创建一个属于自己的个人工作分支，以避免对主分支 master 造成太多的干扰，也方便与他人交流协作。

* 当进行高风险的工作时，创建一个试验性的分支，扔掉一个烂摊子总比收拾一个烂摊子好得多。

* 合并别人的工作的时候，最好是创建一个临时的分支，关于如何用临时分支合并别人的工作的技巧，将会在后面讲述。



#### 创建分支

使用如下命令将创建我们自己的工作分支work，并将以后的工作转移到这个分支上。
    
    $git branch work
    $git checkout work
    

更简单和常用的方法是直接通过 checkout 命令来一次性创建并转移到新建分支上，命令如下：
    
    $ git checkout -b work [start_point]
    

其中 start_point 是一个可选参数，指定新建分支 work 是基于哪个节点，默认为当前所在分支的节点。

#### 删除分支

要删除版本库中的某个分支，使用 git branch -d 命令就可以了，例如：
    
    　　$ git branch -d branch-name
    

但是需要注意的是，如果删除的分支还没有被 merge 到其他分支，删除这样的分支会导致这个分支上所做的改动丢失，因此 git branch -d 命令会失败，提示你这样做会丢失信息。如果你的确想删除这样的分支，不怕信息丢失，那么可以使用 git branch -D 命令，这个命令不会去判断分支的merge状态，例如：
    
    　　$ git branch -D branch-name
    

通常建议使用 -d 参数来删除分支，以防无意的信息丢失。

#### 查看分支

运行下面的命令可以得到你当前工作目录的分支列表：
    
    　　$ git branch
    

在你正在工作的分支的名字前面，会有 * 号标示，比如：
    
    　　$ git branch
    　　bugfix
    　　* master
    

说明有两个本地分支 bugfix 和 master， 其中当前的工作分支为 master。

#### 合并两个分支

合并两个分支：git merge
 既然我们为项目创建了不同的分支，那么我们就要经常地将自己或者是别人在一个分支上的工作合并到其他的分支上去。现在我们看看怎么将 robin 分支上的工作合并到 master 分支中。现在转移我们当前的工作分支到 master，并且将 robin 分支上的工作合并进来。

     $ git checkout master
     $ git merge -m "Merge from robin" robin

 上面的命令会将 robin 分支的改动 merge 到 master，并生成一个新的 commit 节点，这个 commit 的注释信息为 "Merge from robin"
 (kwydwuf注: $ git merge "Merge work in robin" HEAD robin 是老版本的用法，应该废弃 ）
 合并两个分支，还有一个更简便的方式，下面的命令和上面的命令是等价的 （kwydwuf注：git pull 的本意是用来 merge 远端版本库中的某个分支，用在此处没有任何简便之处，可以废弃）。

     $ git checkout master
     $ git pull . robin

但是，此时 git 会出现合并冲突提示：

     Trying really trivial in-index merge...
     fatal: Merge requires file-level merging
     Nope.
     Merging HEAD with d2659fcf690ec693c04c82b03202fc5530d50960
     Merging:
     1d2fa05b13b63e39f621d8ee911817df0662d9b7 Some fun
     d2659fcf690ec693c04c82b03202fc5530d50960 some work
     found 1 common ancestor(s):
     3ecebc0cb4894a33208dfa7c7c6fc8b5f9da0eda a new day for git
     Auto-merging hello
     CONFLICT (content): Merge conflict in hello
     Automatic merge failed; fix up by hand

 git 的提示指出，在合并作用于文件 hello 的 'Some fun' 和 'some work' 这两个对象时有冲突，具体通俗点说，就是在 master, robin 这两个分支中的 hello 文件的某些相同的行中的内容不一样。我们需要手动解决这些冲突，现在先让我们看看现在的 hello 文件中的内容。

     $ cat hello

 此时的 hello 文件应是这样的，用过其他的版本控制系统的朋友应该很容易看出这个典型的冲突表示格式：

     Hello World
     It's a new day for git
     <<<<<<< HEAD/hello
     Play, play, play
     =======
     Work, work, work
     >>>>>>> d2659fcf690ec693c04c82b03202fc5530d50960/hello

 我们用编辑器将 hello 文件改为：

     Hello World
     It's a new day for git
     Play, play, play
     Work, work, work

 现在可以将手动解决了冲突的文件提交了。

     $ git commit -i hello

 以上是典型的两路合并（2-way merge）算法，绝大多数情况下已经够用。但是还有更复杂的三路合并和多内容树合并的情况。详情可参看： git help read-tree， git help merge 等文档。

#### 逆转和恢复

逆转与恢复：git reset
 项目跟踪工具的一个重要任务之一，就是使我们能够随时逆转（Undo）和恢复（Redo）某一阶段的工作。
 git reset 命令就是为这样的任务准备的。它将当前的工作分支的 头 定位到以前提交的任何版本中，它有三个重置的算法选项。
 命令形式：

     git reset [--mixed | --soft | --hard] []
     
 命令的选项：

    --mixed

 仅是重置索引的位置，而不改变你的工作树中的任何东西（即，文件中的所有变化都会被保留，也不标记他们为待提交状态），并且提示什么内容还没有被更新了。这个是默认的选项。
 
    --soft

 既不触动索引的位置，也不改变工作树中的任何内容，我们只是要求这些内容成为一份好的内容（之后才成为真正的提交内容）。这个选项使你可以将已经提交的东西重新逆转至"已更新但未提交（Updated but not Check in）"的状态。就像已经执行过 git update-index 命令，但是还没有执行 git commit 命令一样。

     --hard

 将工作树中的内容和头索引都切换至指定的版本位置中，也就是说自  之后的所有的跟踪内容和工作树中的内容都会全部丢失。因此，这个选项要慎用，除非你已经非常确定你的确不想再看到那些东西了。

###### 参考资料

[看日记学git](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/06/diarygit.pdf)

[git使用简介](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/06/use_git.pdf)

[git中文教程](http://andylinux-wordpress.stor.sinaapp.com/uploads/2012/06/git+.pdf)

[百度百科-git](../../“http:/baike.baidu.com/view/1531489.htm")

 [git的主页](http://git-scm.com)
