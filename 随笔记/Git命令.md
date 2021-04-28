# Git命令

## 入门

- 创建仓库

```sh
git init
```

- 将文件添加到git

```sh
git add readme.txt
```

- 提交

```sh
git commit -m "xxx"
```

> `git commit`命令，`-m`后面输入的是本次提交的说明，可以输入任意内容，当然最好是有意义的，这样你就能从历史记录里方便地找到改动记录。

`git commit`命令执行成功后会告诉你，`1 file changed`：1个文件被改动（我们新添加的readme.txt文件）；`2 insertions`：插入了两行内容（readme.txt有两行内容）。

## 基础

- 查看日志

```sh
git log
#显示一行
git log --pretty=oneline
```

- 版本回退

```sh
#回退上个版本
git reset --hard HEAD^
#通过版本号跳转指定位置
git reset --hard 123xxx 
```

> git 使用 HEAD 表示当前版本 ，上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个`^`比较容易数不过来，所以写成`HEAD~100`

- 查询操作记录

```sh
git reflog
```

- 查看状态

```sh
git status
```

- 对比区别

```sh
git diff HEAD --readme.txt
```

- 丢弃工作区的修改

```sh
git checkout -- readme.txt
```

- 撤销缓存区的修改

```sh
git reset HEAD readme.txt
```

- 删除文件

```sh
git rm test.txt
```

>  先手动删除文件，然后使用git rm <file>和git add<file>效果是一样的。

删除了一样可以回复通过`git checkout -- test.txt`

> 注意：从来没有被添加到版本库就被删除的文件，是无法恢复的

## 远程

- 关联

```sh
git remote add origin git@github.com:xx/xxx.git
```

- 推送本地库

```sh
git push origin master
#首次提交关联本地与远程仓库
git push -u origin master
```

> 由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令

- 删除远程仓库

```sh
#查看远程信息
git remote -v
#删除
git remote rm origin
```

- 克隆

```sh
git clone xxxx.git
```

## 分支管理

`HEAD`严格来说不是指向提交，而是指向`master`，`master`才是指向提交的，所以，`HEAD`指向的就是当前分支。

一开始的时候，`master`分支是一条线，Git用`master`指向最新的提交，再用`HEAD`指向`master`，就能确定当前分支，以及当前分支的提交点：

![git-br-initial](https://gitee.com/kongyin/picture_bed/raw/master/wx_picture/0)

每次提交，`master`分支都会向前移动一步，这样，随着你不断提交，`master`分支的线也越来越长。

当我们创建新的分支，例如`dev`时，Git新建了一个指针叫`dev`，指向`master`相同的提交，再把`HEAD`指向`dev`，就表示当前分支在`dev`上：

![git-br-create](https://gitee.com/kongyin/picture_bed/raw/master/wx_picture/l)

你看，Git创建一个分支很快，因为除了增加一个`dev`指针，改改`HEAD`的指向，工作区的文件都没有任何变化！

不过，从现在开始，对工作区的修改和提交就是针对`dev`分支了，比如新提交一次后，`dev`指针往前移动一步，而`master`指针不变：

![git-br-dev-fd](https://gitee.com/kongyin/picture_bed/raw/master/wx_picture/202104250837.png)

假如我们在`dev`上的工作完成了，就可以把`dev`合并到`master`上。Git怎么合并呢？最简单的方法，就是直接把`master`指向`dev`的当前提交，就完成了合并：

![git-br-ff-merge](https://gitee.com/kongyin/picture_bed/raw/master/wx_picture/2021042508371.png)

所以Git合并分支也很快！就改改指针，工作区内容也不变！

合并完分支后，甚至可以删除`dev`分支。删除`dev`分支就是把`dev`指针给删掉，删掉后，我们就剩下了一条`master`分支：

![git-br-rm](https://gitee.com/kongyin/picture_bed/raw/master/wx_picture/2021042508372.png)

- 创建分支

```sh
git checkout -b dev
```

`git checkout`命令加上`-b`参数表示创建并切换，相当于以下两条命令

```sh
git branch dev
git checkout dev
```

- 查看分支

```sh
git branch
```

> `git branch`命令会列出所有分支，当前分支前面会标一个`*`号

- 合并分支

```sh
git merge dev
```

- 删除分支

```sh
git branch -d dev
```

- 创建并切换分支

```sh
git switch -c dev
#切换已有的分支
git switch master
```

- 查看分支合并记录

```sh
git log --graph --pretty=oneline --abbrev-commit
```

- Fast forward切换分支

```sh
git merge --no-ff -m "xxx" dev
```

> 合并分支时，加上`--no-ff`参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而`fast forward`合并就看不出来曾经做过合并

- 暂存stash功能

```sh
##暂存
git stash
##查看 工作
git stash list
##恢复工作
git stash apply
## 删除工作
git stash drop
##恢复并删除
git stash pop
```

- 复制一个特定的提交到当前分支

```sh
git cherry-pick 4c805e2
```

- 强制删除

```sh
git branch -D xxx
```

- 查看远程仓库的信息

```sh
#查看远程仓库信息
git remote 
#查看详情
git remote -v
```

- 推送分支

```sh
git push origin master
## 推送 dev
git push origin dev
```

- 抓取分支

```sh
git clone git@xxx.git
#默认master 查看
git branch 
#切换至 dev
git checkout -b origin/dev
##添加工作区
git add xxx
#提交缓存
git commit -m "add xxx"
# 推送
git pull origin dev
```

冲突时

```sh
$ git pull
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

    git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

    git branch --set-upstream-to=origin/<branch> dev
```

根据提示建立链接

```sh
git branch --set-upstream-to=origin/dev dev
## 即可拉去
git pull
# 解决冲突后提交并推送
git commit -m "xxx"
git push origin dev
```

- Rebase

```sh
#变基
git rebase
```

> rebase操作可以把本地未push的分叉提交历史整理成直线
>
> rebase的目的是使得我们在查看历史提交的变化时更容易，因为分叉的提交需要三方对比

## 标签

- 打标签

```sh
# 查看分支
git branch
# 切换分支
git checkout master
# 打标签
git tag v1.0
# 查看所有标签
git tag
```

- 指定提交打标签

```sh
# 查看提交信息
git log --pretty=online --abbrev-commit
# 根据提交 id 打标签
git tag v0.9 xxxx
# 查看
git tag
```

-  查看标签信息

```sh
git show v0.9
```

- 创建带有说明的标签

```sh
git tag -a v0.1 - m "xxx" xxx
```

> 创建带有说明的标签，用`-a`指定标签名，`-m`指定说明文字

- 删除本地标签

```sh
git tag -d v0.1
```

- 推送标签

```sh
git push oirgin v1.0
# 推送全部 未推送本地标签
git push origin --tags
```

- 删除远端标签

```sh
# 先删除本地
git tag -d v0.1
# 删除远端
git push origin :refs/tags/v0.1
```

