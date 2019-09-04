## git 操作整理

### 一：必备知识点
![](https://ae01.alicdn.com/kf/He497762411874184b22e6a0d67760afdi.png)

#### 仓库
1. **Remote**：远程主仓库
2. **Repository/History**：本地仓库
3. **Stage/Index**：Git追踪树，暂存区
4. **workspace**：本地工作区（即你编辑的代码）

### 二：git add 提交到暂存区，出错怎么办
一般代码提交流程为：
1. **Working Directory（工作区）** --> `git status` 查看工作区状态
2. **Stage/Index（暂存区）** --> `git add .` 将所有修改加入暂存区
3. **Repository/History（本地仓库）** --> `git commit -m '请求描述'` 将代码提交到本地仓库
4. **Remote（远程仓库）** --> `git push` 将本地仓库代码更新到远程仓库

#### 场景1：
当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。

**git checkout**
```
// 丢弃工作区的修改
git checkout --<文件名>
```
#### 场景2：
当你不但改乱了工作区某个文件的内容，还添加到了暂存区，想丢弃修改，分两步，第一步用命令`git reset HEAD file`，就回到了场景1，第二步按场景1操作。

### 三：git commit 提交到本地仓库，出错怎么办？
1. **提交信息出错**

    更改commit信息
    ```
    git commit --amend -m "新提交信息"
    ```

2. **漏提交**
    
    commit时，遗漏提交部分更新，有两种解决方案：
    + 方案一：再次commit
        ```
        git commit -m "提交消息"
        ```
        此时，git上会出现两次commit
    + 方案二：遗漏文件提交到之前commit上
        ```
        git add missed-file // missed-file 为遗漏提交文件
        git commit --amend --no-edit
        ```
        `--no-edit`表示提交消息不会更改，在git上仅为一次提交
3. **提交错误文件，回退到上一个commit版本，再commit**

    **<font color=orange>git reset</font>**

    删除指定的commit
    ```
    // 修改版本库，修改暂存区，修改工作区
    git reset HEAD <文件名> //把暂存区的修改撤销掉（unstage），重新放回工作区
    // git 版本回退，回退到特定的commit_id版本，可以通过git log查看提交历史，以便确定要回退到哪个版本（commit 之后的即为ID)
    git reset --hard commit_id 
    // 将版本库回退1个版本，不仅仅是将本地版本库的头指针全部重置到指定版本，也会重置暂存区，并且会将工作区代码也回退到这个版本
    git reset --hard HEAD~1
    // 修改版本库，保留暂存区，保留工作区
    // 将版本库回退1个版本，软回退表示将本地版本库的头指针全部重置到指定版本，且将这次提交之后的所有变更都移动到暂存区
    git reset --soft HEAD~1
    ```

    **<font color=orange>git revert</font>**

    撤销某次操作，此次操作之前和之后的commit和history都会保留，并且把这次撤销作为一次最新的提交
    ```
    // 撤销前一次 commit
    git revert HEAD
    // 撤销前前一次 commit
    git revert HEAD^
    // (比如：fa042ce57ebbe5bb9c8db709f719cec2c58ee7ff)撤销指定的版本，撤销也会作为一次提交进行保存
    git revert commit_id
    ```

    **<font color=orange>`git reset`和`git revert`的区别：</font>**
    + `git revert`是用一次新的commit来回滚之前的commit，`git reset`是直接删除指定的commit
    + 在回滚这一操作上看，效果差不多。但是在日后继续merge以前的老版本时有区别。因为`git revert`是用一次逆向的commit“中和”之前的提交，因此日后合并老的branch时，导致这部分改变不会再次出现，但是`git reset`是之间把某些commit在某个branch上删除，因而和老的branch再次merge时，这些被回滚的commit应该还会被引入
    + `git reset`是把HEAD向后移动了一下，而`git revert`是HEAD继续前进，只是新的commit的内容和要revert内容正好相反，能够抵消要被revert的内容

### 四：常用命令
#### 1.<font color=orange> 初始开发git操作流程</font>
+ 克隆最新主分支项目代码 `git clone 地址`
+ 创建本地分支 `git branch 分支名`
+ 查看本地分支 `git branch`
+ 查看远程分支 `git branch -a`
+ 切换分支`git checkout 分支名`（一般修改未提交则无法切换，大小写问题经常会有，可强制切换`git checkout 分支名 -f`非必须慎用）
+ 将本地分支推送到远程分支 `git push 远程仓库 本地分支:远程分支`

#### 2.<font color=orange> git fetch</font>
将某个远程主机的更新，全部/分支取回本地（此时只更新了Repository）它取回的代码对你本地的开发代码没有影响，如需彻底更新需合并或使用`git pull`

#### 3.<font color=orange> git pull</font>
拉取远程主机某分支的更新，再与本地的指定分支合并（相当于fetch加上了合并分支功能的操作）

#### 4.<font color=orange> git push</font>
将本地分支的更新，推送到远程主机，其命令格式与`git pull`相似

#### 5.<font color=orange> 分支操作</font>
+ 使用git下载指定分支命名为 `git clone -b 分支名仓库地址`
+ 拉取远程新分支 `git checkout -b serverfix origin/serverfix`
+ 合并本地仓库 `git merge hotfix` (将hotfix分支合并到当前分支)
+ 合并远程分支 `git merge origin/serverfix`
+ 删除本地分支 `git branch -d hotfix` （删除本地hotfix分支）
+ 删除远程分支 `git push origin --delete serverfix`
+ 上传新命名的本地分支 `git push origin newName`
+ 创建新分支 `git branch branchName` (创建名为branchName的本地分支)
+ 切换到新分支 `git checkout branchName`
+ 创建并切换分支 `git checkout -b branchName`
+ 查看本地分支 `git branch`
+ 查看远程仓库所有分支 `git branch -a`
+ 本地分支重命名 `git branch -m oldName newName`
+ 重命名远程分支对应的本地分支 `git branch -m oldName newName`
+ 从暂存区删除文件 `git rm --cached name`
+ 远程仓库文件的移除 `git remote rm remote-name` （注意这个可能不可用）
+ 把修改后的本地分支与远程分支关联 `git branch --set-upstream-to origin/newName`

### 五：优化操作
#### 1.<font color=orange>拉取代码 pull --rebase</font>
在团队协作过程中，假设你和你的同伴在本地中分别有各自的新提交，而你的同伴先于你`push`了代码到远程分支上，所以你必须先执行`git pull`来获取同伴的提交，然后才能`push`自己的提交到远程分支。

而按照git的默认策略，如果远程分支和本地分支之间的提交线图有分寸的话（即不是fast-forwarded），git会执行一次`merge`操作，因此产生一次没意义的提交记录，从而造成了像上图那样的混乱。

其实在pull操作的时候，使用`git pull --rebase`选型即可很好的解决上述问题。加上`--rebase`参数的作用是，提交线图有分叉的话，git会rebase策略来代替默认的merge策略。

> 需要注意的是，使用`git pull --rebase`比直接pull容易导致冲突的产生，如果预期冲突比较多的话，建议还是直接pull。
>
>git pull = git fetch + git merge
>git pull --rebase=git fetch + git rebase

#### 2.<font color=orange> 合并代码 merge --no-of</font>
上述的`git pull --rebase`策略目的是修整提交线图，使其形成一条直线，而即将要用到的`git merge --no-of branch-name`策略偏偏是反其道，刻意弄出提交线图分叉出来。

假设你在本地准备合并两个分支，而刚好这两个分支是fast-forwarded的，那么直接合并后你得到一个直线的提交线图，当然这样没什么坏处，但如果你像更清晰的告诉你同伴：**这一系列的提交都是为了实习同一个目的**，那么你可以刻意的将这次提交内容弄出一次提交线图交叉。

#### 3.<font color=orange> 总结</font>
+ 使用`git pull --rebase`和`git merge --no-ff`其实和直接使用`git pull` &emsp; `git merge`得到的代码应该是一样的
+ 使用`git pull --rebase`主要是为了将提交线图平坦化，而`git merge --no-ff`则是刻意制造分叉
