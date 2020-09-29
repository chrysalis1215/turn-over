---
title: rebase不再是黑魔法
---
# 为什么我们要用rebase
一般被安利rebase的原因都是追求干净整洁的线性提交。
我被安利的原因在于，在我们团队采用`git-flow`开发模式。在develop分支有很多feature分支的合入，由于merge是按照提交时间线来做合并，不同feature的提交在develop中，你中有我，我中有你。
这会导致一个问题，如果有一个feature合并后想回滚
- 使用reset回滚，大量的散乱的commit中寻找要保留相关feature的commit会非常吃力；
- 使用revert的话，由于历史提交记录不会被擦除，若未同步团队中的每一个人，很有可能会被误解。[例子：merge分支后进行revert操作](#容易误解的地方)

{% asset_img git-flow.png %}
<center>git-flow模式中的feature合入</center>

# 如何理解git rebase?
rebase是将一系列提交按照<font color="red">**原有次序**</font>依次应用到另一个分支上。
详细的来说，rebase会先丢弃本地修改，以目标分支为的提交为基线，将本分支的变更，通过patch的方式，一个一个应用到自己的分支上。
所以，rebase的过程中如果有冲突的话，会跳到一个游离的节点，直到解决完所有的冲突，git才自动会切换到原分支上； 同时这种操作也只会发生在local。

{% asset_img rebase-init.svg %}
<center>git rebase效果图</center>


# git rebase的原理是什么？
当我们执行git rebase的时候，实际上是通过`git patch`去完成变更
> git patch =  commit info + git diff 

patch模式会把`变更内容`和`提交信息`，一起应用到当前分支上去，但是会生成一个新的commit SHA。
以下都是基于git patch的模式:
- `rebase`
- `cherry-pick`
- `reset`


rebase可以看做是一个一个的cherry-pick,  而reset可以理解为一个一个的逆向的cherry-pick。


# 实战记录
## 分支情况介绍
{% asset_img rebase01.png %}

1、develop分支是基于master的m2提交上拉出来的。
2、develop分支新增了 d1 d2 d3的提交
3、master分支新增了m3的提交

## develop合并到master中
如果想把develop  →  master，实际生产环境下，由于master是受保护分支，无法在master上去解决冲突，一般是develop先去合并master, 本地解决冲突，再执行合并master的操作。

### 方式一、merge master

#### 处理冲突
{% asset_img rebase02.png %}
保留两次修改后，develop中的提交以时间线为基数，提交的commit是混合的。

#### 提交记录
{% asset_img rebase03.png %}
含有git默认的提交记录，非常杂乱。
#### 回滚操作
```
// 普通回滚操作
git revert commitSHA

// 回滚合并操作
git revert commitSHA -m  // m 回退到本分支（1）， 或者merge的目标分支（2）
```
下面回滚合并操作
{% asset_img rebase04.png %}
回滚后的git graph
{% asset_img rebase05.png %}
#### 容易误解的地方
git log会记录每一次操作记录,即使这个提交被revert后，也不会消除当初的提交记录。但是不熟悉的人看这个提交的线时，**会误认为develop上是有m3的改动的**！！！
{% asset_img rebase06.png %}

### 方式二、rebase master
#### 处理冲突
{% asset_img rebase07.png %}
手动去处理冲突, 发现rebase的incoming change 和 merge正好相反。 （因为rebase会暂时丢弃本地的改动，以rebase的目标改动为主）
{% asset_img rebase08.png %}
##### 选择一、以目标分支（master）的修改为准
需要直接执行

```
git rebase --skip
```

错误示范：
```
// 手动修改的解决冲突后
git add .
git rebase --continue
```
会报错 No changes - did you forget to use 'git add'? 
{% asset_img rebase09.png %}

此时依然需要执行git rebase --skip
{% asset_img rebase10.png %}
d2的提交有冲突和m3有冲突
{% asset_img rebase11.png %}
d3依然冲突
{% asset_img rebase12.png %}
继续跳过, finally，我的文件和git graph和commit一模一样的了。
{% asset_img rebase13.png %}
因为我丢弃了3次develop的commit和commit-info,develop分支回到和master的分支一样。

#### 选择二、以源分支（develop）的修改为准
手动解决冲突
{% asset_img rebase14.png %}

```
// fix conflict 
git add .
git rebase --continue

```
{% asset_img rebase15.png %}
后面两个改动将会被自动应用，并且分支提交由游离的节点回归到源分支（develop）中。

{% asset_img rebase16.png %}
<center>最后的合并入master的提交非常整洁</center>

#### 回滚操作
```
git reset commitSHA
```

此时可以使用reset进行回滚，非常整洁
{% asset_img rebase18.png %}

---
---
date: 2020-09-25 15:41:24
tags: git
---
