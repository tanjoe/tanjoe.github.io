---
title: '如何使用git submodule和git filter-repo从仓库中分离特定目录'
date: 2020-07-23 10:09:23
img: /images/git.png
tags: 
- git
---

碰到了一个这样的场景：同事为了单元测试，将众多测试资源文件提交到了git仓库内，这导致仓库体积陡然膨胀。但另一方面，编写好的测试用例又确实依赖这些测试资源文件。那么，有没有办法能达到以下目标：

- 分离工程文件和测试资源，以便能够单独管理二者
- 清洗提交记录，将测试资源在提交历史中“抹去”，以便减小仓库的体积

搜索一番，发现了`git submodule`和`git filter-repo`两个工具刚好可以满足这两个需求。



## 1  `git submodule`的使用

`submodule`是git自带的一个工具，详细的介绍参见[Git-工具-子模块]([https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97](https://git-scm.com/book/zh/v2/Git-工具-子模块))，这里不再赘述，只简单说明如何利用`git submodule`满足上述需求。

### 1.1 从仓库中移除测试资源

假设工程结构如下：

```shell
├─MyLib
│  ├─include
│  └─src
├─MyLibTest
   ├─assets
   └─src
```

其中`MyLib`为库的工程，而`MyLibTest`则是`MyLib`的单元测试工程，测试资源存放在`MyLibTest/assets`下，也正是我们需要移除的目录。

首先，在git bash中执行

```shell
git rm -r --cached MyLibTest/assets
```

将`MyLibTest/assets`从git仓库索引中移除，但不实际删除该目录。

> 关于`git rm`的使用，参见[git-rm](https://git-scm.com/docs/git-rm)



### 1.2 新建测试资源仓库

#### 新建远程仓库

在git服务器上新建一个仓库，记作`MyLibTestResource`，用于存放测试资源。

#### 新建本地仓库

新建一个目录，这里记作`TestResource`，在其中建立git仓库，将`MyLibTest/assets`中的内容复制到该目录下，随后提交。再执行

```shell
git remote add origin git@MyLibTestResource.git #git@MyLibTestResource.git替换为MyLibTestResource的真实git地址
```

添加远程仓库，随后执行`git push`推送即可。



### 1.3 添加子模块

回到`MyLib`下，执行

```shell
git submodule add git@MyLibTestResource.git MyLibTest/assets
```

添加子模块`MyLibTestResource`，子模块的内容会同步到`MyLibTest/assets`下。



### 1.4 克隆包含子模块的项目

克隆包含子模块的项目有二种方法：一种是先克隆父项目，再更新子模块；另一种是直接递归克隆整个项目。

#### 克隆父项目，再更新子模块

```shell
#克隆父项目MyLib
git clone git@MyLib.git #git@MyLib.git替换为MyLib的真实git地址

#初始化子模块
cd MyLib
git submodule init

#更新子模块
git submodule update
```

#### 递归克隆整个项目

```shell
git clone git@MyLib.git --recursive
```



## 2  `git filter-repo`的使用

利用`git submodule`可以将测试资源从`MyLib`中分离，但`MyLib`仓库中仍然记录着测试资源的提交与更改，这导致仓库体积仍然很庞大。那么，如何改写git提交历史，将测试资源相关的提交与更改从历史中“抹去”呢？

对于重写历史，我们已有`git commit --amend`、`git rebase`等工具可用，但这些指令都只适用于一个或几个提交的修改，修改大量提交时用这些指令会显得非常繁琐。Git还提供了一个`git filter-branch`用于改写历史中的大量提交，不过很可惜，它至少有以下缺陷：

- 清理速度慢
- 只能按文件名清理

由于`git filter-branch`实现上的缺陷，第三方的重写历史工具应此而生，如[BFG Repo Cleaner](https://rtyley.github.io/bfg-repo-cleaner/)和[git filter-repo](https://github.com/newren/git-filter-repo)。这两个工具应该都可以满足我们的需求，不过由于git官方教程推荐了后者，故决定选取`git filter-repo`。

### 2.1 `git filter-repo`的安装

#### 2.1.1 Python和pip的安装

`git filter-repo`是用Python开发的工具，因此需要安装Python环境，其安装要求如下：

> - git >= 2.22.0 at a minimum; [some features](https://github.com/newren/git-filter-repo#upstream-improvements) require git >= 2.24.0 or later
> - python3 >= 3.5

在https://www.python.org/downloads/release/python-384/下载安装Python 3.8.4（当然，如果是Linux环境可以直接用包管理工具安装）

`pip`是Python的包管理器，我们随后将用`pip`安装`filter-repo`。从Python 3.4开始，官网的安装包中已经自带了`pip`，在安装时用户可以直接选择安装。也可以从https://pypi.org/project/pip/#files下载`pip`的源码包，解压后执行

```shell
python setup.py install
```

完成`pip`的安装。

#### 2.1.2 filter-repo的安装

命令行执行

```shell
pip3 install git-filter-repo
```

即可。



### 2.2 滤除目录

在`MyLib`目录中，执行

```shell
git filter-repo --path MyLibTest/assets --invert-paths
```

即可滤除`MyLibTest/assets`在历史中的记录。其中`--path`用于指定目录，`--invert-paths`说明指定的目录需要从历史中滤除。

> 关于`git filter-repo`的详细使用，参见[git-filter-repo Manual Page](https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html)



### 2.3 重写推送仓库

在更改提交的历史后，需要重新推送仓库，执行

```shell
git push --force
```

即可。

注：使用`--force`选项覆盖提交需要你拥有对应的权限。并且一旦操作完成，本地和远程仓库的提交历史都将被改写，**操作前请确认这样是否能被接受**。