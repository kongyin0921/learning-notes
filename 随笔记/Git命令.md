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

![git-br-initial](https://www.liaoxuefeng.com/files/attachments/919022325462368/0)

每次提交，`master`分支都会向前移动一步，这样，随着你不断提交，`master`分支的线也越来越长。

当我们创建新的分支，例如`dev`时，Git新建了一个指针叫`dev`，指向`master`相同的提交，再把`HEAD`指向`dev`，就表示当前分支在`dev`上：

![git-br-create](https://www.liaoxuefeng.com/files/attachments/919022363210080/l)

你看，Git创建一个分支很快，因为除了增加一个`dev`指针，改改`HEAD`的指向，工作区的文件都没有任何变化！

不过，从现在开始，对工作区的修改和提交就是针对`dev`分支了，比如新提交一次后，`dev`指针往前移动一步，而`master`指针不变：

![git-br-dev-fd](https://www.liaoxuefeng.com/files/attachments/919022387118368/l)

假如我们在`dev`上的工作完成了，就可以把`dev`合并到`master`上。Git怎么合并呢？最简单的方法，就是直接把`master`指向`dev`的当前提交，就完成了合并：

![git-br-ff-merge](https://www.liaoxuefeng.com/files/attachments/919022412005504/0)

所以Git合并分支也很快！就改改指针，工作区内容也不变！

合并完分支后，甚至可以删除`dev`分支。删除`dev`分支就是把`dev`指针给删掉，删掉后，我们就剩下了一条`master`分支：

![git-br-rm](https://www.liaoxuefeng.com/files/attachments/919022479428512/0)

- 创建分支

```sh
git checkout -b dev
```

`git checkout`命令加上`-b`参数表示创建并切换，相当于以下两条命令

```sh
git branch dev
git checkout dev
```

