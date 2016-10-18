---
layout: post
title: Git 常用命令
description: Git 常用命令详解
keywords: git
category: Git
tags: [Git, Command]
---

Git 是一个很强大的分布式版本管理工具，本文介绍git常用的命令。

###1.Git 版本库的初始化

* `git clone`：这是一种较为简单的初始化方式，当你已经有一个远程的Git版本库，只需要在本地克隆一份
例如：`git  clone  git://github.com/someone/some_project.git some_project` 
上面的命令就是将`git://github.com/someone/some_project.git`这个URL地址的远程版本库，完全克隆到本地some_project目录下。

* `git init` 和`git remote`：这种方式稍微复杂一些，当你本地创建了一个工作目录，你可以进入这个目录，使用`git init`命令进行初始化；Git以后就会对该目录下的文件进行版本控制，这时候如果你需要将它放到远程服务器上，可以在远程服务器上创建一个目录，并把可访问的URL记录下来，此时你就可以利用`git remote add`命令来增加一个远程服务器端，
例如：`git  remote  add  origin  git://github.com/someone/another_project.git`
上面的命令就会增加URL地址为`git://github.com/someone/another_project.git`，名称为origin的远程服务器，以后提交代码的时候只需要使用origin别名即可。

<!-- more -->

###2.远程仓库相关命令

* 检出仓库：`$ git clone git://github.com/jquery/jquery.git`

* 查看远程仓库：`$ git remote -v`

* 添加远程仓库：`$ git remote add [name] [url]`

* 删除远程仓库：`$ git remote rm [name]`

* 修改远程仓库：`$ git remote set-url --push [name] [newUrl]`

* 拉取远程仓库：`$ git pull [remoteName] [localBranchName]`

* 推送远程仓库：`$ git push [remoteName] [localBranchName]`

*如果想把本地的某个分支test提交到远程仓库，并作为远程仓库的master分支，或者作为另外一个名叫test的分支，如下：*

* `$git push origin test:master` :提交本地test分支作为远程的master分支

* `$git push origin test:test` :提交本地test分支作为远程的test分支

###3.分支(branch)操作相关命令

* 查看本地分支：`$ git branch`

* 查看远程分支：`$ git branch -r`

* 创建本地分支：`$ git branch [name]` ----注意新分支创建后不会自动切换为当前分支

* 切换分支：`$ git checkout [name]`

* 创建新分支并立即切换到新分支：`$ git checkout -b [name]`

* 删除分支：`$ git branch -d [name]` ---- -d选项只能删除已经参与了合并的分支，对于未有合并的分支是无法删除的。如果想强制删除一个分支，可以使用-D选项

* 合并分支：`$ git merge [name]` ----将名称为[name]的分支与当前分支合并

* 创建远程分支(本地分支push到远程)：`$ git push origin [name]`

* 删除远程分支：`$ git push origin :heads/[name]` 或 `$ gitpush origin :[name]` 

* 创建空的分支：(执行命令之前记得先提交你当前分支的修改，否则会被强制删干净没得后悔)
{% highlight sh %}
$git symbolic-ref HEAD refs/heads/[name]
$rm .git/index
$git clean -fdx
{% endhighlight %}

###4.tag操作相关命令

* 查看：`$ git tag`

* 创建：`$ git tag [name]`

* 删除：`$ git tag -d [name]`

* 查看远程tag：`$ git tag -r`

* 创建远程tag(本地版本push到远程)：`$ git push origin [name]`

* 删除远程tag：`$ git push origin :refs/tags/[name]`

* 合并远程仓库的tag到本地：`$ git pull origin --tags`

* 上传本地tag到远程仓库：`$ git push origin --tags`

* 创建带注释的tag：`$ git tag -a [name] -m 'comments'`

###5.子模块(submodule)相关操作命令

* 添加子模块：`$ git submodule add [url] [path]`
如：`$git submodule add git://github.com/soberh/ui-libs.git src/main/webapp/ui-libs`

* 初始化子模块：`$ git submodule init`  ----只在首次检出仓库时运行一次就行

* 更新子模块：`$ git submodule update` ----每次更新或切换分支后都需要运行一下

* 删除子模块：
1) `$ git rm --cached [path]`
2) 编辑“.gitmodules”文件，将子模块的相关配置节点删除掉
3) 编辑“ .git/config”文件，将子模块的相关配置节点删除掉
4) 手动删除子模块残留的目录


###6.忽略一些文件、文件夹不提交

在仓库根目录下创建名称为`.gitignore`的文件，写入不需要的文件夹名或文件，每个元素占一行即可。


##Git常用命令详解

* `git pull`：从其他的版本库（既可以是远程的也可以是本地的）将代码更新到本地，例如：`git pull origin master`就是将origin这个版本库的代码更新到本地的master主枝，该功能类似于SVN的update

* `git add`：是将当前更改或者新增的文件加入到Git的索引中，加入到Git的索引中就表示记入了版本历史中，这也是提交之前所需要执行的一步，例如`git add app/model/user.rb`就会增加app/model/user.rb文件到Git的索引中，该功能类似于SVN的add

* `git rm`：从当前的工作空间中和索引中删除文件，例如`git rm app/model/user.rb`，该功能类似于SVN的rm、del

* `git commit`：提交当前工作空间的修改内容，类似于SVN的commit命令，例如`git commit -m story #3, add user model`，提交的时候必须用-m来输入一条提交信息，该功能类似于SVN的commit

* `git push`：将本地commit的代码更新到远程版本库中，例如'git push origin'就会将本地的代码更新到名为orgin的远程版本库中

* `git log`：查看历史日志，该功能类似于SVN的log

* `git revert`：还原一个版本的修改，必须提供一个具体的Git版本号，例如`git revert bbaf6fb5060b4875b18ff9ff637ce118256d6f20`，Git的版本号都是生成的一个哈希值

* `git branch`：对分支的增、删、查等操作，例如`git branch new_branch`会从当前的工作版本创建一个叫做new_branch的新分支，`git branch -D new_branch`就会强制删除叫做new_branch的分支，`git branch`就会列出本地所有的分支

* `git checkout`：Git的checkout有两个作用，其一是在不同的branch之间进行切换，例如`git checkout new_branch`就会切换到new_branch的分支上去；另一个功能是还原代码的作用，例如`git checkout app/model/user.rb`就会将user.rb文件从上一个已提交的版本中更新回来，未提交的内容全部会回滚

* `git reset`：将当前的工作目录完全回滚到指定的版本号，如`git reset bbaf6fb5060b4875b18ff9ff637ce118256d6f20`

* `git rebase`：使branch同步到master最新版本

* `git stash`：将当前未提交的工作存入Git工作栈中，时机成熟的时候再应用回来

* `git config`：利用这个命令可以新增、更改Git的各种设置，例如`git config branch.master.remote origin`就将master的远程版本库设置为别名叫做origin版本库

* `git tag`：可以将某个具体的版本打上一个标签，这样你就不需要记忆复杂的版本号哈希值了，例如你可以使用`git tag revert_version bbaf6fb5060b4875b18ff9ff637ce118256d6f20`来标记这个被你还原的版本，那么以后你想查看该版本时，就可以使用revert_version标签名，而不是哈希值
