## [Developing](http://source.android.com/source/developing.html) ##
--------
### Overview ###

在**Android** code上开发时,需要同时使用**Git**和**Repo**.大部分的情况下,使用单独的**git**命令能够代替**repo**操作. 或者使用**repo+git**的混合指令. 在基本的跨网络操作中使用**repo**可以是工作变得更简单高效.

**Git**是为了管理在多个代码库中发布的大型项目而开发的开源代码管理系统.在**android**实际处理中,将使用**git**作为进行本地操作的工具(`local branching/commits/diffs/edits`).它的主要目标是为了管理从爱好者社区到设备提供商的完整支持. 基本上,每个**git**组件仓库都可以被替代,而且特定的**git**组合能够在**android**主分支之外继续进行开发.

**Repo**是构建在**git**之上的一个仓库管理工具.**repo**定义所需的所有**git**仓库列表,上传数据到版本管理系统.并且提供**android**开发流程的部分自动化实现.**repo**不会替代**git**的操作,而是将**git**的操作进行打包以更方便的对代码集合进行处理.

**Gerrit**是一个对以**git**为代码管理系统的项目进行代码**review**系统.它是基于**web**的. **gerrit**鼓励授权用户更多的使用**git**进行代码提交.同时也可以很方便的基于**web**进行代码比较/评论/合并等操作.

----------

### Basic Workflow ###
以仓库为处理对象的基本操作模式如下:

![Basic Android workflow](http://source.android.com/images/submit-patches-0.png)![](http://source.android.com/images/submit-patches-0.png)

1.使用**repo start**开始一个新的分支. 

2.编辑文件.

3.使用**git add**保存修改.

4.使用**git commit**提交修改.

5.使用**repo upload**上传修改到review系统.

----------

### Task Reference ###
以下列出了使用repo和git进行操作的描述集合. 具体使用方式参见**[Downloading the source](http://source.android.com/source/downloading.html)** / **[Using Repo](http://source.android.com/source/using-repo.html)** / **[Learning Git](http://source.android.com/source/git-resources.html)**.


----------

#### 同步代码 ####
同步所有可用项目的文件:

    $ repo sync

同步选择项目的文件:

    $ repo sync prj1 prj2 ...

----------

#### 创建主分支 ####

为指定项目创建分支:

    $ repo start branch_name prj_name

参考**[android.googlesource.com](android.googlesource.com)**来查看所有项目的列表,如果已经存在指定项目的目录,则可以直接使用当前项目?.

在本地工作环境中切换到另一个已经创建的分支:

    $ git checkout branch_name

查看所有存在的分支:

    $ git branch**
    or
    $ repo branches
当前所处的分支名字将由*开头.

----------

#### 缓存文件修改(staging) ####
缺省模式下,git对项目中的修改仅进行通知但并不进行跟踪.为了让git保存当前修改.必须通过commit对其进行标注,这个过程称为**"staging"**.

将当前修改加入缓存区:

    $ git add
这个命令可以接受单个/多个文件名,项目中的目录名作为参数,**git add**不是简单的将文件加入到git仓库中,它同时可以用来缓存文件修改/删除等操作.


----------

#### 查看工作区状态 ####
列出当前文件状态:

    $ repo status
查看未提交的修改:

    $ repo diff
注意**repo diff**显示的是所有当前即将被提交的部分同当前工作目录的本地修改部分的区别. 如果需要查看即将被提交部分同已提交部分的区别,需要使用**git diff**.该命令必须在git仓库目录下执行:

    $ cd ~/work_path/prj
    $ git diff --cached

----------

#### 提交修改 ####
**commit**是git管理中的一个基本单元,它由整个项目当前目录/文件内容的快照(snapshot)来构成.在git中提交:

    $ git commit
此时git要求输入一份提交信息,一般情况下请按照修改内容书写.如果不输入提交信息,**commit**操作会被取消.


----------

#### 上传修改到Gerrit ####
上传修改前,更新到最新版本:

    $ repo sync
然后运行:

    $ repo unload
这个命令将列出所有提交的内容并提示选择上传到review server的那个分支. 如果只存在一个分支,则只会出现**y/n**提示.


----------

#### 同步冲突恢复 ####
如果**repo sync**命令运行提示同步冲突:

    $ //modify if necessary
    $ git add
    $ git commit
    $ git rebase --continue

- 查看未合并的文件(状态为**U**). 
- 按需修改冲突区域. 
- 切换到对应的项目目录,对有问题的文件进行**git add**和**git commit** 操作,然后**rebase**修改. 
- 完成rebase操作后,继续进行**repo sync**.


----------

#### 清理本地库 ####
当修改在**Gerrit**中全部合并之后,更新本地工作目录:

    $ repo sync
安全移除过时的分支:

    $ repo prune

----------

#### 删除本地库 ####
由于所有缓存信息均存在于本地库,只需要删除文件系统的工作目录即可完成删除动作.

    $ rm -fr work_path
此操作会将所有未提交review的修改永久删除.


----------
#### Git & Repo Cheatsheet ####

![Git and Repo cheatsheet](http://source.android.com/images/git-repo-1.png)


----------


