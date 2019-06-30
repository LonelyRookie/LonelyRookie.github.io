---
layout:     post                    # 使用的布局（不需要改）
title:      Git合并特定commits到另一个分支               # 标题 
subtitle:   Git合并特定commits到另一个分支          #副标题
date:       2019-06-30              # 时间
author:     HuangCanCan             # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - git
---

## 前言

这周遇到一个使用git有点特别的操作。如何从一个分支合并特定的commits到另一个分支。有时候只合并你需要的那些commits，不需要的commits就不合并进去了。平常都很少遇到这样的情况，所以查了一下资料，记录一下。

## 合并某个分支上的单个commit

    使用git log或查git log --graph --pretty=oneline --abbrev-commit 查看提交的信息,记住commit id.

    git checkout [branch-name] 切换到合并的分支
    git cherry-pick <commit-id> 把某个commit id提交合并到当前分支

如：

    dd2e86 - 946992 -9143a9 - a6fd86 - 5a6057 [master]
        \
        76cada - 62ecb3 - b886a0 [feature]

比如，feature 分支上的commit 62ecb3 非常重要，它含有一个bug的修改，或其他人想访问的内容。无论什么原因，你现在只需要将62ecb3 合并到master，而不合并feature上的其他commits，所以我们用git cherry-pick命令来做：

    git checkout master  先切换到master分支
    git cherry-pick 62ecb3  再使用cherry-pick命令

现在62ecb3 就被合并到master分支，并在master中添加了commit（作为一个新的commit）。cherry-pick 和merge比较类似，如果git不能合并代码改动（比如遇到合并冲突），git需要你自己来解决冲突并手动添加commit。

## 合并某个分支上的一系列commits

在一些特性情况下，合并单个commit并不够，你需要合并一系列相连的commits。这种情况下就不要选择cherry-pick了，rebase 更适合。

    git checkout -b <newBranchName> <to-commit-id>
    创建一个新的分支，指明新分支的最后一个commit

    git rebase --onto <branchName> <from-commit-id>
    变基这个新的分支到最终要合并到的分支，指明从哪个特定的commit开始

如：
还以上面的为例，假设你需要合并feature分支的commit 76cada ~62ecb3 到master分支。
首先需要基于feature创建一个新的分支，并指明新分支的最后一个commit：

    git checkout -b <newbranch> 62ecb3

然后，rebase这个新分支的commit到master（--onto master）。76cada^ 指明你想从哪个特定的commit开始。

    git rebase --onto master 76cada^

得到的结果就是feature分支的commit 76cada ~62ecb3 都被合并到了master分支。

`再合并的过程中可能出现冲突，出现冲突，必须手动解决后，然后
运行git rebase --continue。`

    系统对冲突的提示：
    fix conflicts and then run "git rebase --continue"
    use "git rebase --skip" to skip this patch
    use "git rebase --abort" to check out the original branch
