---
title: Git先生的故事
date: 2017-11-05 22:44:05
tags: git
categories: git
---
> 很有意思的通俗的总结了git。👿

Git先生是一位很出名的摄影专家，他的主要职责是用它强大的拍摄技术帮我们共享成果，共创未来。为此它准备了许许多多的工具来实现这样的目标。下面我们就来看看Git先生的工作场所，和他为我们的一些痛点带来了哪些解决方案吧。
# 初次见面
老板（用户自己）新买了一块地皮（创建了一个目录），想聘请Git先生到此开设一个工作室来加快这个地皮的建设工作。老板用`git init` 招来了Git先生，Git先生在该目录下生成一个.git目录，用来作为自己的办公室，办公室用来记录自己的工作日志和成果。

让我们来从空中俯瞰下这块新的地皮，和Git先生为它所设计的蓝图吧。
[![](http://idiotsky.me/images1/git-story-1.jpg)](http://idiotsky.me/images1/git-story-1.jpg)

下面我们来解释下，这几个区域的作用：
Working Directory：Git先生的老板所买下的地皮，这个是实实在在物理层面的地皮，我们可以在上面种些花花草草，建点高楼大厦啥的。

Staging Area：Git先生摄影棚所在地，位置位于Git先生的办公室。每当老板完成了某件名垂青史的伟事，他就会命令Git先生把自己这个阶段所干的事情一五一十的搬到摄影棚拍照记录下来。`git add` 就是把修改搬到摄影棚，`git commit`就是命令Git先生拍照，而拍完照后，摄影棚马上会被打扫干净。

Repository：Git先生办公室的某个区域，专门用来存储照片用的。

Remote：这是一块云端区域，Git先生会在工作完一段时间后，就把自己的作品上传上去。这样做，一方面是用来保存自己的作品，以备意外发生，另一方面也是提供给其他有兴趣的老板们一起做这个项目。
<!-- more -->
# Go to work
一切准备妥当后，Git先生马上就投入到了紧张的工作当中。老板首先就迫不及待的在地皮上上种了一朵花，然后马上命令Git来拍照留念。
[![](http://idiotsky.me/images1/git-story-2.jpg)](http://idiotsky.me/images1/git-story-2.jpg)
当然结果是失败的，Git也很苦恼，自己已经把所有流程和老板说过一遍了，但老板还是会鲁莽行事。然后Git先生又向老板耐心的解释了一下针对Git目录下某个修改的4种状态。
> Untracked/Tracked
> Not Staged/Staged
> 比如你新建一个文件，它的状态就是 Untracked 的，你不能对 Untracked 的文件进行任何Git操作，除了先使用`git add` 让它先变为Tracked 状态。一个文件被Track后，以后的修改如果未用`git add`，那这个修改就叫Not Staged，需要add后，让它变为Staged才能进行Commit。

老板按照Git先生的说法又执行了一遍，这次他成功了。Git又向老板说，你可以用`git log`来查看我已经拍过的照片。

老板学会这招后，又给这块地皮创建了树、草，并且也都分别让Git先生拍了照片保存。

`git log`后我们看到了这三张照片，如果要查看详情还可以使用`git log -p`。
[![](http://idiotsky.me/images1/git-story-3.jpg)](http://idiotsky.me/images1/git-story-3.jpg)
> 导演注：Commit的id为对当前文件夹内容做SHA-1得来。

# 上点儿色吧
老板想把树涂成红色的，再给树取个名字叫big tree。他记得Git先生告诉过他可以用`git diff`来查看自己所做的改动
[![](http://idiotsky.me/images1/git-story-4.jpg)](http://idiotsky.me/images1/git-story-4.jpg)
看到了自己的修改后，老板满意的点点头，然后用`git add .`把它们都丢进了摄影棚。过后就出去忙其他事情了，回来后他发现自己忘记离开前做了啥事情了。此时他再用`git diff`查看，发现里面空空如也。老板愤怒的叫来了Git先生问他是咋回事。Git先生友善的解释了原因。

“`git diff`显示了您当前修改和我办公室中所记录的最新一张照片之间的差异，但是您已经把这些改动都挪到我的摄影棚里了，git diff就没法查看了，如果您想看我摄影棚里摆了哪些东西。你可以使用`git diff --staged/cached`哦”

老板按照Git先生所说，果然看到了他以前的修改记录。

> 导演注：stage相关的命令一般都与Staging Area相关，git add 也可以写成 git stage，这两个命令是一样的。

不过当老板看到他把树设成了红色，觉得不合理，想放弃这次修改。他要如何去做呢？

`git reset <file>` 把这个文件的修改从Staging Area中去除
`git checkout -- <file>` 放弃工作区文件的修改
> 导演注：这里使用 `git checkout <file>` 也行，之所以使用--，因为该命令与切换分支的命令一样，万一这个文件名和某个分支名重名，则`git checkout <file>`就变成切换分支了。

老板不禁感叹，幸好自己没有进行commit。Git先生告诉老板说，即使你commit了，也不用怕，我也有几种解决方案。
一，放弃整张照片
`git reset HEAD~1` HEAD表示指向最新那张照片的指针，～1表示要想起回退一张，此时我们有三种回退方式可选`--soft`表示只删除照片，照片的修改恢复到Staging Area中,`--mixed`不但删除照片，也不恢复Staging Area中的状态（不加选项时就为此中方法）,`--hard`不但删除照片，而且连工作区域的修改也被回退掉

二，我们再产生一张想放弃的照片的反修改的照片
`git revert <commit-id>` 产生此commit的反修改，并提交此处commit-id不必是最新一次，可以是任意处的。
第一种方案适用于，你的commit还未push到云端的场景，第二种，如果你的修改已经push到云端，那么为了尊重历史记录，最好就是生成一个方向修改来回退错误部分，让其他人知道历史。

> 导演注：HEAD指针记录了正在操作的节点的commit id，每个分支都有属于自己的HEAD指针，并且只有一个

# 拨弄你的指针
老板经过上次的事件，发现自己可能会因为一时冲动做出一些错误的决定。就问Git先生是否有办法把自己所有操作行为都记录下来，而且还允许自己撤销任何一种错误的操作。Git先生向老板解释说：所有对HEAD指针的操作都会被记录下来。

可以用`git reflog`查看到老板的所有HEAD操作
[![](http://idiotsky.me/images1/git-story-5.jpg)](http://idiotsky.me/images1/git-story-5.jpg)
最上面，我们可以看到是老板彻底回退了给树添加名字和颜色修改，执行了`git reset --hard HEAD~1`，而如果老板突然又后悔了，想恢复添加名字和颜色的修改。那么我们就可以通过执行`git reset --hard HEAD@{1}`来把操作回退到`HEAD@{1}`时，也就是加入名字和颜色那次commit。
> 这里的reset也有三个选项，`--soft`，`--mixed`，`--hard`，因为这里执行的是恢复操作，所以这三个选项在这里的作用也需要反过来理解，hard自不用说，就是完全恢复到某个操作时的状态，而soft表示，虽然把HEAD指针拨到了某个操作时的状态，但在staging area中会产生可以让恢复后的状态重新修改回来的修改，就像物质与反物质那样。mixed同理。

以上可能难以理解，这里我们再举个应用场景来说明下：
我们知道`git revert`每次只能回退某个commit，那我们如何同时revert掉多个commit呢。针对这种场景我们就可以进行以下操作
````
//拨动HEAD指针到5add0e9
git reset --hard 5add0e9 
//恢复到以前的commit处信息，并且在staging area生成了中反修改
git reset --soft HEAD@{1} 
// 注意此处用了--soft
git commit -m 'revert to 5add0e9'
````

# 多干点事
老板：地皮准备好了， 我们既要种花，种草，还要盖个楼房啥的，种花要花几天功夫，种草好又得花几天。这几样事咱能不能一起干呢？干完了，分别拍个照片再合一块岂不美哉。

Git先生：老板英明，针对这种情况我也早有准备。

`git branch <branch-name>` 创建一个分支，不加`<branch-name>`则为列出当前所有分支`git branch -d <branch-name> `删除分支 -D 为强制删除分支`git checkout <branch-name>` 切换到该分支下`git merge <branch-name>` 合并`<branch-name>`分支内容到本分支下
>针对老板说的这种情况，我们只需要创建如下分支。然后分别在flower里种花，grass中种草，building中盖楼，最后在master分支中把完成的照片merge过来就行。
>➜ GitTestRepo git:(master) git branch flower
>➜ GitTestRepo git:(master) git branch grass
>➜ GitTestRepo git:(master) git branch building
>➜ GitTestRepo git:(master) git branch
>building
>flower
>grass
>\* master

> 导演注：分支前加*号的为当前工作分支

# 合作共赢
Git先生：报告老板，您这块地皮很大，要是只有我们开发，那得花上很久时间，何不把它开放出去，让其他老板们一起进来把这块蛋糕做大呢。

老板：好主意，我们要怎么做呢？

Git先生：一切交给我，不过因为地皮开放出去后，涉及到多方共同开发。有些注意事项还希望老板能听我说道说道，否则危害甚大。

老板：请讲请讲！

如我们第一张图所示，我们可以利用git push来把自己所有的照片上传到云端，让其他人也可以参与进来开发。既然是云端，那么首先我们就需要指明下这个云端地皮的地址是哪里。

`git remote add origin https://github.com/CPPAlien/GitTestRepo.git`//这里一般用origin，当然你可以换成其他任何名，你也可以添加多个remote地址git remote -v 可以用来查看所有云端地址信息
`git push -u <remote> <branch> git push --set-upstream <remote> <branch>`用这两个命令来指明某个分支所对应到的remote地址。如果不指定，你在执行git push时需要明确写出remote和branch。
[![](http://idiotsky.me/images1/git-story-6.jpg)](http://idiotsky.me/images1/git-story-6.jpg)

因为是多人合作，所以就有可能别人在云端先与你提交了一些修改，而此时就需要进行git pull操作，把别人的修改拉取下来合并到本地。但直接git pull行为是不太安全的，因为它会直接产生merge行为，可能会导致你本地数据错乱。所以我们一般用git fetch，正确流程如下

`git fetch origin master`//获取origin上的master分支，会在本地自动创建一个的origin/master的临时分支。
`git log -p master..origin/master`//比较本地的master分支和远端的master分支，看下差别。
`git merge origin/master` 或 `git rebase origin/master` //如果差别是在自己的认知范围，那么就进行合并操作，这样本地的master分支就与云端保持一致了。如果本地有未push的commit，则会产生Merge的commit行为。Merge的过程中有可能因为多人对同一个文件的修改而造成冲突。
`git mergetool`//打开merge工具，merge完后保存，然后手动提交merge后的结果。完成上述操作后，就可以把自己本地的commit提交到云端了。

> 导演注：git merge 和 git rebase的区别，rebase是找到两者共同的commit处，把它者的修改接上去，然后再把自己的修改接在它者的修改后面，不会产生merge行为。看历史图时也不会像merge那样有分叉。

从以下执行rebase后的提示，也可知二三
>➜ GitTest git:(master) git rebase origin/masterFirst
>rewinding head to replay your work on top of it...Fast-forwarded master to origin/master.

# 其他小技巧
`git cherry-pick <commit-id>`
老板如果想把其他分支上的一些照片拣过来使用，可以使用此命令。如果该照片与本分支无冲突，则直接会在本分支上加上一条commit，如果有冲突，则需要解决冲突后重新提交。
`git stash/ git stash pop`
如果老板当前有些工作没有commit。但有些云端的commit或者其他分支的commit是自己后续开发所要依赖的，那就可以使用git stash把当前未提交的修改放入到缓存栈，等合并操作完成后，再用git stash pop，把修改再加回来，你可以用git stash list查看当前的缓存栈。

# 后记
Git和Mercurial 都是在2005年时出现，分别由Linus和Matt主导开发。而两者的出现也源于一个共同的事件，2005年初BitKeeper宣布向开源社区收费。Mercurial在英语中有反复无常的意思，而Git也可以翻译成无用之人，Matt直接说他取名Mercurial的用意就是讽刺BitKeeper的开发者。

from https://www.toutiao.com/i6484504341452440077/