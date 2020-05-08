---
title: simple study the principle of git
date: 2020-03-28 16:26:12
tags: git
---

git 作为程序员经常使用的分布式版本控制工具，知其然也应该知其所以然，所以我们来浅谈下git的原理。

*申明：本文不是原创，更多的记录下 git 的知识，文章尾部有参考文章链接。*

<!--more-->

我们经常使用的命令有如下几个：

`git pull`

`git clone`

`git add` 

`git commit` 

`git push`

`...`

那我们就选择几个关键且常用的命令来探究下内部的原理。

git 的基本对象有三个，blob 对象，tree 对象和 commit 对象，这个可以了解下。

git 的工作空间，暂存区和git仓库等概念可以自行了解下，如图所示。

![git_workspace_show](git_workspace_show.JPG)

这是一个使用 git 管理的项目目录，你可以认为，其中 .git 目录就是本地 git 仓库。而 .git 目录下的index 就是工作暂存区。目录中除 .git 以外的空间就是用户工作目录，是用户在操作系统的文件目录下可以实实在在操作的文件集。

假设工作空间里有我们创建编辑好的各个文件，当我们执行 `git add .`命令后，git 本地仓库中会多出来一些 Blob 对象。

![blob object](p1s1.png)

这些 blob 对象的内容是对用户空间里文件内容进行二进制加密压缩，而 blob 对象的名字就是文件内容的 sha1 算法得到大的 hash 值（这个也是每个 blob 对象唯一的ID）。

接着，我们执行 `git commit -m 'test'`命令。

![tree object](p1s2.png)

执行完 `git commit`命令后，首先会在 git 仓库里创建一个对应当前 blob 对象的 目录结构的 tree  对象。tree 对象的内容是一个目录索引，存放的是指向的各个 blob 对象的 ID ，还有 blob 对象对应的工作空间文件的名字和权限等信息。tree 对象的 ID 就是 tree 对象的内容的 hash 值。

然后，git 还会在 git 仓库里创建一个 commit 对象。

![commit object](p1s3.png)

这个 commit 对象可以看作是当前项目的一个快照（snapshot），每个 commit 对象里都对应着一个版本，实际上，git 就是通过操作 "commit 链" 来进行版本的切换的。commit 对象里存放的是 tree 对象的信息，当前认证的作者的信息，提交者的信息以及提交 commit 的内容等。commit 对象的 ID 是commit 对象内容的 hash 值。

当然，如下图所示，git 对 commit 的操作是通过分支来进行的，所以会有分支来指向 commit 对象，master 默认是主分支。

![head_master_show](p1s4.png)

至此，git 的基本原理就大致略知一二了， git是储存一个文件的内容、目录结构、commit信息和分支的。**其本质上是一个key-value的数据库加上默克尔树形成的有向无环图（DAG）** ，对 git 的版本控制就是在 ”commit 链“上来回移动。

下图展示了 git 操作的完整过程。

![dynamic process of git](p2s3.gif)



非常有意思的是，git 采用默克尔树的结构可以防止被人篡改文件，任何人想要更改其中某个文件而不被其他人发现的话，就必须得篡改 git 仓库中所有的文件，因为所有的文件的 ID 也就是 hash 值都是关联在一起。当然，如果有人对你远端的 git中央仓库 执行了 `git pull`  或者 `git clone` 命令，那他就有了一个完整的未被篡改的 git 仓库，这样就构成了分布式。这样的设计也决定了基本没人可以恶意篡改一个有影响力的 git 仓库，即使未经说明篡改了历史被人发现，那也会遭到别人的鄙视。



git 的原理其实很复杂，指令也相当繁杂，如果想要深入学习 git 的原理，那就得花更多得时间去看书和思考。

大家一致认同的剖析 git 原理的好书，[《Pro Git》]( https://book.douban.com/subject/3420144/ )

本文基本是全部借鉴了 [lazne大佬]( https://www.lzane.com/ )的这篇文章中的图。这篇文章讲的深入浅出，[参考文章传送门]( https://www.lzane.com/tech/git-internal/ )。

还有bilibili讲解的视频：[探究git原理]( https://www.bilibili.com/video/BV1RJ411X7kh )

