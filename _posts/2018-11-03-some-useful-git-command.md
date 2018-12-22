---
layout: post
title: Git 中几个好用的操作
date: 2018-11-03 08:58:35 +0800
categories: 
---


#### 1: revert "revert"

某次错误的将 feature/x 合并到了 dev，因为当时的 dev 不考虑合并该 feature，自然解决办法是 revert 这次 merge。但是后来需要 feature/x 再次合并到 dev，结果却发现 feature/x 上后面提交的一些代码没有了，折腾下尝试把 dev 合到 feature/x 上结果还是丢代码。后来在伟大的 stackOverflow 上发现了可以再一次 revert “revert” 那次操作，试了下果然可以。真的强到不行。后来自己想了一下第一次的 revert 并不是撤销你的所有操作，它也是一个会将 git 状态指针改变的一个行为。

我在想如果第一次的 feature 用 rebase 合并会怎么样？ 笑哭。

#### 2：cherry-pick

只要你的 commit 信息在，就不怕在任何分支冲掉你的代码。

#### 3：git stash

当前分支还在开发，要切到别的分支做一些其他的事情，但是当前分支构不成 commit 的粒度，可以用 git stash 暂存。详细的用法可以 Google，在别的分支做好事情后，回到 stash 的那个分支，要记得 git stash pop。

#### 4：prune origin

git fetch 去拉取所有的远程分支，但是本地还存在许多远端早就删除的好多分支，导致本地分支目录杂乱，这时候执行 “git remote prune origin” 就可以清除掉本地那些远端不存在的分支。

#### 5：git reflog

这个命令可以记录当前仓库自从 clone 之后所有的操作，merge、rebase、commit 等，即使当前 commit 不小心被覆盖也没事。

综上，个人觉得最重要的一点就是每次的 commit 粒度和 commit 信息一定要合理清晰，这样才能更为精准的进行版本控制管理。