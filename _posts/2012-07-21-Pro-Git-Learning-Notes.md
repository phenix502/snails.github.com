---
layout:       post
title:        "Git Basic"
description: 
banner: 
categories: 
- git
---

#版本控制概念

版本控制(Reversion Control)是一种记录文件若干文件内容变化，以便将来直接查阅特定版本的系统。

#版本控制系统的发展

##本地版本控制系统

本地控制系统是比较早的版本控制系统，它使用简单的数据库来记录文件的历次更新差异。最流行的为rcs；在许多计算机系统中都还可以砍到它的影子。

##集中化的版本控制系统

本地版本控制系统只是在本地用作版本控制，而对于多人协同工作却无法做到，于是出现了集中化的版本控制系统(Central Version Control System)。此类系统有CVS,Subversion以及Perforce等等。集中版本控制系统都有相应的权限控制部分，对不同的目录分配不同的权限；我们可以看到每个文件的更新历史。它的缺点就是对于版本控制服务器的依赖很强，如果服务器挂掉，那么会导致有些工作无法进行下去，如果访问器磁盘挂掉，那么后果可能是灾难性的。我们现在开发使用的客户端为VSS，日常工作时那个速度..好恶心

##分布式版本控制系统

分布式版本控制系统(Distributed Version Control System)兼容了前面的两种控制系统。代表有Git, Mercurial和Bzzaar、Darcs.它在获取代码时提取原始仓库的快照和最新文件，如果服务器挂掉，那么会有本地的记录来恢复。这样的系统其实是维护本地的仓库和远程的仓库。

#Git的历史

Git的历史也是是在管理Linux内核的历史。在1991-2002年期间，社区人员大存归档的繁琐事务上。在2002年，Linux社区开始和BitKeeper合作，使用BitKeeper来管理和维护代码。在2005系统到期，作为商业软件，社区不能再无偿使用它。不受制于人，只能靠自己努力，拥有自主的东西，于是Linus Torvalds开始设计属于自己的版本控制系统。他们对新的版本控制系统定下了如下的目标:

* 简单
* 速度快
* 对非线性开发的支持（多分支）
* 完全分布式
* 能管理大型软件项目。

#Git与常规版本控制系统的区别

##保存文件的快照，而非比较差异

CVS和Subversion等控制系统，不仅判断文件是否修改，还记录文件更新的哪一部分内容。Git与之大不相同，它直接保存文件的快照，如果文件修改了，那么对文件进行快照；如果没有修改，那么就不保存它。Git只对上次保存的快照作连接。

##几乎所有操作都可以在本地完成

由于Git属于分布式版本控制软件，本地就保存着和服务器的仓库，如果需要对比文件的差异性操作，只用在本地操作就足够了。而传统的集中化版本控制系统都需要从服务器获取要比较的文件与本地或与服务器的某个版本文件进行比较。显然Git操作会更快。

另外Git可以在本地随时进行更新和提交，最后统一push到服务器；对于这一点，在断网的情况下，Subversion和CVS只能本地修改，不能进行提交，这意味着本地只能保留最新近修改的文件，而不能保留中间过程中的文件历史。

##保持数据完整性

Git使用SHA-1算法对所有数据进行校验和计算，并使用该校验和作为该版本的索引。每个SHA-1索引是由40个十六进制的字符构成。对所有文件进行校验和计算，这意味着如果某个文件修改了或丢失了能够迅速判断出来。

##Git的状态

Git的状态有三种：已提交(Commited),已修改(Modified)和已暂存(Staged)。已提交说明文件已经保存在本地数据库中；已修改说明修改了文件的内容，但是未提交；已暂存说明已修改的文件放在下次提交时要保存的清单中了。

Git的工作区域包括：Git本地数据目录、工作目录和暂存区域。

本地文件Checkout后会进入工作目录，修改后的文件会放入暂存区域，提交后的文件会从暂存区域转向本地数据目录。

对于Git来说，最重要的文件是.git目录。它保存着元数据和对象数据库，每次clone镜像仓库时，实际拷贝的就是它。暂存区域实际是简单的文件，放在.git目录下。

##Git配置

Git提供了git config工具来对Git相关的环境变量进行配置，这些配置文件决定了Git在各个环节的具体工作方式。

* /etc/gitconfig文件是对系统内部的所有用户都普遍适用的配置，git config时使用--system选项。
* ~/.gitconfig文件是适用于某个用户的配置，git config 时使用 --global选项。
* 当前项目下的.git/config文件是针对某个项目有效的配置。

这三种配置文件会依次覆盖，从下到上。

###配置用户信息

用户信息是要配置自己的用户名和邮箱，每次Git提交时都会用到这两个信息，用于说明谁提交了更新，并记录历史。

    git config --global user.name "david"
    git config --global user.email example@example.com 

如果没有--global选项，默认在当前项目的.git/config文件下。

###文本编辑器

Git有时候需要额外输入一些信息，这里就用到了文本编辑器。

    git config --global core.editor mvim

###差异分析工具

在解决合并冲突时，使用差异分析工具是很重要的。

    git config --global merge.tool vimdiff

Git这里配置的可以有kdiff3, tkdiff, meld, xxdiff, emerge, vimdiff, gvimdiff等等。

查看所有的配置信息使用git config --list就可以。

#Git基础

##获取项目的Git仓库

获取Git仓库的方法有两种，一是自己创建，二是从远程获取。

在项目目录下执行`git init`，这条命令会初始化一个仓库，这个仓库现在是空的。从远程获得仓库的方法为`git clone https://<URL>`,git clone支持的协议包括git://和http(s)，user@server:/<Path>.git。git工程下面会有一个`.git`目录用来存放Git需要的数据和资源。

如果需要忽略某些文件，则需要编辑Git仓库下面的`.gitignore`文件，将不跟踪的文件列入其中。

    *.[oa]      #忽略以o或a为结尾的文件
    *~          #忽略以~结尾的文件
    tmp/        #忽略tmp目录，这里需要有/
    !lib.a      #排除lib.a，即要跟踪lib.a
    doc/*.txt   #忽略doc目录下的txt文件

##Git常用命令

git获取帮助的方法为

    git <命令> --help
    man git <命令>

1. 查看当前状态

    git status
    # On branch master
    nothing to commit (working directory clean)

2. 跟踪新文件

    git add README

`git add`支持通配符，如果跟踪的是目录，那么它会自动进行迭代跟踪。

3.比较文件变化

    git diff

这条命令会显示尚未暂存的文件更新了哪些部分，默认比较的是当前文件和暂存区域快照直接的差异。如果要查看缓存的文件和上次提交的快照之间的差异，可以使用`git diff --cached`。

4. 提交更新

    git commit -m <message>

将已经暂存的文件提交，-m参数用来对本次提交写comment，将更新内容简要写出来。

5. 如果想从暂存区域将已经跟踪的某个文件移除，可以使用`git rm`命令，此命令也会将文件删除掉。如果该文件已经修改过且已经在暂存区，必须添加强制删除参数`-f`。如果只希望仅仅从仓库跟踪列表中删除，而保留文件，那么需要添加`--cached`参数，即`git rm --cached <文件>`。

>注意：如果使用通配符，注意星号`*`一般最好添加一个反斜杠\进行转义。因为Git有自己的文件模式扩展匹配方式，在上面的星号`*`会自动进行递归。如果不转义，则达不到此效果。

6. 移动文件 
  
    git mv <src> <dest>

此命令运行后，Git并不跟踪文件移动操作。如果在Git中重命名了某个文件，仓库中存储的元数据并不会体现出这是一次改名操作。

7. 查看提交历史

    git log

默认会按提交时间列出所有的更新，最近的更新排在最上面。

git log可以使用`--pretty=format:"<format string>"`来对git log的结果进行美化：

    选项           说明
    %H            提交对象的完整Hash
    %h            提交对象的简短Hash
    %T            树对象的完整Hash 
    %t            树对象的简短hash
    %P            父对象的完整Hash 
    %p            付对象的简短Hash
    %an           作者的名字
    %ae           作者的email 
    %ad           作者修订日期
    %ar           作者修订日期，按多久之前的方式显示
    %cn           提交者的名字
    %ce           提交者的email 
    %cd           提交日期
    %cr           提交日期，按多久之前的方式显示 
    %s            提交说明

>作者和提交者的区别是：作者是实际作出修改的人，提交者是将修改提交到仓库中的人。

git log还支持按时间段对提交记录进行查询：
    
    -(n)最近的n条提交
    --since, --after 显示指定实际自后的提交
    --until, --before 指定时间之前的提交
    --author 指定指定作者的记录
    --committer 显示指定提交者相关的提交

8. 撤销操作

    git commit --amend

此命令将当前的暂存区域快照提交，如果刚才提交为完成任何改动，那么相当于重新编辑提交说明。

    git commit -m 'initial commit'
    git add forgoten_file
    git commit --amend

上面的命令会将第二次提交替换掉第一次提交的内容。

9. 撤销暂存的文件

    git reset HEAD <file>

10. 取消对文件的修改
    
    git checkout <file>

11. 添加远程仓库

    git remote add <local name> git://github.com/example/testcase.git 

    git fetch origin #假设origin为上面配置的local name 

    git push origin master #推送分支master到origin服务器端

12. 删除和重命名远程仓库
    
    git remote rename origin new_origin
    git remote rm new_origin

13. 打标签

    git tag #显示所有标签
    git tag -l 'v1.4' #只显示v1.4的tag

标签有轻量级(lightweight)和含附注的(annotated)。轻量级标签是不会变化的分支，实际上它指向特定提交对象的引用。附注标签实际上是存储仓库中的一个独立对象，有自身的校验和信息，包含着标签的名字，电子邮件和日期以及标签说明。

    git tag -a v1.4.1 -m 'stable version'

    git show v1.4.1 #显式对应标签的信息

    git tag -v [tag-name] #使用共钥验证签名，共钥需要放在keyring中。

如果有自己的私钥，可以使用用GPG来签署标签，

    git tag -s v1.4.2 -m 'signed version'

轻量级标签：
   
    git tag v1.4.3

对于以前的某个版本签标签，需要在标签后面加上提交对象的校验和(前面几位就可以了)

   git tag -a v1.9.0 af82423

如果需要将标签推送到远程服务器，使用`git push origin [tag-name]`，弄人不会将标签传送到远程服务器。推送所有的标签上去，可以使用`--tags`选项。


#分支

Git仓库中有5个对象：表示文件快照内容的blob对象，记录目录树内容及其中各个文件对应blob对象索引的tree对象，指向tree对象的索引和其它提交信息元数据的commit对象。

1.创建分支

    git checkout -b newbranch

2. 切换分支

    git checkout master

3. 合并分支

    git merge <要合并的分支> #合并到当前所在的分支

    合并过程中，如果遇到在两个分支同时修改了相同的文件，那么就需要手工进行修改合并。或者2选1，或者把两者不同部分手动修改。

4. 删除分支
   
    git branch -d <branch_name> #-D表示强制删除

5. 显示分支

    git branch 
    可选的参数：
    -v 表示显示最后一次的commit信息。
    --merged 和 --no-merged表示已经合并和未进行合并的分支。

##分支式工作流程

###长期分支

由于Git使用简单的三方合并（即如果要合并的和当前的分支属于树的两个分支，那么在合并时会取两个分支共同的父节点+当前+要合并的分支进行合并），就算在较长一段时间内，反复多次把某个分支合并到另一分支，也不会很难。此类分支适合完成特定任务，随着开发的推进，将这个分支合并到其它分支。

###特性分支

特性(Topic)分支是指一个短期的，用来实现单一特性或与其相关工作的分支。

##远程分支

如果需要将自己的分支推送到远程仓库，那么可以使用`git push (远程仓库名) (分支名)`命令。

    git push origin mybranch

mybranch分支名会被Git自动扩展为refs/heads/mybranch:refs/heads/mybranch,意为取出本地的mybranch分支，推送到远程的mybranch。也可以使用`git push origin mybranch:newbranch`来实现相同的效果。

从远程服务器同步过来的分支，即使用fetch命令获取的分支只会获取数据，但是本地不会产生新的分支，需要手动checkout:
   
    git checkout -b mybranch origin/mybranch

删除远程分支：

    git push origin :mybranch

它的实际语法是:git push [远程名] [本地分支]:[远程分支],上面命令的意思是提取本地的空分支推送到远程。

合并分支除了使用merge命令，还可以使用rebase命令。对于两个不同的分支如果使用merge，则取两个分支的共同祖先进行三方合并。衍合(rebase)会回到两个分支的共同祖先，提取两个分支每次提交时的不同，把差异分别保存在临时文件里然后从当前分支转换到要衍合的分支，依次序施用每个补丁。

    git checkout tobe_rabased
    git rebase master #rebase the tobe_rabased to the master

#服务器上的Git

Git支持本地传输、SSH协议、Git协议和HTTP协议。

    git clone file:///opt/git/mygit.git 
    git clone /opt/git/mygit.git 
    git clone ssh://user@server:mygit.git 
    git clone user@server:mygit.git #default use the SSH Protocol 

