---
title: Git常用命令总结
date: 2019-08-13 09:45:39 +0800
categories: [Git]
tags: [git]
---
## 1.配置用户信息
`git config --global user.name "<name>"`  设置默认名字

`git config --global user.email "<email>"`  设置默认Email

## 2.基本命令
### 2.1 常用命令
查看帮助

```
git help <cmd>
git <cmd> --help
man git-<cmd>
```

`git <cmd> -h`  查看简短帮助

`git init`  在当前目录创建空仓库

`git add <file>`  将文件添加到暂存区

`git commit [-a] -m "<message>"`  将暂存区中的文件提交到仓库
* `-a`：自动将所有已跟踪的文件添加到暂存区
* 注意：若git add后又对文件做了修改，则提交前需再次git add，否则只会提交上一次git add的内容
* 提交信息可以是单行，也可以是多行：第一行称为“标题”，之后是一个空行，空行之后的部分称为“主体”（见`git commit --help`的DISCUSSION一节）。要指定多行提交信息，省略`-m`选项，Git将自动打开一个文本编辑器（默认是Vim），在其中输入提交信息即可

`git commit --amend [-m "<message>"]`  替换上一次提交，如果不指定提交信息则使用原提交信息
* 如果文件有变化则相当于将修改合并到上一次提交
* 如果文件没有变化则相当于修改提交信息

`git status`  查看仓库状态

### 2.2 比较文件的差别
`git diff [<file>]`  比较工作目录和暂存区

`git diff --cached [<commit>] [<file>]`  比较暂存区和指定提交，可用HEAD表示当前分支指向的提交，也可用提交的SHA-1或其前几位表示指定的提交，默认为HEAD；`--staged`同`--cached`

`git diff <commit> [<file>]`  比较工作目录和指定提交

`git diff <commit1> <commit2>`

`git diff <commit1>..<commit2>`  
比较指定的两次提交（从commit1到commit2如何变化）

`git diff <commit1>...<commit2>`  比较commit1和commit2的公共祖先与commit2

![diff](https://marklodato.github.io/visual-git-guide/diff.svg)

### 2.2.1 GitHub pull request diff
假设在master分支的提交A上创建了一个主题分支topic，并创建了两次提交B和G，期间两次将master分支合并到topic分支，合并提交为D和H，提交历史如下图所示：

```
    B--D--G--H--topic
   /  /     /
--A--C--E--F--master
```

如果想查看真正属于topic分支引入的diff（即提交B和G的累加diff，而不包括提交D和H，就像在GitHub上创建一个从topic分支到master分支的pull request所能看到的diff），可使用上面的三点形式git diff命令：`git diff master...topic`，该命令将比较master分支和topic分支的公共祖先（提交F）与topic分支最新位置（提交H），等价于`git diff $(git merge-base master topic) topic`，即`git diff F topic`，这恰好就是提交B和G的累加diff。

参考：
* [How to make git diff show the same result as github's pull request diff?](https://stackoverflow.com/questions/37763836/how-to-make-git-diff-show-the-same-result-as-githubs-pull-request-diff)
* [About comparing branches in pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-comparing-branches-in-pull-requests)
* [Provide a way to view the diff for a pull request branch](https://gitlab.com/tortoisegit/tortoisegit/-/issues/3427)

### 2.3 版本回退
注意：git checkout和git reset命令在指定和不指定文件名时功能不同！

`git reset [<mode>] <commit>`  **将当前分支移动到指定提交**（HEAD仍指向当前分支），暂存区和工作目录中的文件如何修改取决于`<mode>`：
* `--soft`：不更新暂存区和工作目录
* `--mixed`：更新暂存区，不更新工作目录（默认选项）
* `--hard`：更新暂存区和工作目录

这里的`<commit>`除了SHA-1还可用`HEAD^`表示上一个版本，`HEAD~n`表示往上n个版本，完整格式参考`git help gitrevisions`

`git reset [<commit>] -- <file>`  将暂存区中的指定文件替换成仓库中指定提交（默认为HEAD）的内容，不影响工作目录（执行该命令后`git diff --cached <file>`将没有差别），该命令是`git add <file>`的反向操作，即取消暂存

`git checkout -- <file>`  将工作目录中的指定文件替换成暂存区的内容（执行该命令后`git diff <file>`将没有差别），该命令将丢弃本地修改，不可恢复！

`git checkout <commit> -- <file>`  将工作目录和暂存区中的指定文件都替换成仓库中指定提交的内容（执行该命令后`git diff <file>`和`git diff --cached <file>`都将没有差别），该命令将丢弃本地修改，不可恢复！

![基本命令1](https://marklodato.github.io/visual-git-guide/basic-usage.svg)

![基本命令2](https://marklodato.github.io/visual-git-guide/basic-usage-2.svg)

Git 2.23.0引入了一个新的命令git restore，可替代git reset和git checkout进行撤销操作

`git restore [-s <commit> | --source=<commit>] [-S | --staged] [-W | --worktree] <file>`  恢复工作目录或暂存区中的指定文件
* `--staged`和`--worktree`：指定要恢复的位置
  * `--staged`：只恢复暂存区，等价于`git reset <commit> <file>`
  * `--worktree`：只恢复工作目录，等价于`git checkout -- <file>`
  * `--staged --worktree`：恢复暂存区和工作目录，等价于`git checkout <commit> -- <file>`
  * 二者都未指定：只恢复工作目录
* `--source`：指定恢复内容的来源，如果未指定：如果有`--staged`选项则默认为HEAD，否则从暂存区恢复

### 2.4 查看提交历史
`git log [<options>] [<revision-range>] [[--] <path>...]`  查看提交历史

#### 通用选项
* `--abbrev-commit`：只显示SHA-1的前几个字符
* `--relative-date`：显示相对日期
* `--graph`：以图形方式展示
* `--pretty=<format>`：指定每次提交的展示格式，`<format>`可以是`oneline, short, medium, full, fuller, reference, email, raw, format:"<string>"`之一，其中最后一种是自定义格式，可以使用类似于printf的%占位符，详见git log --help的PRETTY FORMATS一节
* `--oneline`：等价于`--pretty=oneline --abbrev-commit`

例如，要获取指定提交的父提交id，可以使用`git log --pretty=%P -1 <commit>`或`git log -1 <commit>^`

#### 限制提交范围选项
* `-n <number>`或`-<number>`：只显示最后n次提交
* `--since=<date>, --after=<date>`：只显示指定日期之后的提交，其中`<date>`可以是类似于"2008-01-15"的绝对日期，也可以是类似于"2 weeks ago"的相对日期
* `--until=<date>, --before=<date>`：只显示指定日期之前的提交
* `--author=<pattern>`：只显示作者姓名匹配指定模式的提交
* `--grep=<pattern>`：只显示提交信息匹配指定模式的提交
* `--all`：显示所有分支的提交

#### path参数
`<path>`参数表示只显示指定的文件发生变化的提交

#### `--pretty`参数常用占位符

| 占位符 | 描述 |
| ----- | ----- |
| `%H` | 提交的哈希值 |
| `%h` | 提交的简短哈希值 |
| `%an` | 作者姓名 |
| `%ae` | 作者邮箱 |
| `%ad` | 作者提交日期 |
| `%ar` | 作者提交相对日期 |
| `%s` | 提交信息标题 |
| `%b` | 提交信息主体 |

#### revision-range参数
`<revision-range>`参数的作用是列出沿着给定提交的父链接能够到达的所有提交，但排除可以从前面带有`^`的提交到达的提交。可以理解为集合操作，每个提交表示从该提交可达的提交集合，结果是所有提交对应集合的并集与前面带有`^`的提交对应集合的差集。

例如，假设有以下提交记录：

```
       C (foo)
      /
A - B - D - E (bar)
      \
       F - G (baz)
```

则命令`git log foo bar ^baz`将列出从foo或bar可达、但从baz不可达的提交，即C、D、E。解释：用S(x)表示从提交x可达的提交集合，则S(foo bar \^baz) = S(foo) ∪ S(bar) - S(baz) = {A, B, C} ∪ {A, B, D, E} - {A, B, F, G} = {C, D, E}

`<commit1>..<commit2>`等价于`^<commit1> <commit2>`，例如`git log foo..bar`等价于`git log ^foo bar`，列出D、E。解释：S(foo..bar) = S(^foo bar) = S(bar) - S(foo) = {A, B, D, E} - {A, B, C} = {D, E}

`<commit1>...<commit2>`的结果是两个提交对应集合的对称差，例如`git log foo...baz`列出C、F、G。解释：S(foo...baz) = S(foo) △ S(baz) = (S(foo) - S(baz)) ∪ (S(baz) - S(foo)) = {C} ∪ {F, G} = {C, F, G}

### 2.5 删除文件
`git rm <file>`  从暂存区和工作目录删除文件

`git rm --cached <file>`  仅从暂存区删除文件

若要仅从工作目录删除文件而不改变暂存区，直接从本地删除文件即可

`git clean [-d] [-f] [<path>...]`  删除所有未跟踪的文件
* `-d`：也删除未跟踪的目录
* `-f`：强制删除
* 如果未指定路径则默认为当前目录

### 2.6 重命名和移动
`git mv <src> <dst>`  将文件src重命名为dst，相当于执行以下三条命令：

```
mv <src> <dst>
git rm <src>
git add <dst>
```

`git mv -r <src> <dst>`  将文件夹src重命名为dst

若要将文件foo.txt移动到bar目录可以使用`git mv foo.txt bar/foo.txt`

## 3.分支管理
`git branch [-r | -a] [-v]`  列出分支
* 无参数：仅列出本地分支
* `-r`：列出远程分支
* `-a`：列出本地和远程分支
* `-v`：显示分支指向的提交
* `-vv`：显示分支指向的提交及跟踪的远程分支

`git branch <branch-name> [<start-point>]`  创建新分支，起始点默认为HEAD

`git branch -m [<old-branch>] <new-branch>`  重命名分支，如果省略`<old-branch>`则默认为当前分支

`git branch (-d | -D) <branch-name>`  删除分支，如果未与其他分支合并则不能删除，但使用参数-D可以强制删除（未合并的提交将会丢失）

`git checkout <branch-name>`  切换到指定分支（改变HEAD指向的分支，不改变分支本身）并更新暂存区和工作目录，如果有未暂存或未提交的修改则git拒绝切换

`git checkout -b <branch-name>  [<start-point>]`  创建并切换到新分支，相当于git branch+git checkout

Git 2.23.0引入了一个新的命令git switch，可替代git checkout进行切换分支操作

`git switch <branch-name>`  切换到指定分支，等价于`git checkout`

`git switch -c <branch-name> [<start-point>]`  创建并切换到新分支，等价于`git checkout -b`

`git switch -`  切换到之前所在的分支

### 合并
`git merge [--no-ff] [-m "<message>"] <branch>`  将指定分支合并到**当前分支**上，如果没有提供提交信息则默认为"Merge branch 'xxx' into xxx"
* 如果当前分支指向的提交是被合并分支指向提交的祖先，则只将当前分支移动到指定的提交，不生成新提交（fast-forward合并），但如果有`--no-ff`选项则生成新提交（使用`--no-ff`合并后的历史有分支信息，而fast-forward合并看不出曾经做过合并）
* 否则进行一次三方合并（当前分支、被合并分支以及二者的公共祖先），在当前分支上生成一次新的提交

典型用法：将开发分支合并到主分支，当前分支应当是主分支

```
git checkout master
git merge new-feat
```

合并冲突：合并时如果两个分支都修改了同一文件就会产生冲突
* `git status`  查看存在冲突的文件
* 手工解决冲突
* `git add <file>`  将解决冲突后的文件重新添加到暂存区，标记为已解决
* `git merge --continue`  继续合并，也可用git commit完成合并
* `git merge --abort`  放弃合并

### 变基
`git rebase <to-branch> [<from-branch>]`  变基(rebase)操作，将`<from-branch>`的提交重新应用到`<to-branch>`上，如果指定了`<from-branch>`则先执行`git checkout <from-branch>`，否则默认为当前分支
* 相当于先将`<from-branch>`上的提交临时保存起来，之后`git reset --hard <to-branch>`，最后依次将提交重新“复制粘贴”到`<from-branch>`
* rebase与merge的区别是可以保持线性的提交历史

典型用法：将开发分支rebase到主分支上，当前分支应当是开发分支

```
git checkout new-feat
git rebase master
```

rebase操作也可能产生合并冲突，解决方法与merge类似
`git rebase --continue`  继续rebase
`git rebase --abort`  放弃rebase

## 4.远程仓库
### 4.1 生成SSH key
使用SSH协议克隆远程仓库和向远程仓库推送时需要关联SSH key，使用HTTP协议则不需要

参考：[Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

在Git Bash中执行以下命令：`ssh-keygen -t rsa -b 4096 -C "your_email@example.com"` 提示输入保存文件位置，直接按回车使用默认位置~/.ssh/id_rsa；提示输入密码，可直接按回车跳过
`eval $(ssh-agent -s)` 运行ssh-agent
`ssh-add ~/.ssh/id_rsa` 将SSH密钥添加到ssh-agent

将SSH key添加到GitHub账户：[Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

查看关联的SSH key：[Reviewing your SSH keys](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/reviewing-your-ssh-keys)

```
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_rsa
ssh-add -l -E md5
```

多个SSH key：同一台机器可以创建多个SSH key（例如需要使用不同的邮箱），在创建SSH key时提示输入文件位置的步骤输入不同的文件名即可（例如~/.ssh/id_rsa_2），此时需要创建配置文件
* 在.ssh目录下创建一个config文件，内容如下：

```
Host github.com
    HostName github.com
    User <your name>
    IdentityFile ~/.ssh/id_rsa_2
    PreferredAuthentications publickey
```

否则会报错git@github.com: Permission denied (publickey)
* 测试：`ssh -T git@github.com`

### 4.2 远程仓库命令
`git clone <url>`  克隆远程仓库

`git remote [-v]`  列出已关联的远程仓库

`git remote add <name> <url>`  添加远程仓库，习惯上名称取origin，GitHub仓库的url格式为git@github.com:{username}/{project-name}.git或https://github.com/{username}/{project-name}.git，需要先在GitHub上创建相应的仓库

`git push [-u] [<remote> [<branch>]]`  将指定分支的提交推送到远程仓库并更新远程分支
* 如果省略分支名则默认为**当前分支**，如果省略远程仓库名则默认为origin
* 如果远程仓库没有对应的分支则使用参数`-u`使本地分支与远程分支关联
* 常用形式：`git push origin foo`  将foo分支的提交推送到远程仓库，更新origin/foo分支

`git fetch [<remote> [<branch>]]`  获取远程仓库指定分支的提交
* 如果省略分支名则获取**所有分支**，如果省略远程仓库名则默认为origin
* 对于新的远程分支origin/foo，该命令不会自动创建对应的本地分支foo，需要手动创建：`git checkout -b foo origin/foo`，这将自动建立foo和origin/foo的关联

`git pull [--rebase] [<remote> [<branch>]]`  获取远程仓库指定分支的提交并与**当前分支**合并
* 如果省略分支名则获取**所有分支**，如果省略远程仓库名则默认为origin
* 相当于git fetch+git merge，若有冲突则需手动解决冲突
* 如果指定了`--rebase`选项则使用rebase合并，即git fetch+git rebase
* 常用形式：`git pull origin foo`  获取远程仓库foo分支的提交，并将当前分支合并到origin/foo（当前分支应当是foo，此时应当是fast-forward合并），相当于

```
git fetch origin foo
git merge origin/foo
```

* 无参数形式：`git pull` 获取远程仓库所有分支，并将当前分支合并到origin/当前分支，如果当前分支没有跟踪远程分支则不合并，相当于

```
git fetch
git merge origin/当前分支
```

### 4.3 远程分支管理
`git branch -vv`  查看本地分支跟踪的远程分支

`git branch [-t | --track] <branch> <remote-branch>`  创建新分支，并跟踪指定的远程分支
* 如果git branch的第二个参数是一个远程分支，则即使不指定-t选项也会自动建立关联
* 例如：`git branch -t foo origin/foo`或`git branch foo origin/foo`将创建foo分支，并使其跟踪远程分支origin/foo

`git checkout -b <branch> (-t | --track) <remote-branch>`  创建新分支，跟踪指定的远程分支，并切换到新分支
* 类似于git branch，如果git checkout的第二个参数是一个远程分支，则可省略`-t`或`-b`选项
* 例如：以下几条命令等价，都将创建foo分支，使其跟踪远程分支origin/foo并切换到foo分支：
  ```
  git checkout -b foo -t origin/foo
  git checkout -b foo origin/foo
  git checkout -t origin/foo
  git checkout foo
  ```
* 如果git checkout切换分支时分支名不存在，但匹配一个远程分支名，则将自动创建一个本地分支并跟踪该远程分支，因此上面最后一条命令和前两条等价

`git branch -u <remote-branch> [<branch>]`  建立本地分支和远程分支的关联，`<branch>`默认为当前分支

`git branch -d -r <remote-branch>`  删除本地的远程分支，不删除本地分支（只有当分支已在远程仓库中不存在时才有意义）
* 例如：如果foo分支已在远程仓库origin中不存在，则`git branch -d -r origin/foo`删除本地的origin/foo分支，不删除foo分支

`git push -d <remote> <branch>`  删除远程仓库的分支和本地的远程分支，不删除本地分支
* 例如：git push -d origin foo删除远程仓库的foo分支和本地的origin/foo分支，不删除本地的foo分支

`git remote prune <remote>`  删除所有远程仓库中已经不存在的远程分支，不删除本地分支

### 4.4 多人协作工作流程
（1）切换到master分支，拉取远程仓库的修改

```
checkout master
git pull
```

* 如果本地有未提交的修改可使用`git stash`：

```
git stash
git pull
git stash pop
```

（2）在本地创建一个新分支（假设名为new-feat），用于开发新功能

```
git checkout -b new-feat
# develop
git commit -m "new feature"
```

* 开发同一个功能最好保持单次提交，如果已经在本地提交，但尚未推送到远程仓库，之后又有其他修改，则可使用`git commit --amend`将修改“合并”到上一次提交

（3）将提交推送到远程仓库

```
git push -u origin new-feat
```

* 如果已经推送到远程仓库，之后又有其他修改，要保持单次提交可强制推送到远程new-feat分支

```
git commit --amend
git push -f origin new-feat
```

* 如果向远程仓库推送时master分支上已经有很多新的提交，则之后合并时可能产生合并冲突，可以先在本地rebase到master分支再推送，从而提前解决合并冲突

```
git fetch origin master
git rebase master
git push origin new-feat
```

（4）在GitHub或GitLab上创建PR、代码评审、合并到master分支

## 5.标签管理
`git tag`  列出所有标签

`git tag <name> [<commit>]`  在指定的提交上创建标签，默认为HEAD

`git tag -d <name>`  删除标签

`git show <tag-name>`  查看标签详细信息

## 6.储藏工作目录
`git stash [push [-m <message>]]`  临时保存工作目录中所有修改并恢复到HEAD指向的提交

`git stash list`  列出所有已储藏的工作目录

`git stash show [<stash>]` 比较`<stash>`与其所在的提交，`<stash>`的格式为`stash@{n}`或`n`，默认为最近一次stash（即`stash@{0}`），下同

`git stash apply [<stash>]`  恢复指定的stash

`git stash drop [<stash>]`  删除指定的stash

`git stash pop [<stash>]`  相当于git stash apply+git stash drop

## 参考
* [Git Reference](https://git-scm.com/docs)
* [Pro Git](https://git-scm.com/book/en/v2)
* [A Visual Git Reference](http://marklodato.github.io/visual-git-guide/index-en.html)
* [Git教程 - 廖雪峰](https://www.liaoxuefeng.com/wiki/896043488029600)
