---
title: git
tags:
  - git
categories:
  - 其他
date: 2018-08-18 15:52:08
---

> 工作中经常会用到 git,都只是停留在简单的操作上，对于出现的问题也只会百度解决，无法知其所以然，所以准备好好了解一下 git，做到知其所以然。以下为阅读《progit》做的一些笔记，方便后期查阅。

# Git

## 起步

### 同其他版本控制系统的差异

- Git 存储的是项目随时间改变的快照而不是与初始版本的差异
  - Git 存储的是每个改变点项目整体快照
    ![](diff_git.png)
    - 如果某文件没有被修改，Git 不会重新存储该文件
  - 其他常见版本控制系统存储的是每个文件与初始版本的差异
    ![](diff_cvs.png)
- Git 近乎所有操作都是本地执行
  - 速度快
  - 可以离线操作
- Git 保证完整性
  - 用 SHA-1 散列（hash，哈希）来计算校验和，这意味着不可能在 Git 不知情的情况下更改任何文件内容和目录
  - SHA-1 散列长这样
  ```
  24b9da6552252987aa493b52f8696cd6d3b00373
  ```

### Git 管理的文件的三种状态

- 通过 Git 管理的文件会处于下面三种状态之一
  - 已提交(committed)
    - 已提交表示数据已经安全的保存在本地数据库中
  - 已修改(modified)
    - 已修改表示修改了文件，但还没保存到数据库中
  - 已暂存(staged)
    - 已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

### Git 的三个工作区域

- Git 仓库目录(.git 文件夹)
  - Git 仓库目录是 Git 用来保存项目的元数据和对象数据库的地方。这是 Git 中最重要的部分，从其它计算机克隆仓库时，拷贝的就是这里的数据
- 工作目录(当前目录)
  - 工作目录是对项目的某个版本独立提取出来的内容。这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改
- 暂存区域(一个保存了下次提交信息的文件)
  - 暂存区域是一个文件，保存了下次将提交的文件列表信息，一般在 Git 仓库目录中。别名`索引`

### 基本的 Git 工作流程

1. 在工作目录中修改文件
2. 暂存文件，将文件的快照放入暂存区域
3. 提交更新，找到暂存区域的文件，将快照永久性存储到 Git 仓库目录
   ![](git_directory.png)

### Git 的三个存储配置地方

- `/etc/gitconfig` 系统通用配置，针对系统中的**每个用户**都有效；
  - 可用带有`--system`选项的`git config`命令让 Git 读取此配置
  - win 系统下，此配置文件在 Git 安装目录中
- `~/.gitconfig`或`~/.config/git/config`，此配置仅针对**当前用户**有效；
  - 可用带有`--global`选项的`git config`命令让 Git 读取此配置。
  - win 系统下，此文件保存在`c/users/用户/.config`文件中
- 当前仓库下面的`.git/config`，此配置仅针对当前仓库有效。
  - 可用带有`--local`选项的`git config`命令让 Git 读取此配置
  - 在某仓库下直接`git config`相当于`git config --local`
- 三种配置的优先级： **仓库>当前用户>通用配置**

### 设置用户信息

- 针对当前用户配置基本用户信息

```
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

- `--global`只需运行一次

### 检查配置信息

```
git config --list
```

- Git 会列出所有配置项

## Git 基础

### 获取 Git 仓库

- 在现有项目或目录下导入所有文件到 Git 中
  ```
  cd project/
  git init
  ```
- 从一个服务器克隆一个现有的 Git 仓库
  ```
  git clone [url]
  ```
  - url 支持下面几种类型协议
    - `https`
    - `git://`
    - `ssh`

### Git 文件状态的生命周期

- 工作目录下的文件
  - 未使用 Git 追踪的 (`Untracked`)文件
    - 使用 add 操作，状态会变成`Staged`
  - 使用 Git 追踪的(`Tracked`)文件
    - 未修改过的(`Unmodified`)
      - 文件编辑后状态会变成`Modified`
      - 删除文件后，状态会变成`Untracked`
    - 修改过的(`Modified`)
      - `Modified`状态的文件使用暂存操作，状态会变成(`Staged`)
    - 暂存过的(`Staged`)
      - 使用 commit 操作后，状态会变成`Unmodified`

![](git_process.png)

### 查看当前文件状态(git status)

- `git status`
- `git status -s`或者`git status --short`
  - 使用紧凑格式展示状态
  ```
  $ git status -s
  M README
  MM Rakefile
  A  lib/git.rb
  M  lib/simplegit.rb
  ?? LICENSE.txt
  ```
  - `??`代表未追踪
  - `A`代表已经暂存
  - `空格M`代表文件已经修改过了，但还没有放入暂存区中
  - `M空格`代表文件已经修改过了，并且已经放入暂存区中
  - `MM`代表在工作区被修改并提交到暂存区后又在工作区中被修改了

### 暂存已修改文件/将未追踪文件添加到追踪列表中(git add)

- `git add`
  - 可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等
  - 本质:“添加内容到下一次提交中”->将文件添加到暂存区中，用于下次提交

### 忽略文件(.gitignore)

- 添加`.gitignore`文件
  - 以`#`开头的行代表注释
  - 可以使用 glob 模式(正则)匹配。
    - `[abc]`代表匹配 abc 其中一个字符
    - `a/**/z`代表`a/z`、`a/b/z`、`a/b/c/z`
  - 以（`/`）开头防止递归。
  - 以（`/`）结尾指定目录。
  - 可以在模式前加上惊叹号（`!`）取反。

```
# 忽略任何.a文件
*.a
# 不忽略lib.a文件
!lib.a
# 仅仅忽略当前目录下下的TODO文件，不忽略当前目录子目录下面的TODO文件
/TODO
# 忽略build目录下面的所有文件
build/
# 忽略doc目录下的.txt文件，但不忽略doc子目录下的.txt文件
doc/*.txt
# 忽略doc所有目录下的所有.pdf文件
doc/**/*.pdf
```

### 查看差异(git diff)

- 查看`当前工作目录`尚未暂存的文件更新了什么->`当前工作目录`文件和`暂存区域中快照`文件的差异
  - `git diff`
    - 比较的是工作目录中当前文件和暂存区域快照之间的差异，也就是修改之后还没有暂存起来的变化内容
    - git diff 本身只显示尚未暂存的改动，而不是自上次提交以来所做的所有改动。所以有时候你一下子暂存了所有更新过的文件后，运行 git diff 后却什么也没有，就是这个原因
- 查看`当前工作目录`已经暂存的文件更新了什么，`暂存区域`与你`最后提交`之间的差异->`暂存区域快照`和`最后一次提交之间的差异`
  - `git diff --cached`和`git diff --staged`(二者是同一个意思)

### 提交更新(git commit)

- `git commit`
  - 提交一次更新(暂存区的一次快照)到本地仓库中
    - 提交的是放在暂存区域的快照
    - 任何还未暂存的仍然保持已修改状态，可以在下次提交时纳入版本管理
    - 每一次运行提交操作，都是对你项目作一次快照，以后可以回到这个状态，或者进行比较
- `git commit -m "[fix] - 修复一个bug"`
  - 添加提交说明

### 跳过暂存区域直接提交更新(git commit -a)

- `git commit -a`
  - Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 `git add` 步骤

### 移除文件(git rm)

- `git rm <file>`
  - 从暂存区和工作区中移除某一文件
- `git rm --cached <file>`
  - 只从暂存区中移除某一文件，但会保留工作区的文件

### 移动文件(git mv)

- Git 不会显示追踪文件的移动，文件移动可以分解成多个动作
- `git mv README.md README`
  - 等价于下面三个动作
  ```
  $ mv README.md README
  $ git rm README.md
  $ git add README
  ```

### 查看提交历史(git log)

- `git log`
  - 会列出所有更新(提交)。最近的更新在最上面
- `git log --stat`
  - 会附带每次提交简略的统计信息
- `git log --pretty=oneline`
  - 将每个提交放在一行，方便查看
- 自定义 pretty 格式
  - `git log --pretty=format:"%h - %an, %ar : %s"`
  - 常用选项
  ```
  %H 提交对象（commit）的完整哈希字串
  %h 提交对象的简短哈希字串
  %T 树对象（tree）的完整哈希字串
  %t 树对象的简短哈希字串
  %P 父对象（parent）的完整哈希字串
  %p 父对象的简短哈希字串
  %an 作者（author）的名字
  %ae 作者的电子邮件地址
  %ad 作者修订日期（可以用 --date= 选项定制格式）
  %ar 作者修订日期，按多久以前的方式显示
  %cn 提交者(committer)的名字
  %ce 提交者的电子邮件地址
  %cd 提交日期
  %cr 提交日期，按多久以前的方式显示
  %s 提交说明
  ```
- 使用图表形式查看
  - `git log --graph`
- 限制条件

  - `git log -2` -> 显示最近 2 个提交
  - `git log --author="xxx"` -> 查找对应作者的提交
  - `git log --committer="xxx"` -> 查找对应提交者的提交
  - `git log --grep="xxx"` -> 查找有对应关键词的提交
  - `git log -Sstringname` -> 查找删除/添加了 `stringname` 字符串的提交
  - `git log --since="2018-08-22 21:43"` -> 特定时间点之后的提交
  - 常见限制选项

  ```
  -(n) 仅显示最近的 n 条提交
  --since, --after 仅显示指定时间之后的提交。
  --until, --before 仅显示指定时间之前的提交。
  --author 仅显示指定作者相关的提交。
  --committer 仅显示指定提交者相关的提交。
  --grep 仅显示含指定关键字的提交
  -S 仅显示添加或移除了某个关键字的提交
  ```

### 撤销操作

- Git 撤销是不可逆的
- 重新修改(覆盖)上次`commit`
  - `git commit --amend`
    - 可以修改上次提交时的 message 并重新提交(覆盖之前提交)，如果在此命令之前执行了添加文件到暂存区的操作，则会在此次提交时带上新添加到暂存区中的文件。
    ```
    git commit -m 'inital'
    git add test.txt
    git commit --amend
    ```
    - 则 test.txt 也会被包含到这次提交中
- 将某文件从暂存区中移出
  - `git reset HEAD <file>`
  ```
  git add test.txt
  git reset HEAD test.txt
  ```
- 撤销工作区中对文件的修改(放弃所有更改)
  - `git checkout -- <file>`
  ```
  echo test > test.txt
  git checkout -- test.txt
  ```
  - 本质是拿之前一个版本的文件来覆盖当前文件

## 远程仓库的使用

### 查看已经配置的远程仓库(git remote)

- 使用`git remote`
  - 查看所有已经配置的远程仓库的简写；默认名为`origin`
- 使用`git remote -v`
  - 查看所有远程仓库的简写及对应 url
  - 如果你使用 `clone` 命令克隆了一个仓库，命令会自动将其添加为远程仓库并默认以 `origin`为简写

### 查看远程仓库详细信息

- 使用`git remote show origin`
  - 展示远程仓库`origin`的详细信息
  ```
  $ git remote show origin
  * remote origin
  Fetch URL: https://github.com/schacon/ticgit
  Push URL: https://github.com/schacon/ticgit
  HEAD branch: master
  Remote branches:
  master tracked
  dev-branch tracked
  Local branch configured for 'git pull':
  master merges with remote master
  Local ref configured for 'git push':
  master pushes to master (up to date)
  ```
  - 会告诉你当前处于 master 分支，以及`git pull`和`git push`时关联的远程分支

### 添加远程仓库(git remote add)

- `git remote add <shortname> <url>`
  - 添加一个仓库地址为`url`,简写名为`shortname`的仓库

### 从远程仓库中抓取与拉取

- `git fetch [remote-repo-short-name]`
  - `fetch`仅仅会访问远程仓库，从中拉取所有你还没有的数据(远程仓库中本地还没有的数据)，并不会跟当前工作区文件进行合并或修改他们。仅仅是拉取下来供你查看
  - 必须注意 `git fetch` 命令会将数据拉取到你的本地仓库 - 它并不会自动合并或修改你当前的工作。当准备好时你必须手动将其合并入你的工作。
  -

### 推送到远程仓库(git push)

- `git push [remote-repo-short-name] [remote-branch-name]`
  - 推送本地当前分支到远程的特定分支上
  - 默认情况下省略`remote-repo-short-name`和`remote-branch-name`时，则默认推送到`origin`的`master`分支上
  ```
  git push
  // 等价于
  git push origin master
  ```

### 修改远程分支的本地引用名

- 使用`git remote rename <oldName> <newName>`

```
$ git remote rename pb paul
$ git remote
origin
paul
```

### 切断本地仓库和远程仓库之间的关联

- 使用`git remote rm <remote-repo-short-name>`

```
$ git remote rm paul
$ git remote
origin
```

## 标签

### 查看所有标签

- 使用`git tag`
  - 会列出所有`tag`名
- 使用正则
  - `git tag -l 'v1.8.*'`
  ```
  $ git tag -l 'v1.8.5*'
  v1.8.5
  v1.8.5-rc0
  v1.8.5-rc1
  v1.8.5-rc2
  v1.8.5-rc3
  v1.8.5.1
  v1.8.5.2
  v1.8.5.3
  v1.8.5.4
  v1.8.5.5
  ```

### 创建标签

- 使用`git tag -a <tagName> -m [messages]`

```
$ git tag -a v1.4 -m 'my version 1.4'
$ git tag
v0.1
v1.3
v1.4
```

- 查看某一 `tag` 详细信息
  - 使用`git show <tagName>`
  ```
  $ git show v1.4
  tag v1.4
  Tagger: Ben Straub <ben@straub.cc>
  Date: Sat May 3 20:19:12 2014 -0700
  my version 1.4
  commit ca82a6dff817ec66f44342007202690a93763949
  Author: Scott Chacon <schacon@gee-mail.com>
  Date: Mon Mar 17 21:52:11 2008 -0700
  changed the version number
  ```
- 针对某次提交打 tag
  - 使用`git tag -a <tagName> <commitHash>`
  ```
  // 针对9fceb02版本打tag
  $ git tag -a v1.2 9fceb02
  ```

### 推送 tag 到远程仓库

- 使用`git push <remote-repo-short-name> [local-tag-name]`
  - 类似推送分支到远程仓库
  ```
  // 推送本地的v1.5tag到远程的origin仓库
  $ git push origin v1.5
  ```
- 一次推送多个 tag 到远程仓库
  - 使用`git push <remote-repo-short-name> --tags`
  ```
  // 把所有不在远程仓库服务器上的标签全部传送到那里
  $ git push origin --tags
  ```

### 检出某一标签

- 使用`git checkout -b [branchname] [tagname]`
  - 相当于重新开了一个分支

```
$ git checkout -b version2 v2.0.0
Switched to a new branch 'version2'
```

### 给 Git 命令起别名

- 通过配置`config`文件

```
// 可以用git co代替git checkout
$ git config --global alias.co checkout
// 用git br 代替 git branch
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status
```

## Git 分支

### 未完待续
