# git

## 安装

官网上提供了各种操作系统的安装方式，写的很详尽，就不赘述了。[安装传送门](http://git-scm.com/downloads)

## 基本命令

### 配置

系统级别配置参数
	
	--system

用户全局配置参数

	--global
	
单独项目配置参数

	--local
	
常用配置
	
	git config --global user.name "xxxx" #用户名
	git config --global user.email "xxxx@xxx.com" #邮箱
	git config --global core.editor vim #编辑器
	git config --global alias.st status #按这种方法，配置别名
	git config -l #列举所有配置
	

### 初始化仓库

新建一个目录，该目录下执行init命令，创建本地仓库

	mkdir Storage
	cd Storage
	git init
	
假如需要创建远程仓库，加上bare参数
	
	git --bare init

### 克隆仓库

获取一个本地仓库的克隆
	
	git clone <仓库位置> <克隆名称>
	
假如有一个路径为/Users/zhaochen/works/study/git/test的本地仓库test，获取它的克隆，并起名为newTest，命令如下

	git clone /Users/zhaochen/works/study/git/test newTest
	
上述使用的是绝对路径，也可以使用相对路径	
	
克隆远程服务器上的仓库

	git clone username@host:/path/to/repository
	
例如，克隆一下pm2在gitHub上的仓库	
	
	git clone https://github.com/Unitech/pm2.git pm2
	
在clone的时候添加bare参数，那么克隆为远程仓库

	git clone --bare <源路径>
	
### 工作流

每一个本地仓库都维护着3棵“树”：

* 工作目录，持有实际文件
* 暂存区，又叫Index，缓存着你的改动
* HEAD，它指向你的最后一次提交

### 本地改动提交到暂存区

使用add命令将你作出的改动提交到暂存区
	
	git add <filename> //添加单个文件
	git add * //添加所有更改
	
### 将暂存区的内容提交到HEAD

	git commit -m "代码提交描述"
	
还有一个快捷方式

	git commit -a -m "代码提交描述"
	
使用上述命令即可不需要在之前使用git add或者git rm命令。
	
### 推送改动

本地改动添加到HEAD中，执行如下命令以将这些改动提交到远端仓库：

	git push origin master
	
可以把 master 换成你想要推送的任何分支。 

如果你还没有克隆现有仓库，并欲将你的仓库连接到某个远程服务器，你可以使用如下命令添加：

	git remote add origin <server>
	
如此你就能够将你的改动推送到所添加的服务器上去了。

#### 版本恢复reset
从暂存区恢复到工作区，即将已经add进暂存区的改动从暂存区撤销

	git reset <file> 
	git reset .  
彻底恢复到某个版本，可以通过git log获取版本id

	git reset --hard $id
	
如果不小心使用了错误的HEAD重置，会发现HEAD指向了重置的版本id，该版本之后的版本提交都不见了，使用git log也无法找到，使用如下命令显示所有的版本记录，master可以换成任意分支

	git reflog show master | head

### 分支

在你创建仓库的时候，master 是“默认的”分支。
	
	git branch //查看本地所有分支
	git branch -r //查看远程分支 
	
	git branch branch_name //新建一个名为branch_name的分支
	git checkout -b branch_name //新建一个名为branch_name的分支并切换过去
	
	git checkout branch_name //切换到名为branch_name的分支
	
	git branch -d branch_name //删除分支
	git branch -D branch_name //强制删除分支
	git push origin :branch-name //删除远程分支	
		
### 合并与更新

从远程仓库更新你的本地仓库

	git pull

指定分支更新

	git pull origin branch_name
	
同步远程仓库和本地仓库，尤其是当远程有新的分支，本地要获取

	git fetch
		
要合并其他分支到你的当前分支

	git merge branch_name
	
使用该命令后，git后自动去合并。遗憾的是，这可能并非每次都成功，并可能出现冲突（conflicts）。使用diff命令可以查看哪些文件的哪些地方有冲突。

	git diff 	
	
手动解决这些冲突之后，你需要执行如下命令以将它们标记为合并成功：
	
	git add filename //filename是产生冲突的文件
	
提交本地改动到远程

	git push //提交所有分支
	git push origin branch_name //提交指定分支
	git push origin branch_name //将本地新建的分支push到远程仓库
	git push -f origin branch_name //使用本地分支内容强制替换远程分支	
### 暂存管理
	
	git stash //撤销工作区的改动，并将这些改动暂存到一个堆栈中
	git stash list //查看堆栈中的缓存
	git stash apply $id //将对应id的缓存恢复到工作区，栈中仍存储该改动
	git stash pop //将栈顶的改动恢复到工作区，栈中移除该改动
	git stash clear #清空暂存栈
### 标签

为软件发布创建标签是推荐的。这个概念早已存在，在 SVN 中也有。你可以执行如下命令创建一个叫做 1.0.0 的标签：			

	git tag 1.0.0 1b2e1d63ff
	
1b2e1d63ff 是你想要标记的提交 ID 的前 10 位字符。可以使用下列命令获取提交 ID：

	git log
	
### 替换本地改动

假如你操作失误（当然，这最好永远不要发生），你可以使用如下命令替换掉本地改动：

	git checkout -- <filename>
	
此命令会使用 HEAD 中的最新内容替换掉你的工作目录中的文件。已添加到暂存区的改动以及新文件都不会受到影响。

假如你想丢弃你在本地的所有改动与提交，可以到服务器上获取最新的版本历史，并将你本地主分支指向它：

	git fetch origin
	git reset --hard origin/master
	
### 远程
	
	git remote -v //查看远程服务器地址和仓库名称
	git remote show origin //查看远程服务器仓库状态
	git remote add origin <远程地址> //添加远程仓库地址
	git remote set-url origin <远程地址> //修改远程地址
	git remote rm //删除远程创库地址
	
### 实用小贴士

内建的图形化 git：

	gitk
	
彩色的 git 输出：

	 git config color.ui true
	 
### 原文地址
本文大部分例子来源于下面文章

* [常用git命令汇总](http://jianshu.io/p/0f2ffa404ac1)

* [the simple guida to git](http://rogerdudler.github.io/git-guide/index.zh.html) 	 
	 
### 更多参考资料

* [社区参考书](http://rogerdudler.github.io/git-guide/index.zh.html)
