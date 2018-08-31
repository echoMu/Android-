### 一 、认识Git

#### 1.Git

Git是 Linus Torvalds 开发的一个开放源码的版本控制软件, 是一个快速的分布式版本控制系统，拥有强大的分支管理。



**Git 与 SVN 区别**

它和SVN的区别在于，SVN是存储每次提交之间的差异，而Git正好与之相反，它会把每次提交的文件的的全部内容都记录下来。

如果你是一个具有使用SVN背景的人，你需要做一定的思想转换，来适应Git提供的一些概念和特征。

Git 与 SVN 区别点：

- Git是分布式的，SVN不是，这是GIT和其它非分布式的版本控制系统，例如SVN，CVS等，最核心的区别；

- Git把内容按元数据方式存储，而SVN是按文件，所有的资源控制系统都是把文件的元信息隐藏在一个类似.svn,.cvs等的文件夹里；

- Git分支和SVN的分支不同，分支在SVN中一点不特别，就是版本库中的另外的一个目录。

  

**核心概念**

Git和其他版本控制系统如SVN的一个不同之处就是有暂存区的概念，接下来我们来理解下Git 工作区、暂存区和版本库的概念 。

* #### 工作区（Working Directory）

  就是你在电脑里能看到的目录 。

* #### 版本库（Repository）

  版本库又名仓库（**repository**），你可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。 

  工作区有一个隐藏目录`.git`，这个其实是Git的版本库。 

  Git的版本库里存了很多东西，其中最重要的就是称为**stage（或者叫index）**的**暂存区**，还有Git为我们自动创建的第一个分支`master`，以及指向`master`的一个指针叫`HEAD`。 

  

  下面这个图展示了工作区、版本库中的暂存区和版本库之间的关系： 

  

  ![img](http://upload-images.jianshu.io/upload_images/817079-7cafc9b3423d6710.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

  

  我们把文件往Git版本库里添加的时候，是分两步执行的：

  第一步是用`git add`把文件添加进去，实际上就是把文件修改添加到暂存区；

  第二步是用`git commit`提交更改，实际上就是把暂存区的所有内容提交到当前分支。

  

  **Git的工作流程**

  一般工作流程如下：

  * 克隆 Git 资源作为工作目录。
  * 在克隆的资源上添加或修改文件。
  * 如果其他人修改了，你可以更新资源。
  * 在提交前查看修改。
  * 提交修改。
  * 在修改完成后，如果发现错误，可以撤回提交并再次修改并提交。

  

#### 2.gitlab

[GitLab](https://about.gitlab.com/)是利用 Ruby on Rails 一个开源的版本管理系统，实现一个自托管的Git项目仓库，可通过Web界面进行访问公开的或者私人项目。它拥有与Github类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。 

我们所有的工作都是在本地机器上完成的，而项目组内已经创建好了代码托管服务GitLab，等到推送代码时我们要做的就是将本地项目Push到远程仓库，或者提出merge request进行分支合并，其他人就可以访问到你的代码更新。 



### 二、安装, 配置Git

#### 1.安装

在Windows下安装Git是很简单的，访问 [git官网下载页面](https://git-scm.com/downloads)，选择Download for Windows，然后按默认选项安装即可 。安装成功后，AS一般会自动地引入git环境。

打开setting–>Version Control–>Git–>Path to Git executable，看到git.exe已经被设置好了，点击Test按钮，会提示AS已经成功引入了git环境。


#### 2.配置

使用Git的第一件事就是设置你的用户名和email,这些就是你在提交commit时的签名。

* *使用AS*

  在AS中，当你打开菜单栏的VCS->Git->Clone，输入仓库信息后，这时候就会要求你输入名字和email，输入即可。

* *使用bash（或AS的terminal工具，下同）*

  打开git bash 工具，执行下面的命令之后，就设置好了用户名称和邮箱。

  ```bash
  $ git config --global user.name "you name"
  $ git config --global user.email "you email"
  ```
需要注意的是，--global表示你这台机器上所有的Git仓库都会使用这个配置，当然你也可以不用这个选项，直接进入某个仓库的目录为其指定不同的用户名和Email。



### 三、基本用法

#### 1.获取git仓库

使用 git clone 拷贝一个 Git 仓库到本地，让自己能够查看该项目，或者进行修改。

如果你需要与他人合作一个项目，或者想要复制一个项目，看看代码，你就可以克隆那个项目。

为了得一个项目的拷贝(copy),我们需要知道这个项目仓库的地址(Git URL)。Git能在许多协议下使用，所以Git URL可能以ssh://, http(s)://，git://为前缀。

我们需要从gitlab上的Git仓库拷贝项目（类似 **svn checkout**） ，这里采用gitlab上的git项目，使用其http协议的URL来拉取仓库代码。

* *使用AS*

  打开菜单栏的VCS->Git->Clone，输入Git URL和本地要放的目录，目录默认为仓库名，也可以修改。点击Clone，AS就会下载该仓库的代码并导入工程。

* *使用bash*

  克隆仓库的命令格式为： 

  ```bash
  git clone <repo> <directory>
  ```

  参数说明：

  - repo:Git 仓库。
  - directory:本地目录，不填代表当前目录

  例如：

```bash
$ git clone http://*.*.*.*/practice/git-demo.git
```

将项目克隆到本地之后，我们就可以开始工作啦。



#### 2.正常工作流程

**把文件添加到仓库**

`git add`不但是用来添加不在版本控制中的新文件，也用于添加已在版本控制中但是刚修改过的文件。在这两种情况下, git都会获得当前文件的快照并且把内容暂存(stage)到索引中，为下一次commit做好准备。

* *使用AS*

  在AS增加文件会有如下弹框询问你是否将文件添加到Git暂存区，你可以选择添加，还可以记住选择，也可以先不Add。
  
在文件需要被Add时，在AS文件视图选择要Add的文件，点击右键VCS->Git->Add完成Add。

​	如何知道文件已经被Add？

​	当文件名称颜色由处于未Add状态的红色变为已Add但未被commit状态的绿色，文件已经被成功Add。

* *使用bash*

  例如，使用`git add`命令将新增的2个文件添加到Git暂存区中。

```bash
$ git add 1.md 2.md
```



**把文件提交到本地仓库**

使用`git add`命令将想要快照的内容写入缓存区， 而执行`git commit`则是将缓存区内容添加到本地仓库中。 

Git 会为你的每一个提交记录你设置过的的用户名与电子邮箱地址。

* *使用AS*

  想要提交这些修改过的代码时，依次点击VCS->Git->Commit file，勾选需要commit的文件，在Commit Message框输入提交的注解，点击右下方commit按钮完成。

* *使用bash*

  输入`git commit`命令，在-m 选项后面加上本次修改的注释，完成后就会记录一个新的项目版本。

```bash
$ git commit -m "add 1.md 2.md"
[master (root-commit) 692d1f4] add 1.md 2.md
 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 1.md
 create mode 100644 2.md
```



**推送到远程仓库**

* *使用AS*

  依次点击VCS->Git->Push,点击push。

* *使用bash*

  输入`git push`命令推送本地的"master" 分支去更新远程的"master"分支。

```bash
$ git push
Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 211 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To http://10.20.1.19/practice/git-demo.git
 * [new branch]      master -> master
```



**更新远程库**

完成推送后，别人需要拉取你的更新。

* *使用AS*

  依次点击VCS->Git->Pull,在下图上面箭头处勾选想要拉取的分支，点击pull。

* *使用bash*

  输入`git pull`命令，`git pull`命令执行两个操作: 它从远程分支(remote branch)抓取修改的内容，然后把它合并进当前的分支。

```bash
$ git pull
```

**查看Log日志** 

在使用 Git 提交了若干更新之后，又或者克隆了某个项目，想回顾下提交历史，我们可以使用`git log`来查看。 

* *使用AS*

  打开底部面板的version control查看log日志，点击某条记录，可以看到该条记录的详细提交信息(提交日志、更新文件细节、本地改变细节、分支等各种信息) 。

* *使用bash*

```bash
$ git log
commit c9589dda1b341e604663f3f134a2486e314ab65c (HEAD -> master)
Author: your name <your name@phone580>
Date:   Mon Aug 13 14:55:32 2018 +0800

    add files 1.md 2.md
```



#### 3.分支与合并

**查看分支** 

点击AS右下角的Git:master->New branch，然后出现下面的界面，在这里可以查看本地分支和远程分支情况，并且还能对本地和远程分支进行查看、切换、合并、删除的相关操作。

**创建分支**

一个Git仓库可以维护很多开发分支，现在我们来创建一个新的叫”dev-1.0.0”的分支。

* *使用AS*

  点击AS右下角的Git:master->New branch，输入分支名称，并勾选Checkout banch。点击ok之后会发现刚才右下角的master变成了你建的分支名字，说明创建成功了。

* *使用bash*

```bash
$ git branch dev-1.0.0
```

不过，到这一步只说明见创建成功了一个本地分支而已，在远程分支上还没有你刚才创建的这个分支。这时候，我们就可以在新的分支上进行各种操作了。



**操作分支**

* *使用AS*

  切换分支：点击AS右下角的Git:（分支名），点击你想要切换过来的分支名称，点击Checkout完成切换。

  在这个菜单里，你还可以对分支再进行merge、rename和delete等操作。

* *使用bash*

  切换分支时使用`git checkout`命令。

```bash
$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
```

​	假如你不再需要这个分支了，也可以删除它。

```bash
$ git branch -d dev-1.0.0
```

​         `git branch -d`只能删除那些已经被当前分支的合并的分支。



**如何合并**

现在，我们将分支切到dev-1.0.0，在1.md文件里写入一句话"this is a file named 1.md."，将这个改变add并commit。

将分支切换到master。

* *使用AS*

  点击AS右下角的Git:master，点击想要合并过来的分支dev-1.0.0，点击merge。

* *使用bash*

  你可以用下面的命令来合并两个分离的分支：`git merge`。

```bash
$ git merge dev-1.0.0
Updating 692d1f4..656dc61
Fast-forward
 1.md | 1 +
 1 file changed, 1 insertion(+)
```

​	可以看到，dev-1.0.0分支上的改变已经被合并到master上了。

```bash
$ cat 1.md
this is a file named 1.md.
```



**解决冲突**

当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。

现在切换到dev-1.0.0，创建dev-1.0.1分支并切换到此分支，在1.md文件第2行增加一句话“this is dev-1.0.1.”，并add和commit。

切换到dev-1.0.0，在1.md文件第2行增加一句话“this is dev-1.0.0.”，并add和commit。

这个时候，我们来合并dev-1.0.1分支，发现产生了冲突，那我们就需要来解决冲突了。

* *使用AS*

  有冲突的merge操作后，会弹出这样的界面，告诉了我们产生冲突的文件，在右边也提供了解决冲突的几种方式，可以根据需要解决冲突。 这里有3个选项，接受你自己的代码、接受别人的代码和合并。我们选择合并，进入合并界面。

		我们需要在下面的界面手动处理产生冲突的代码，最左边的是你本地的代码，最右边的是要合并的分支上的代码，中间是合并之后的结果。根据实际情况来合并，合并完成后，点击右下角的Apply完成合并。

* *使用bash*

  输入`git merge`命令。

```bash
$ git merge dev-1.0.1
Auto-merging 1.md
CONFLICT (content): Merge conflict in 1.md
Automatic merge failed; fix conflicts and then commit the result.
```

​	输入`git diff`命令就可以查看当前有哪些文件产生了冲突，冲突的详细内容是什么。

```bash
$ git diff
diff --cc 1.md
index 420232e,a7a6fd5..0000000
--- a/1.md
+++ b/1.md
@@@ -1,2 -1,2 +1,6 @@@
  this is a file named 1.md.
- this is dev-1.0.0
 -this is dev-1.0.1
++<<<<<<< HEAD
++this is dev-1.0.0
++=======
++this is dev-1.0.1
++>>>>>>> dev-1.0.1
```

​	可以看到，当前冲突的内容是在1.md文件里，出现在第2行内容。

​	编辑有冲突的文件，解决了冲突后把这个文件添`add`到索引(index)中去，用`git commit`命令来提交，就像平时修改了一个文件 一样。

​	使用`git log`查看我们刚才的操作，已经成功解决冲突，合并完成了。

```bash
$ git log
commit 0e04d8d2d516ef507ceebb8b0e4113e5cdf12ded (HEAD -> dev-1.0.0)
Merge: 66f08fc 5cc0fd0
Author: name <name@xxx.com>
Date:   Thu Aug 9 15:19:02 2018 +0800

    Merge branch 'dev-1.0.1' into dev-1.0.0
```



#### 4.标签

发布一个版本时，我们通常先在版本库中打一个标签（tag），这样，就唯一确定了打标签时刻的版本。将来无论什么时候，取某个标签的版本，就是把那个打标签的时刻的历史版本取出来。所以，标签也是版本库的一个快照。

Git的标签虽然是版本库的快照，但其实它就是指向某个commit的指针（跟分支很像对不对？但是分支可以移动，标签不能移动），所以，创建和删除标签都是瞬间完成的。tag就是一个让人容易记住的有意义的名字，它跟某个commit绑在一起。 



**创建标签**

在Git中打标签非常简单，首先，切换到需要打标签的分支上 。

* *使用AS*

  一次点击VCS->Git->Tag，弹出下面窗口，在Tag Name框中输入tag名称，在Commit框选择对应的某个commit，在Message框输入注解，然后点击Create Tag就完成了。

* *使用bash*

  敲命令`git tag <name>`就可以打一个新标签。

  默认标签是打在最新提交的commit上的。 

  可以用命令`git tag`查看所有标签。

  

### 四、多人协作

#### 1.项目初始化

- 技术负责人建立代码库，进行项目的初始化工作。

- mater分支的存放的应该是经过测试的，保持稳定的代码。

- 只有技术负责人拥有对master分支的操作权限，其他人员对master上的所有变更只能通过merge request从test分支或hotfix分支合并。

- 技术负责人拥有master分支的merge权限，在每一次merge操作完成之后，应该立即打上对应的版本的tag。

  

#### 2.日常开发

- 技术负责人从master分支上拉取以dev-版本号为名称的开发分支（例如dev-1.0.0），供开发人员进行新版本的开发。
- 在dev进行多人协作时，每位开发人员在提交各自代码的时侯，应注意要先pull代码，如果有冲突，则解决冲突，并在本地提交；没有冲突或者解决掉冲突后，再push到远程库上。
- 新版本开发完成后，从该版本的dev分支上拉取以test-版本号为名称的测试分支（例如test-1.0.0），准备提测。



#### 3.产品提测

- test分支用于测试阶段，任何人员不得在上面进行任何的开发和bug修复，所有的改变都从dev分支中合并过来。
- test分支名称和dev分支名称一一对应，一条dev开发分支就对应一条test测试分支（例如dev-1.0.0分支就对应test-1.0.0分支）。
- 如果测试出bug，开发人员应该在对应的dev分支上修复bug，并提交，通知技术负责人合并到test分支上面去。
- test分支测试通过后，通过提merge request通知技术负责人，将该test分支合并到master上，并添加该次版本号tag（例如tag-1.0.0）。



#### 4.紧急修复

- 线上发生缺陷需要紧急修复时，由技术负责人从master上checkout代码建立hotfix分支，以hotfix-版本号为名称（例如hotfix-1.0.1），开发人员拉起该分支进行修复工作。
- hotfix分支测试通过之后，通过提merge request通知技术负责人，将该hotfix分支合并到master上，并添加该次版本号tag（例如tag-1.0.1）。

[github传送门]()