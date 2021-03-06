---
title: git的图解和tips
date: 2017-11-05 22:44:05
tags: git
categories: git
---

> 总结git相关比较好的内容

# 图解

## 基本用法

[![](http://idiotsky.top/images3/git-1.png)](http://idiotsky.top/images3/git-1.png)

上面的四条命令在工作目录、暂存目录(也叫做索引)和仓库之间复制文件。
<!-- more -->
* git add files 把当前文件放入暂存区域。
* git commit 给暂存区域生成快照并提交。
* git reset -- files 用来撤销最后一次git add files，你也可以用git reset 撤销所有暂存区域文件。
* git checkout -- files 把文件从暂存区域复制到工作目录，用来丢弃本地修改。

你可以用 git reset -p, git checkout -p, or git add -p进入交互模式。

也可以跳过暂存区域直接从仓库取出文件或者直接提交代码。

[![](http://idiotsky.top/images3/git-2.png)](http://idiotsky.top/images3/git-2.png)


* git commit -a 相当于运行 git add 把所有当前目录下的文件加入暂存区域再运行。git commit.
* git commit files 进行一次包含最后一次提交加上工作目录中文件快照的提交。并且文件被添加到暂存区域。
* git checkout HEAD -- files 回滚到复制最后一次提交。

## 约定

后文中以下面的形式使用图片。

[![](http://idiotsky.top/images3/git-3.png)](http://idiotsky.top/images3/git-3.png)

绿色的5位字符表示提交的ID，分别指向父节点。分支用橘色显示，分别指向特定的提交。当前分支由附在其上的HEAD标识。 这张图片里显示最后5次提交，ed489是最新提交。 master分支指向此次提交，另一个maint分支指向祖父提交节点。

## 命令详解

### Diff

有许多种方法查看两次提交之间的变动。下面是一些示例。

[![](http://idiotsky.top/images3/git-4.png)](http://idiotsky.top/images3/git-4.png)

### Commit

提交时，git用暂存区域的文件创建一个新的提交，并把此时的节点设为父节点。然后把当前分支指向新的提交节点。下图中，当前分支是master。 在运行命令之前，master指向ed489，提交后，master指向新的节点f0cec并以ed489作为父节点。

[![](http://idiotsky.top/images3/git-5.png)](http://idiotsky.top/images3/git-5.png)

即便当前分支是某次提交的祖父节点，git会同样操作。下图中，在master分支的祖父节点maint分支进行一次提交，生成了1800b。 这样，maint分支就不再是master分支的祖父节点。此时，合并 (或者 衍合) 是必须的。

[![](http://idiotsky.top/images3/git-6.png)](http://idiotsky.top/images3/git-6.png)

如果想更改一次提交，使用 git commit --amend。git会使用与当前提交相同的父节点进行一次新提交，旧的提交会被取消。

[![](http://idiotsky.top/images3/git-7.png)](http://idiotsky.top/images3/git-7.png)

另一个例子是分离HEAD提交,后文讲。

### Checkout

checkout命令用于从历史提交（或者暂存区域）中拷贝文件到工作目录，也可用于切换分支。

[![](http://idiotsky.top/images3/git-8.png)](http://idiotsky.top/images3/git-8.png)

当给定某个文件名（或者打开-p选项，或者文件名和-p选项同时打开）时，git会从指定的提交中拷贝文件到暂存区域和工作目录。比如，git checkout HEAD~ foo.c会将提交节点HEAD~(即当前提交节点的父节点)中的foo.c复制到工作目录并且加到暂存区域中。（如果命令中没有指定提交节点，则会从暂存区域中拷贝内容。）注意当前分支不会发生变化。

[![](http://idiotsky.top/images3/git-9.png)](http://idiotsky.top/images3/git-9.png)

当不指定文件名，而是给出一个（本地）分支时，那么HEAD标识会移动到那个分支（也就是说，我们“切换”到那个分支了），然后暂存区域和工作目录中的内容会和HEAD对应的提交节点一致。新提交节点（下图中的a47c3）中的所有文件都会被复制（到暂存区域和工作目录中）；只存在于老的提交节点（ed489）中的文件会被删除；不属于上述两者的文件会被忽略，不受影响。

[![](http://idiotsky.top/images3/git-10.png)](http://idiotsky.top/images3/git-10.png)

如果既没有指定文件名，也没有指定分支名，而是一个标签、远程分支、SHA-1值或者是像master~3类似的东西，就得到一个匿名分支，称作detached HEAD（被分离的HEAD标识）。这样可以很方便地在历史版本之间互相切换。比如说你想要编译1.6.6.1版本的git，你可以运行git checkout v1.6.6.1（这是一个标签，而非分支名），编译，安装，然后切换回另一个分支，比如说git checkout master。然而，当提交操作涉及到“分离的HEAD”时，其行为会略有不同，详情见在下面。


### HEAD标识处于分离状态时的提交操作

当HEAD处于分离状态（不依附于任一分支）时，提交操作可以正常进行，但是不会更新任何已命名的分支。(你可以认为这是在更新一个匿名分支。)

[![](http://idiotsky.top/images3/git-11.png)](http://idiotsky.top/images3/git-11.png)

一旦此后你切换到别的分支，比如说master，那么这个提交节点（可能）再也不会被引用到，然后就会被丢弃掉了。注意这个命令之后就不会有东西引用2eecb。

[![](http://idiotsky.top/images3/git-12.png)](http://idiotsky.top/images3/git-12.png)

但是，如果你想保存这个状态，可以用命令git checkout -b name来创建一个新的分支。

[![](http://idiotsky.top/images3/git-13.png)](http://idiotsky.top/images3/git-13.png)

### Reset

reset命令把当前分支指向另一个位置，并且有选择的变动工作目录和索引。也用来在从历史仓库中复制文件到索引，而不动工作目录。

如果不给选项，那么当前分支指向到那个提交。如果用--hard选项，那么工作目录也更新，如果用--soft选项，那么都不变。

[![](http://idiotsky.top/images3/git-14.png)](http://idiotsky.top/images3/git-14.png)

如果没有给出提交点的版本号，那么默认用HEAD。这样，分支指向不变，但是索引会回滚到最后一次提交，如果用--hard选项，工作目录也同样。

[![](http://idiotsky.top/images3/git-15.png)](http://idiotsky.top/images3/git-15.png)

如果给了文件名(或者 -p选项), 那么工作效果和带文件名的checkout差不多，除了索引被更新。

[![](http://idiotsky.top/images3/git-16.png)](http://idiotsky.top/images3/git-16.png)

### Merge

merge 命令把不同分支合并起来。合并前，索引必须和当前提交相同。如果另一个分支是当前提交的祖父节点，那么合并命令将什么也不做。 另一种情况是如果当前提交是另一个分支的祖父节点，就导致fast-forward合并。指向只是简单的移动，并生成一个新的提交。

[![](http://idiotsky.top/images3/git-17.png)](http://idiotsky.top/images3/git-17.png)

否则就是一次真正的合并。默认把当前提交(ed489 如下所示)和另一个提交(33104)以及他们的共同祖父节点(b325c)进行一次三方合并。结果是先保存当前目录和索引，然后和父节点33104一起做一次新提交。

[![](http://idiotsky.top/images3/git-18.png)](http://idiotsky.top/images3/git-18.png)

### Cherry Pick

cherry-pick命令"复制"一个提交节点并在当前分支做一次完全一样的新提交。

[![](http://idiotsky.top/images3/git-19.png)](http://idiotsky.top/images3/git-19.png)

### Rebase

衍合是合并命令的另一种选择。合并把两个父分支合并进行一次提交，提交历史不是线性的。衍合在当前分支上重演另一个分支的历史，提交历史是线性的。 本质上，这是线性化的自动的 cherry-pick

[![](http://idiotsky.top/images3/git-20.png)](http://idiotsky.top/images3/git-20.png)

上面的命令都在topic分支中进行，而不是master分支，在master分支上重演，并且把分支指向新的节点。注意旧提交没有被引用，将被回收。

要限制回滚范围，使用--onto选项。下面的命令在master分支上重演当前分支从169a6以来的最近几个提交，即2c33a。

[![](http://idiotsky.top/images3/git-21.png)](http://idiotsky.top/images3/git-21.png)

同样有git rebase --interactive让你更方便的完成一些复杂操作，比如丢弃、重排、修改、合并提交。没有图片体现这些，细节看这里:git-rebase(1)

# tips

## 展示帮助信息
```sh
git help -g
```

## 回到远程仓库的状态
抛弃本地所有的修改，回到远程仓库的状态。  
```sh
git fetch --all && git reset --hard origin/master
```

## 重设第一个commit
也就是把所有的改动都重新放回工作区，并**清空所有的commit**，这样就可以重新提交第一个commit了
```sh
git update-ref -d HEAD
```

## 展示工作区和暂存区的不同
输出**工作区**和**暂存区**的different(不同)。
```sh
git diff
```

还可以展示本地仓库中任意两个commit之间的文件变动：
```sh
git diff <commit-id> <commit-id>
```

## 展示暂存区和最近版本的不同
输出**暂存区**和本地最近的版本(commit)的different(不同)。
```sh
git diff --cached
```

## 展示暂存区、工作区和最近版本的不同
输出**工作区**、**暂存区** 和本地最近的版本(commit)的different(不同)。
```sh
git diff HEAD
```

## 快速切换分支
```sh
git checkout -
```

## 删除已经合并到master的分支
```sh
git branch --merged master | grep -v '^\*\|  master' | xargs -n 1 git branch -d
```

## 展示本地分支关联远程仓库的情况
```sh
git branch -vv
```

## 关联远程分支
关联之后，`git branch -vv`就可以展示关联的远程分支名了，同时推送到远程仓库直接：`git push`，不需要指定远程仓库了。
```sh
git branch -u origin/mybranch
```

或者在push时加上`-u`参数
```sh
git push origin/mybranch -u
```

## 列出所有远程分支
-r参数相当于：remote
```sh
git branch -r
```

## 列出本地和远程分支
-a参数相当于：all
```sh
git branch -a
```

## 创建并切换到本地分支
```sh
git checkout -b <branch-name>
```

## 创建并切换到远程分支
```sh
git checkout -b <branch-name> origin/<branch-name>
```

## 删除本地分支
```sh
git branch -d <local-branchname>
```

## 删除远程分支
```sh
git push origin --delete <remote-branchname>
```

或者  
```sh
git push origin :<remote-branchname>
```

## 重命名本地分支
```sh
git branch -m <new-branch-name>
```

## 查看标签
```
git tag
```

展示当前分支的最近的tag
```sh
git describe --tags --abbrev=0
```

## 本地创建标签
```sh
git tag <version-number>
```

默认tag是打在最近的一次commit上，如果需要指定commit打tag：
```sh
$ git tag -a <version-number> -m "v1.0 发布(描述)" <commit-id>
```

## 推送标签到远程仓库
首先要保证本地创建好了标签才可以推送标签到远程仓库：
```sh
git push origin <local-version-number>
```

一次性推送所有标签，同步到远程仓库：
```
git push origin --tags
```

## 删除本地标签
```sh
git tag -d <tag-name>
```

## 删除远程标签
删除远程标签需要**先删除本地标签**，再执行下面的命令：
```sh
git push origin :refs/tags/<tag-name>
```

## 切回到某个标签
一般上线之前都会打tag，就是为了防止上线后出现问题，方便快速回退到上一版本。下面的命令是回到某一标签下的状态：
```sh
git checkout -b branch_name tag_name
```

## 放弃工作区的修改
```sh
git checkout <file-name>
```

放弃所有修改：  
```sh
git checkout .
```

## 恢复删除的文件
```sh
git rev-list -n 1 HEAD -- <file_path> #得到 deleting_commit

git checkout <deleting_commit>^ -- <file_path> #回到删除文件 deleting_commit 之前的状态
```

## 以新增一个commit的方式还原某一个commit的修改
```sh
git revert <commit-id>
```

## 回到某个commit的状态，并删除后面的commit
和revert的区别：reset命令会抹去某个commit id之后的所有commit
```sh
git reset <commit-id>  #默认就是-mixed参数。

git reset –mixed HEAD^  #回退至上个版本，它将重置HEAD到另外一个commit,并且重置暂存区以便和HEAD相匹配，但是也到此为止。工作区不会被更改。

git reset –soft HEAD~3  #回退至三个版本之前，只回退了commit的信息，暂存区和工作区与回退之前保持一致。如果还要提交，直接commit即可   

git reset –hard <commit-id>  #彻底回退到指定commit-id的状态，暂存区和工作区也会变为指定commit-id版本的内容
```

## 修改上一个commit的描述
```sh
git commit --amend
```

## 查看commit历史
```sh
git log
```

## 查看某段代码是谁写的
blame的意思为‘责怪’，你懂的。
```sh
git blame <file-name>
```

## 显示本地更新过HEAD的git命令记录
每次更新了HEAD 的git 命令比如 commint、amend、cherry-pick、reset、revert等都会被记录下来（不限分支），就像shell的history一样。
这样你可以reset 到任何一次更新了HEAD 的操作之后，而不仅仅是回到当前分支下的某个commit 之后的状态。
```
git reflog
```

## 修改作者名
```sh
git commit --amend --author='Author Name <email@address.com>'
```

## 修改远程仓库的url
```sh
git remote set-url origin <URL>
```

## 增加远程仓库
```sh
git remote add origin <remote-url>
```

## 列出所有远程仓库
```sh
git remote
```

## 查看两个星期内的改动
```sh
git whatchanged --since='2 weeks ago'
```

## 把A分支的某一个commit，放到B分支上
这个过程需要`cherry-pick`命令，[参考](http://sg552.iteye.com/blog/1300713#bc2367928)
```sh
git checkout <branch-name> && git cherry-pick <commit-id>
```

## 给git命令起别名
简化命令

```sh
git config --global alias.<handle> <command>

比如：git status 改成 git st，这样可以简化命令

git config --global alias.st status
```

## 存储当前的修改，但不用提交commit

```sh
git stash
```

## 保存当前状态，包括untracked的文件
untracked文件：新建的文件
```sh
git stash -u
```

## 展示所有stashes
```sh
git stash list
```

## 回到某个stash的状态
```sh
git stash apply <stash@{n}>
```

## 回到最后一个stash的状态，并删除这个stash
```sh
git stash pop
```

## 删除所有的stash
```sh
git stash clear
```

## 从stash中拿出某个文件的修改
```sh
git checkout <stash@{n}> -- <file-path>
```

## 展示所有tracked的文件
```sh
git ls-files -t
```

## 展示所有untracked的文件
```sh
git ls-files --others
```

## 展示所有忽略的文件
```sh
git ls-files --others -i --exclude-standard
```

## 强制删除untracked的文件
可以用来删除新建的文件。如果不指定文件文件名，则清空所有工作的untracked文件。`clean`命令，**注意两点**：  
1. clean后，删除的文件无法找回
2. 不会影响tracked的文件的改动，只会删除untracked的文件

```sh
git clean <file-name> -f
```

## 强制删除untracked的目录
可以用来删除新建的目录，**注意**:这个命令也可以用来删除untracked的文件。详情见上一条

```sh
git clean <directory-name> -df
```

## 展示简化的commit历史
```sh
git log --pretty=oneline --graph --decorate --all
```

## 把某一个分支到导出成一个文件
```sh
git bundle create <file> <branch-name>
```

## 从包中导入分支
新建一个分支，分支内容就是上面`git bundle create`命令导出的内容
```sh
git clone repo.bundle <repo-dir> -b <branch-name>
```

## 执行rebase之前自动stash
```sh
git rebase --autostash
```

## 从远程仓库根据ID，拉下某一状态，到本地分支
```sh
git fetch origin pull/<id>/head:<branch-name>
```

## 详细展示一行中的修改
```sh
git diff --word-diff
```

## 清除gitignore文件中记录的文件
```sh
git clean -X -f
```

## 展示所有alias和configs
**注意：** config分为：当前目录（local）和全局（golbal）的config，默认为当前目录的config

```sh
git config --local --list (当前目录)
git config --global --list (全局)
```

## 展示忽略的文件
```sh
git status --ignored
```

## commit历史中显示Branch1有的，但是Branch2没有commit
```sh
git log Branch1 ^Branch2
```

## 在commit log中显示GPG签名
```sh
git log --show-signature
```

## 删除全局设置
```sh
git config --global --unset <entry-name>
```

## 新建并切换到新分支上，同时这个分支没有任何commit
相当于保存修改，但是重写commit历史  
```sh
git checkout --orphan <branch-name>
```

## 展示任意分支某一文件的内容
```sh
git show <branch-name>:<file-name>
```

## clone下来指定的单一分支
```sh
git clone -b <branch-name> --single-branch https://github.com/user/repo.git
```

## 忽略某个文件的改动
关闭 track 指定文件的改动，也就是 Git 将不会在记录这个文件的改动
```
git update-index --assume-unchanged path/to/file
```

恢复 track 指定文件的改动
```
git update-index --no-assume-unchanged path/to/file
```

## 忽略文件的权限变化
不再将文件的权限变化视作改动
```sh
git config core.fileMode false
```

## 以最后提交的顺序列出所有Git分支
最新的放在最上面   

```sh
git for-each-ref --sort=-committerdate --format='%(refname:short)' refs/heads/
```

## 在commit log中查找相关内容
通过grep查找，given-text：所需要查找的字段

```sh
git log --all --grep='<given-text>'
```

## 把暂存区的指定file放到工作区中
不添加参数，默认是-mixed
```sh
git reset <file-name>
```

## 强制推送
```sh
git push -f <remote-name> <branch-name>
```


# 参考

http://marklodato.github.io/visual-git-guide/index-zh-cn.html?no-svg

https://github.com/521xueweihan/git-tips/blob/master/README.md