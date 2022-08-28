# Git
Git对待数据的方式：直接记录快照，而非差异比较。每次提交更新，或在Git中保存项目状态时，主要是对当时的全部文件制作一个快照并保存这个快照的索引。如果文件灭有修改，则Git不在重新存储该文件，而是只保留一个链接指向之前存储的文件。
#### 其他版本管理系统
![图片](assets/IMG_2.png)

#### Git
![图片](assets/IMG_3.png)

## Git的三种状态
1.  **已提交（committed）**：数据已经安全的保存在本地数据库中。
2.  **已修改（modified）**：已修改表示修改了文件，但还没保存到数据库中。
3.  **已暂存（staged）**：表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

## 快速使用入门
### 获取 Git 仓库

有两种取得 Git 项目仓库的方法。

1.  在现有目录中初始化仓库: 进入项目目录运行 `git init` 命令,该命令将创建一个名为 `.git` 的子目录。
2.  从一个服务器克隆一个现有的 Git 仓库: `git clone [url]` 自定义本地仓库的名字: `git clone [url] directoryname`

### 记录每次更新到仓库

1.  **检测当前文件状态** : `git status`
2.  **提出更改（把它们添加到暂存区**）：`git add filename` (针对特定文件)、`git add *`(所有文件)、`git add *.txt`（支持通配符，所有 .txt 文件）
3.  **忽略文件**：`.gitignore` 文件
4.  **提交更新:** `git commit -m "代码提交信息"` （每次准备提交前，先用 `git status` 看下，是不是都已暂存起来了， 然后再运行提交命令 `git commit`）
5.  **跳过使用暂存区域更新的方式** : `git commit -a -m "代码提交信息"`。 `git commit` 加上 `-a` 选项，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 `git add` 步骤。
6.  **移除文件** ：`git rm filename` （从暂存区域移除，然后提交。）
7.  **对文件重命名** ：`git mv README.md README`(这个命令相当于`mv README.md README`、`git rm README.md`、`git add README` 这三条命令的集合)

### 一个好的 Git 提交消息

一个好的 Git 提交消息如下：

```
标题行：用这一行来描述和解释你的这次提交

主体部分可以是很少的几行，来加入更多的细节来解释提交，最好是能给出一些相关的背景或者解释这个提交能修复和解决什么问题。

主体部分当然也可以有几段，但是一定要注意换行和句子不要太长。因为这样在使用 "git log" 的时候会有缩进比较好看。
```

提交的标题行描述应该尽量的清晰和尽量的一句话概括。这样就方便相关的 Git 日志查看工具显示和其他人的阅读。

### 推送改动到远程仓库

-   如果你还没有克隆现有仓库，并欲将你的仓库连接到某个远程服务器，你可以使用如下命令添加：`git remote add origin <server>` ,比如我们要让本地的一个仓库和 Github 上创建的一个仓库关联可以这样`git remote add origin https://github.com/Snailclimb/test.git`
    
-   将这些改动提交到远端仓库：`git push origin master` (可以把 *master* 换成你想要推送的任何分支)
    
    如此你就能够将你的改动推送到所添加的服务器上去了。
    

### 远程仓库的移除与重命名

-   将 test 重命名为 test1：`git remote rename test test1`
-   移除远程仓库 test1:`git remote rm test1`

### 查看提交历史

在提交了若干更新，又或者克隆了某个项目之后，你也许想回顾下提交历史。 完成这个任务最简单而又有效的工具是 `git log` 命令。`git log` 会按提交时间列出所有的更新，最近的更新排在最上面。

**可以添加一些参数来查看自己希望看到的内容：**

只看某个人的提交记录：

```
git log --author=bob
```

### 撤销操作

有时候我们提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。 此时，可以运行带有 `--amend` 选项的提交命令尝试重新提交：

```
git commit --amend
```

取消暂存的文件

```
git reset filename
```


撤消对文件的修改:

```
git checkout -- filename
```


假如你想丢弃你在本地的所有改动与提交，可以到服务器上获取最新的版本历史，并将你本地主分支指向它：

```
git fetch origin
git reset --hard origin/master
```

### 分支

分支是用来将特性开发绝缘开来的。在你创建仓库的时候，*master* 是“默认”的分支。在其他分支上进行开发，完成后再将它们合并到主分支上。

我们通常在开发新功能、修复一个紧急 bug 等等时候会选择创建分支。单分支开发好还是多分支开发好，还是要看具体场景来说。

创建一个名字叫做 test 的分支

```
git branch test
```

切换当前分支到 test（当你切换分支的时候，Git 会重置你的工作目录，使其看起来像回到了你在那个分支上最后一次提交的样子。 Git 会自动添加、删除、修改文件以确保此时你的工作目录和这个分支最后一次提交时的样子一模一样）

```
git checkout test
```

![](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-3%E5%88%87%E6%8D%A2%E5%88%86%E6%94%AF.png)

你也可以直接这样创建分支并切换过去(上面两条命令的合写)

```
git checkout -b feature_x
```

切换到主分支

```
git checkout master
```

合并分支(可能会有冲突)

```
 git merge test
```

把新建的分支删掉

```
git branch -d feature_x
```

将分支推送到远端仓库（推送成功后其他人可见）：

```
git push origin
```

# Docker
## 容器
容器就是将软件打包成标准化单元，以用于开发、交付和部署
- 容器镜像是轻量、可执行的独立软件包，包含软件运行所需的所有内容：代码、运行时环境、系统工具、系统库和设置。
- 容器化软件适用于基于linux和Windows的应用，在任何环境中都能始终如一地运行
- 容器赋予了软件独立性，时期免收外在环境差异的影响，从而有助于减少团队间在相同基础设施上运行不同软件时的冲突。

![图片](assets/IMG_4.png)

## 物理机、虚拟机与容器
物理机：普通的主机，笔记本，台式  
 
虚拟机：建立在物理机上  
 
容器：可以建立在物理机和虚拟机上
![图片](assets/IMG_5.png)



## Docker的特点
- 轻量：多个Docker容器共享机器的操作系统内核，能迅速启动，只占用很少的计算和内存资源
- 标准：Docker容器基于开放式标准，能够在所有主流操作系统上运行
- 安全：Docker赋予应用的隔离性不仅限于彼此隔离，还独立于底层的基础设施

## Docker的基本概念
- 镜像（Image）
- 容器（Container）
- 仓库（Repository）
![图片](assets/IMG_6.png)

### 镜像：一个特殊的文件系统
提供容器运行时所需的程序、库、资源、配置等文件，为运行时准备一些配置参数（如匿名卷、环境变量、用户等）。构建镜像时，会一层层构建，前一层是后一层的基础，每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。
### 容器：镜像运行的实体
镜像与容器之间的关系，就像类与实例一样，镜像时静态的定义，容器是镜像运行时的实体。  
容器的实质是进程，运行于属于自己的独立的命名空间
### 仓库：集中存放镜像文件的地方
镜像仓库是 Docker 用来集中存放镜像文件的地方类似于我们之前常用的代码仓库


## 常见命令
### 基本命令
```
docker version # 查看docker版本
docker images # 查看所有已下载镜像，等价于：docker image ls 命令
docker container ls #	查看所有容器
docker ps #查看正在运行的容器
docker image prune # 清理临时的、没有被使用的镜像文件。-a, --all: 删除所有没有用的镜像，而不仅仅是临时文件；
```
### 拉取镜像
```
docker search mysql # 查看mysql相关镜像
docker pull mysql:5.7 # 拉取mysql镜像
docker image ls # 查看所有已下载镜像
```
### 删除镜像
1. 在镜像没有被容器引用前,可以通过标签名称或镜像ID删除
```
docker rmi [image]
```
2. 在镜像被容器引用后，首先需要暂停容器，然后才能进行删除
```
docker stop c4c
```
### 容器指令
通过镜像运行一个容器,首先下载镜像,然后再运行
```
docker pull tomcat:8.0-jre8;
docker run tomcat:8.0-jre8;
```
其中`CONTAINER_ID`为容器的 id，`IMAGE`为镜像名，`COMMAND`为容器内执行的命令，`CREATED`为容器的创建时间，`STATUS`为容器的状态，`PORTS`为容器内服务监听的端口，`NAMES`为容器的名称。
因为容器具有隔离性，若是想通过8080端口访问容器内部的tomcat，则需要对宿主机端口与容器内的端口进行映射：
```
docker run -p 8080:8080 tomcat:8.0-jre8 
```
前一个`8080`为宿主机端口，后一个`8080`为容器内的端口

```
docker ps -a # 将所有运行和非运行的容器全部列举出来
docker ps -q # 查询正在运行的容器id
docker start c2f # 启动
docker restart c2f # 重启
docker stop c2f # 停止
docker kill c2f # 立即停止
docker rm -f c2f # 强制删除
docker rm -f $(docker ps -qa) # 删除所有容器
docker logs c2f # 查看容器的运行日志
docker exec -it c2f bash # 与容器进行交互

```

现在我们已经能够进入容器终端执行相关操作了，那么该如何向 tomcat 容器中部署一个项目呢？

```
docker cp ./test.html 289cc00dc5ed:/usr/local/tomcat/webapps
```

通过`docker cp`指令能够将文件从 CentOS 复制到容器中，`./test.html`为 CentOS 中的资源路径，`289cc00dc5ed`为容器 id，`/usr/local/tomcat/webapps`为容器的资源路径，此时`test.html`文件将会被复制到该路径下。

```
[root@izrcf5u3j3q8xaz ~]# docker exec -it 289cc00dc5ed bash
root@289cc00dc5ed:/usr/local/tomcat# cd webapps
root@289cc00dc5ed:/usr/local/tomcat/webapps# ls
test.html
root@289cc00dc5ed:/usr/local/tomcat/webapps#
```

若是想将容器内的文件复制到 CentOS 中，则反过来写即可：

```
docker cp 289cc00dc5ed:/usr/local/tomcat/webapps/test.html ./
```

所以现在若是想要部署项目，则先将项目上传到 CentOS，然后将项目从 CentOS 复制到容器内，此时启动容器即可。

## Docker数据卷
实现宿主机与容器之间的文件共享，如果对宿主机的文件进行修改，则直接影响到容器，无需再进行复制

现在若是想将宿主机中`/opt/apps`目录与容器中`webapps`目录做一个数据卷，则应该这样编写指令：

```
docker run -d -p 8080:8080 --name tomcat01 -v /opt/apps:/usr/local/tomcat/webapps tomcat:8.0-jre8
```
Docker 会将容器内的`webapps`目录与`/opt/apps`目录进行同步，而此时`/opt/apps`目录是空的，导致`webapps`目录也会变成空目录，所以就访问不到了。

此时我们只需向`/opt/apps`目录下添加文件，就会使得`webapps`目录也会拥有相同的文件，达到文件共享

这种方式设置的数据卷称为自定义数据卷，因为数据卷的目录是由我们自己设置的，Docker 还为我们提供了另外一种设置数据卷的方式：

```
docker run -d -p 8080:8080 --name tomcat01 -v aa:/usr/local/tomcat/webapps tomcat:8.0-jre8
```

此时的`aa`并不是数据卷的目录，而是数据卷的别名，Docker 会为我们自动创建一个名为`aa`的数据卷，并且会将容器内`webapps`目录下的所有内容复制到数据卷中，该数据卷的位置在`/var/lib/docker/volumes`目录下：

```
[root@centos-7 volumes]# pwd
/var/lib/docker/volumes
[root@centos-7 volumes]# cd aa/
[root@centos-7 aa]# ls
_data
[root@centos-7 aa]# cd _data/
[root@centos-7 _data]# ls
docs  examples  host-manager  manager  ROOT
```

此时我们只需修改该目录的内容就能能够影响到容器。