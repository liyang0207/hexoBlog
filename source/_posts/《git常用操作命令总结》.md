---
title: 《git常用操作命令总结》
date: 2018-09-09 14:37:55
tags:
  - git
  - 随写
---

git工作流现在已经成了大大小小科技型公司的标配，程序员日常撸代码必备工具。不同的公司根据业务的展开及开发的进度都会制定最合适的git流程。上家公司开发人员少，流程基本就是`develop->release->master`，有线上bug修复，就基于`master`拉`hotfix`分支，测完之后通过`gitlab`合并回`master`和`develop`分支，基本上属于单线操作，简单清晰。到新公司后，开发需求数倍增加，开发人员也比较多，各种流程繁杂，需要合并分支，处理冲突，多线开发的情况也越来越多。以往学习的那点git知识已经不能满足现在的开发了（其实就是git学的不精），最近又重新把git流程过了一遍，今天来总结一下。

### 几个基础概念

#### 工作区及文件状态

git分为三个工作区域：`工作区 Working Directory`、`暂存区 Staging Area`、`GIT仓库 repository`，git仓库又有本地和远程的概念，一般来讲，本地仓库会领先于远程仓库，不过也有例外情况，下面再具体讲。

git的文件状态。git仓库中的文件，不外乎这几种状态：

- `Untracked 未追踪`，即新建一个`a.js`文件，还没有被git追踪，不会到版本库内。
- `Unmodified 未修改`，该文件在git版本库内，但是还没有被修改。
- `modified 已修改`，该文件在git版本库，已经修改，但还没有暂存。
- `Staged 已暂存`，有修改的文件已经通过`git add <file>`添加到`暂存区`。

仓库内文件的状态可以通过`git status`查看。

![gitblog01](/images/git常用操作命令总结/gitblog01.png)

`a.js`是新增的文件，还没有被追踪，`c.js`修改了，但是还没有添加到`暂存区`，`b.js`修改后已经添加到`暂存区`，此时`commit`将只会提交`b.js`。

#### 快照流

每次提交保存时，git主要对当时的全部文件制作一个快照并保存这个快照的索引。 为了高效，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。

### 各种撤消

针对不同情况的撤消操作命令，在`git status`中都能看到，这里来总结一下：

- `git rm <file>` ：将某个已被追踪的文件删除，提交之后该文件不会再被追踪，同时本地也会删除该文件。
- `git rm --cached <file>` : 同上，不再追踪某个文件，但是本地磁盘内不会删除该文件，回到`Untracked`状态。
- `git checkout -- <file>` : 将整个文件的修改恢复，慎用，`modified` -> `unmodified`，一般使用`IDE`工具修改最好。
- `git reset HEAD <file>` : 将暂存区的文件回退至已修改状态，`staged` -> `modified`，文件`add`之后回退可用。
- `git reset --hard HEAD^` : 回退到上个版本，`commit`之后算一个版本。
- `git reset --hard commit_id` : 移动HEAD指针，会退到指定commit_id的版本。
- `git commit --amend` : 覆盖之前的提交信息，运行此命令会将暂存区文件提交，同时启动`vim`编辑器，可以编辑上一次的提交信息，最终提交记录里只会有这一次修改后的信息。

### 打标签

Git 可以给历史中的某一个提交打上标签，以示重要。 比较有代表性的是人们会使用这个功能来标记发布结点（v1.0 等等）。工作中可以给某次提交打上标签，运维发布线上代码时会以此为准。

- `git tag` ：列出所有标签。
- `git tag -l release` ：列出所有包含release字段的tag。
- `git tag -a v1.0 -m 'my v1.0 version'` : 创建附注标签，使用`git show v1.0`查看详细信息。
- `git tag v1.0` ：创建轻量标签。
- `git tag v1.0 commit_id` ： 给某个历史提交`commit_id`打标签，可以使用`git log --pretty=online`查看历史请求。 
- `git push origin [tagname] ` :  推送单个标签到远端
- `git push origin --tags` ： 推送所有本地标签到远端
- `git checkout -b [branchname] [tagname]` : 在特定的标签上创建一个新分支

### 分支操作

分支操作比较常用，总结一下命令行：

- `git branch develop` : 新建develop分支，但是不切换。
- `git checkout master` : 切换至master分支。
- `git checkout -b develop`  ：基于当前分支新建develop分支，并切换至develop分支。
- `git branch -d dev` ：删除`dev`分支。
- `git branch -r` ：查看远程分支列表
- `git checkout -b iss53 origin/iss53` : 拉取远端分支iss53并在本地新建该分支。
- `git checkout --track origin/iss53` : 同上，简化版本。
- `git push -u origin iss54` :  推送本地分支iss54至远程仓库origin/iss54，同时建立追踪关系。
- `git push origin --delete iss54` : 删除远程分支iss54

### 合并及处理冲突

#### 合并

先来张合并图：

![gitblog02](/images/git常用操作命令总结/gitblog02.png)

上图展示了将`iss53`分支合并至`master`分支的过程。首先切换到`master`分支，保证`master`分支最新，然后`git merge iss53`，此时Git 会使用两个分支的末端所指的快照（`commit6`和 `commit4-hotfix`）以及这两个分支的工作祖先（`commit3`），做一个简单的三方合并。

注意观察指针的位置，此时`master`分支指向了`commit7`，`iss53`仍然指向`commit6`的位置，这就是平时开发最常见的情况，保留自己的开发分支进度，同时将自己的代码合并回提测/上线分支。问题来了，如果我反向合并呢？即在`iss53`分支上运行`git merge master`呢？

考虑一下这种场景：你的同事开发了一个需求A，并且已经通过上面的方式将他的代码合入了`master`分支；你负责的需求B此时开发了一半，突然发现有一个地方需要基于需求A的代码才能继续进行。这时候就可以将`master`上的代码合入你自己的分支`iss53`，此时你的代码已经包含了`master`中commit的代码，流程图如下：

![gitblog03](/images/git常用操作命令总结/gitblog03.png)

如果之后`master`分支再没有任何代码合入，你的需求开发完之后，合入`master`分支时将会是一次`fast-forward`合并，`master`的指针会直接快进指向`iss53`所在的位置。

另一个需要注意的是：代码合并时合并的只是你之前`commit`提交的代码修改，只对比这些文件。

#### 解决冲突

代码为什么冲突，用上图举例来说，自`commit3`之后，master分支和iss53分支分别有`commit4-hotfix`和`commit5/6`提交，如果这两个分支提交中修改了同一个文件的相同位置，则会报文件有冲突，需要解决冲突才能继续合并。解决流程就是：先手动解决文件a的冲突，然后`git add a.js`，接着`git commit -m 'fix conflict'`即可，此时可以推送到远端仓库。可以通过`git status`查看未解决冲突的文件。

![gitblog04](/images/git常用操作命令总结/gitblog04.png)

这个过程中可以通过`git merge --abort`来终止合并。

解决冲突最重要的是要细心，要仔细查看`git status`未解决冲突的文件，一一去解决。如果不加思索的直接`git add .`，则会将未解决冲突的文件移动到暂存区，然后提交上去。如果又推送到了远程仓库，处理起来会比较麻烦。如果遇到这种情况，可以用以下方法解决：

1. 在本地`master`分支上`git reset --hard HEAD^`，回退这次提交。
2. `git push -f`，强制推送本地分支到远端。因为这个时候你的本地分支已经落后于远端分支，如果正常流程`git pull`，你的代码会回到冲突后的提交状态，相当于白白`reset`了代码。如果直接使用`git push`，git会提示你本地分支落后于远端，必须`git pull`才能推送。使用`git push -f`，强制将远程仓库的代码替换为你当前本地分支的代码，相当于远端代码`reset`回退。
3. 再次`git merge iss53`，将代码合并回`master`，解决冲突，`add -> commit -> push`。

以上方法建立在你的冲突代码推送到远端后，你的队友没有人再次合并代码到`master`并且推送到远端。如果有这种情况，处理起来会更复杂，因为你的回退，然后强制推送，会将他们的代码回退掉！这时候就需要每一个队友的共同配合才能解决，异常复杂和麻烦。所以一定要细心细心再细心，尽量不要`reset`。

### 储藏与清理

有这样一个场景，你在你的分支iss54上开发需求，改了好多文件代码，这个时候老大来告诉你，develop分支上有个地方有问题，需要你去确认一下。一般这时候你切换分支是无法切换的，git会提示你先暂存才可以。但是你暂时又不想提交，因为提交过之后你的修改记录就没有了，下次再切换回来，哪些文件调整过，在`IDE`里就无法清晰的看到。这个时候你可以使用`git stash`相关命令：

- `git stash` :  将当前分支的修改先储存到栈内(清理一下工作区)，方便切换分支。
- `git stash list` : 查看栈内储存列表。
- `git stash apply` : 应用最近的一次储存，之前暂存过的文件会回到modified状态。
- `git stash apply --index` : 恢复到储存时的状态，包括已经暂存的文件状态。
- `git stash apply  stash@{2}` : 应用指定的储存。
- `git stash drop stash@{0}` :  删掉栈内指定的储存。

### 思维导图总结

![gitblog_xmind](/images/git常用操作命令总结/gitblog_xmind.png)

以上的命令及处理方法基本上已经可以满足日常的开发了。之前命令行用的少，都是使用IDE内自带的可视化工具来操作，虽然比较简单明了，但速度是无法跟命令行比的。不过使用IDE可以方便的切换分支，拉取远端分支，提示未解决的冲突等，日常使用要方便的多。推荐两者结合着使用，毕竟我们的目的是把工作做好，把任务完成，在git上不必过分纠结于过程。

参考：[git](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%85%B3%E4%BA%8E%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6)