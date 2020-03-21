---
title: 分工与合作——Git和Github使用教程
date: 2020-04-01 00:00:00
categories:
- 教程
tags:
- Git
- Github
toc: true
---
# 前言

是否有遇到过写着写着想回到之前版本，却又不记得具体实现；又或是想和队友共享代码，每次修改发送文件；抑或是想换个电脑写却不想用email等搬运全部文件这类烦不胜烦类似的问题，伟大的程序员们自然早就为我们造好了轮子，那就是版本控制的利器——Git以及cloud based Github。本教程也主要讲Git与Github的使用。



# 1. 背景介绍

本来想粗暴写一下安装使用教程，想了想还是先写一点背景介绍，不感兴趣可以直接跳过。



## 版本控制

首先我们需要了解一个概念——Version Control，也就是版本控制。当我们写代码的时候总会有意无意制造出一些bug，有时候我们会想返回前一次没有问题的时候，因此这就是我们为什么需要版本控制。

简单来说，版本控制就是在不同时间节点保存你的程序，然后你可以通过它回看甚至回到之前保存的版本。



## 什么是Git

Git是在2005年的时候初次开发出来的版本控制利器，并风靡全球。Git是一个安装并管理本地系统的工具而且可以给你提供现有文件的你保存的不同版本。因为是本地的，下载之后就不再需要网络也可以使用。



## 什么是Github

Github有一点像可视化的Git，并且是在线的服务。让你可以在线管理你的Git仓库（这个具体会在后面讲）。通过Github，你可以分享你的程序，让其他合作者一起进行编辑。它不仅保存了Git的全部功能，还进行了扩充，它可以让你在任意电脑任意地区访问，只要你有权限。它最大的优点就是它是一个很庞大的数据库，你可以搜索，阅读甚至使用别人写好的程序。当然Github还有很多替代物如Gitlab之类，本文就不赘述了。



## Git vs. Github

简单的说，Git是一个帮助你管理和追踪本地源码历史的版本控制工具，而Github则是一个基于云端运用Git技术的让你管理Git仓库的服务。



# 2. Git

首先，我们来康康Git的使用。

*注意：所有和Git相关的命令都是`git`开头噢。*



## 安装

如果已经安装过，可以跳过。

**Linux**

如果是Fedora或其他类似的 RPM-based distribution, 譬如RHEL 和 CentOS：

```shell
$ sudo dnf install git-all
```

或Debian-based distribution像Ubuntu

```shell
$ sudo apt install git-all
```



**macOS**

可以先试

```shell
$ git --version
```

如果没有安装，应该会弹出安装请求，也可以去https://git-scm.com/download/mac下载。



**Windows**

这个稍微有点烦，见https://git-scm.com/book/en/v2/Getting-Started-Installing-Git。



## 新建一个Git仓库

安装好之后我们就可以开始用了，Git的一个仓库（Repo）就是你的一个项目。我们首先新建一个文件夹

```shell
$ mkdir firstRepo
$ cd firstRepo
```

初始化一个仓库

```shell
$ git init
```

这时候会显示这么一行

`Initialized empty Git repository in /Users/tina/Desktop/firstRepo/.git/`, 

后面一部分是你当前仓库的路径，会根据路径不同进行变化。通过`.git`我们可以知道Git其实是创建了一个隐藏文件夹，所以我们运行`ls`这个命令，并不会看到有关Git的信息。



## 添加文件

我们依然先在当前目录建文件，文件类型无所谓。这里就用`txt`文件了。

```shell
$ touch first.txt
$ ls
first.txt
```

此时我们可以看见文件里有一个文件叫`first.txt`，但是这个文件并不在我们Git仓库里（划重点），为了看仓库里有什么，我们使用

```shell
$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	first.txt

nothing added to commit but untracked files present (use "git add" to track)
```

这个命令我们之后具体讲，现在我们看到`untracked files`这里，这个的意思就是说这些个文件在当前目录下但是没有保存到我们仓库里，所以Git不会对它的改变进行追踪。把文件添加到仓库里，我们使用

```shell
$ git add first.txt
```

此时`first.txt`已经被加进去了。如果我们想添加多个文件呢，可以把想加入的文件用空格隔开，写在后面，像这样

```shell
$ git add file1 file2  
```

不过我们还有一个更简单的方法，那就是`.`

```shell
$ git add .
```

这样可以把当前目录下全部的没有track的文件都加进来（是不是很方便！）



## 删除文件

提到添加就不得不说删除，如果一个文件不想被跟踪了怎么办呢，那就删掉啦。

```shell
$ git rm file
```

*注意：这个操作不会在当前目录删除文件*

一个比较重要的flag

`--force`

~~顾名思义~~, 这个flag是用来强制删除文件的。一般来说，如果你的文件没有被commit（下面就讲这个），是不可以被删除的，但是加上这个flag就可以删除了。



## “截图”

~~咳，实话实话，我也不知道commit怎么准确翻译成中文。~~现在我们来看Git非常非常重要的一个命令，`commit`。它其实就是类似一个截图，存下来你当前的项目完成情况并保存下来，以后可以回顾甚至回到当前节点。至于怎么回到我们后面再讲，现在就来进行“截图”。这个截图只是对于Git追踪的文件，所以它与add是不可分割的。

```shell
$ git commit -m "Your message about the commit"
[master (root-commit) b345d9a] This is my first commit!
 1 file changed, 1 insertion(+)
 create mode 100644 first.txt
```

这样就建了一个新的commit啦，看到这个`b345d9a`东西了么，这个是一个`commit id`。但是如果你对文件进行了修改，又运行了这个语句，你会看到这么一行`Already Up to date`. Bug？Nonono， 这就是需要我们`add`来参与了。因为在Git心里，你的文件还是上一次`add`来的，他一看，文件和上一次`commit`的没有变化呀，不截图不截图。所以我们要重新`add`一遍，这个时候的`add`就是一个更新的操作了，我们每次`commit`之前都要先`add`再commit。

不过，这两步某些情况下是可以合并的，变成

```shell
$ git commit -am "Your message about the commit"
```

这样做就是更新我跟踪的所有文件并进行现有成果截图，但是如果你新建的文件还是要用单独的`add`来进行添加哦。

*注意：这个语句千万不要瞎写哦，以后你可能会用到*



## 分支

虽然我一直~~刻意~~没有提到上面出现过几次的一个词`master`，它是什么呢，它是我们的主分支。想象一棵树，它就是我们的树干。分支的作用是当你想修改某个部分的代码，添加新功能，但却不想影响之前写好的代码，就可以分出一个`branch`，在上面进行。不同分支之间不会相互干扰，除非你进行合并等操作。

我们初始化一个Git仓库时，主分支就已经存在，在此基础上，我们可以新建自己的branch。使用

```shell
$ git branch branchName
$ git branch
* master
  branchName
```

第一个命令是新建，第二个是列出全部分支。*表示当前所在分支。切换分支我们使用

```shell
$ git checkout branchName
$ git branch
  master
* branchName
```

通常来说，我们新建一个分支就是为了切换到新分支，所以我们可以把上两步合二为一

```shell
$ git checkout -b <my branch name>
```

*注意：我们新建分支的内容是和你建分支时所在的一致，要注意新建分支时所在的branch哦，通常情况我们都是在master上加分支。*

在进行写代码，修改之后，我们想把分支上内容合并至master，使用

```shell
$ git checkout master
$ git merge branchName
```

这里稍微有一些饶人，我们需要在master上进行`merge`操作来使得master与分支一致。现在我们来简单讲解下`merge`。

假设在master上我们有几次commit后，新建一个分支A，几次commit之后，我们想把A合并到master上。合并之前，

common base—— — commit —— commit (master)

​								｜— commit —— commit (A)

合并后

common base—— — commit —— commit (master) —— new merge commit

​								｜— commit —— commit (A) ——｜

这个时候master上就有了A上面的内容啦。

merge分为两种，一种是`fast forward`, 另外一种是`3-way merge`。

第一种很直接，合并前

common base——                                              (master)

​								｜— commit —— commit (A)

合并后

common base——                                                            —— new merge commit

​								｜— commit —— commit (A) ——｜

就不细讲Git原理，康图。

第二种其实就是第一个那个图。~~小朋友，你是否有很多问号，为什么就这样合并起来，而没有冲突~~。merge的过程很容易产生的问题就是冲突！尽管Git的merge已经很nb了，但依然不可避免产生冲突，但这些我们可以很容易地解决。



## 冲突

这是Git里经常遇到并且要解决的问题！当你merge不同分支时，很容易遇到。为了让大家直观感受，我们创造一个冲突。

```shell
$ git checkout master
$ echo 'this is conflicted text from master' > first.txt
$ git commit -am 'added one line'
[master 8cc7111] added one line
 1 file changed, 1 insertion(+)
$ git checkout branchName
Switched to branch 'branchName'
$ echo 'this is conflicted text from feature branch' > first.txt
$ git commit -am 'added one line'
[a 23f5790] added one line
 1 file changed, 1 insertion(+)
$ git checkout master
Switched to branch 'master'
$ git merge branchName
Auto-merging first.txt
CONFLICT (content): Merge conflict in first.txt
Automatic merge failed; fix conflicts and then commit the result.
```

YES!!冲突出现了，现在我们要解决它。这个时候打开此文件

```shell
$ vim first.txt
```

~~vim是最好的text editor！~~

我们会看到这个

```txt
<<<<<<< HEAD
this is conflicted text from master
=======
this is conflicted text from feature branch
>>>>>>> branchName
```

`<<<<<<<` 和`=======`是告诉我们哪里有冲突，并且是哪个branch。就这个而言前第二行是master上内容而第四行则是branchName上的内容，我们选取需要内容后记得删除135行哦！

Vim使用`i`进行编辑，完成后`esc`然后打`:q`推出。这个时候冲突已经解决完啦，我们需要进行一次commit来结束这次的合并操作。

```shell
$ git commit -am 'solve conflict'
```



## 其他

### git status

其实算是看当前分支的一个状态吧，可以看到是否有未被追踪的文件，有没有变化没有被commit。使用

```shell
$ git status
```



### git log

简单粗暴，显示commit记录，使用

```shell
$ git log
```

会显示出一串commit记录，每个包括id，作者，时间，分支，以及commit的时候的那个message。



### 切换到某次commit

```shell
$ git checkout specific-commit-id
```

这个`commit id`就需要我们去`git log`里面找来，嗯就是那一长串看不懂的hash。至于你具体想切换到哪个，就看你commit的时间以及你自己写的commit信息啦（滑稽）~~瞎写的话现在就作茧自缚了嘻嘻~~。



## 结语

Git大致就讲这么多啦，想更了解具体的Git，阔以参考document。https://git-scm.com/doc



# 3. Github

不得不说Github是真的好用，造好的轮子随便用，~~甚至还可以找到作业答案，当然要自己写作业了！~~，而且Github推出的Github desktop是真香。使用很简单，无师自通。言归正传，我们来康康Github怎么用。



## 注册及安装

想注册的话你需要准备的东西有：一个或多个邮箱。一个账号可以绑定N个邮箱，然后去https://github.com/进行注册。如果是学生且想免费申请Github pro可以访问https://education.github.com/pack。

申请完之后我们打开命令行

```shell
$ git config --global user.name "<your_name_here>"
$ git config --global user.email "<your_email@email.com>"
```

*注意：写名字时去除括号但保留引号，邮箱使用申请时的邮箱。以及，如果只是想在当前仓库用的话，去掉`global`就ok了*

现在我们连接ssh到Github，这样就可以通过ssh进行clone操作而不是http。先复制ssh key

```shell
$ pbcopy < ~/.ssh/id_rsa.pub
```

然后去Github网页，点击头像，出现下拉菜单，点击`Settings`，在用户设置的菜单栏选择`SSH and GPG keys`，点击`New SSH key`，在`title`栏写上你设备的名字，方便你辨认，把刚刚复制好的key复制到`key`那里。然后点击添加，如果出现输密码框就输入密码。



## 新建仓库

先去Github上建立一个新仓库，点击`Repositories`旁`New`按钮，填写Repo的名字后就可以确认建立。新建完后自动跳转到这个初始页面，在这里我们可以看到这个仓库的https地址，点击旁边的复制按钮备用。



## 连接仓库

现在，之前Git里的操作都可以正常用了，不过效果依然存在本地，想要和云端连接，我们需要学一些新操作。首先和remote连接

```shell
$ git remote add origin remote repository URL
```

这里的URL就是上一步操作复制得到的。然后

```shell
$ git push -u origin master
```

运行完之后，我们就可以在Github上看到啦。每当我们想要发布一个新的分支的时候，都需要运行这个命令，要将master改成分支的名字即可。



## 更新

当remote和本地的版本不一致时，我们会需要进行更新，可能是本地快于remote也可能是remote快于本地。我们使用`git pull`来拉取remote的更新，使用前要记得先commit本地的版本哦。如果要更新remote版本，我们直接用`git push`就可以做到了。



## 克隆

在Github上打开你要克隆的仓库主页，点击Clone or Download，然后可以选择用`https`还是`ssh`进行克隆，复制链接，打开命令行

```shell
$ git clone url
```

如果不是你的仓库，只能使用`https`哦，或者也可以选择下载。



## Github网页

既然他是有网页版的，那自然不能忽略网页版提供的服务。

### 搜索

我要说的搜索是在页面顶部Navigation bar里的那个搜索框，在哪里可以进行关键词搜索，能检索到有相关信息的公开仓库。在搜索结果的页面，我们可以选取特定语言（这个项目使用的语言），结果的排序方法等等。还有其他一些信息，就自己去看啦。

每个仓库都会显示`stars`，一般来说`stars`越多，认可度越高，也就越好。



### 关于Repo

因为~~要素过多~~有很多内容，我就选取部分说明，其他的功能自己康康就好啦。



**文件**

点到自己的任意一个仓库，我们可以直接在网上编辑文件。点开文件A，然后点击`Edit`就可以进行修改，修改完成后使用下方的`commit`进行保存。也可以直接新建文件，上传文件，删除文件等。不过比较神奇的操作是新建文件夹。具体如下：

点击`Create New File`，在文件名的地方输入文件夹名称并加上`/`, ~~神奇的事情发生了~~，文件夹就建好了。不过不可以建空文件夹哦，所以它会强行让你写一个文件名。



**合作**

如果想和组员共同编辑一个仓库，我们点击`settings`，然后`Manage Access`，这个时候会让你输入一下密码。进去后可以修改当前仓库权限以及邀请合作者，提供输入合作人的Github名字来进行查找和添加。



## 其他

**怎么fetch一个云端的分支（不在本地的）？**

```shell
$ git checkout --track origin/daves_branch
```



**如果不想用命令行怎么办？**

Github Desktop你值得拥有：https://desktop.github.com/



# 4. 写在最后

感谢大家看完我的~~废话~~教程，这大概是本人~~多年~~一年的使用经常用到的部分，希望可以给你们带来帮助。