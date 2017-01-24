# 研发团队GIT开发流程新人学习指南

![FullSizeRende](media/14642498575507/FullSizeRender.jpg)


[TOC]

本文定位于为使用GIT标准分支开发流程的开发团队新人提供一份参考指南，其中的内容都是我们公司在研发团队初创时所遵循的一些开发流程标准，经过近一年的实践，虽说还有很多不足，但是随着团队经验的丰富和人员的扩张，我会适时地更新本文，分享我们在使用GIT开发流程中遇到的问题和解决方案。

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。

## 分支流程说明

### 简介

![](https://oayrssjpa.qnssl.com/2017-01-24-14724616710606.jpg)

项目中长期存在的两个分支

* `master`：主分支，负责记录上线版本的迭代，该分支代码与线上代码是完全一致的。
* `develop`：开发分支，该分支记录相对稳定的版本，所有的feature分支和bugfix分支都从该分支创建。

其它分支为短期分支，其完成功能开发之后需要删除

* `feature/*`：特性（功能）分支，用于开发新的功能，不同的功能创建不同的功能分支，功能分支开发完成并自测通过之后，需要合并到 **develop** 分支，之后删除该分支。
* `bugfix/*`：bug修复分支，用于修复不紧急的bug，普通bug均需要创建bugfix分支开发，开发完成自测没问题后合并到 **develop** 分支后，删除该分支。
* `release/*`：发布分支，用于代码上线准备，该分支从**develop**分支创建，创建之后由测试同学发布到测试环境进行测试，测试过程中发现bug需要开发人员在该release分支上进行bug修复，所有bug修复完后，在上线之前，需要合并该release分支到**master**分支和**develop**分支。
* `hotfix/*`：紧急bug修复分支，该分支只有在紧急情况下使用，从**master**分支创建，用于紧急修复线上bug，修复完成后，需要合并该分支到**master**分支以便上线，同时需要再合并到**develop**分支。

### 必读文章

[团队中的 Git 实践](http://www.open-open.com/lib/view/open1461324562769.html)
[Git 在团队中的最佳实践--如何正确使用Git Flow](http://www.open-open.com/lib/view/open1451353135339.html)



## 分支命令规范

### 特性（功能）分支

功能分支的分支名称应该为能够准确描述该功能的英文简要表述

    feature/分支名称
    
例如，开发的功能为 *新增商品到物料库*，则可以创建名称为 `feature/material-add`的分支。

### bug修复分支、紧急bug修复分支

bug修复分支的分支名称可以为Jira中bug代码或者是描述该bug的英文简称

    bugfix/分支名称
    hotfix/分支名称

比如，修复的bug在jira中代号为**MATERIAL-1**，则可以创建一个名为`bugfix/MATERIAL-1`的分支。

### release分支

release分支为预发布分支，命名为本次发布的主要功能英文简称

    release/分支名称

比如，本次上线物料库新增的功能，则分支名称可以为`release/material-add`。


## 常用操作命令简介

### 基本操作

基本命令这里就不多说了，基本跟以前一样，唯一的区别是注意分支是从哪里拉去的以及分支的命名规范。涉及到的命令主要包含以下，大家自己学习：

* git commit 
* git add [--all]
* git push
* git pull
* git branch [-d]
* git merge
* git cherry-pick
* git checkout [-b] BRANCH_NAME
* git stash

分支操作参考 [Git常用操作-分支管理](http://b.aicode.cc/git/2015/09/10/Git%E5%B8%B8%E7%94%A8%E6%93%8D%E4%BD%9C-%E5%88%86%E6%94%AF%E7%AE%A1%E7%90%86.html)


### 使用`git flow`简化操作

`git flow`是git的一个插件，可以极大程度的简化执行git标准分支流程的操作，可以在[gitflow-avh](https://github.com/petervanderdoes/gitflow-avh)安装。

> 如果是windows下通过安装包安装的git，则该插件默认已经包含，可以直接使用。

#### 初始化

使用`git flow init`初始化项目

    $ git flow init
    
    Which branch should be used for bringing forth production releases?
       - develop
       - feature-fulltext
       - feature-vender
       - master
    Branch name for production releases: [master]
    
    Which branch should be used for integration of the "next release"?
       - develop
       - feature-fulltext
       - feature-vender
    Branch name for "next release" development: [develop]
    
    How to name your supporting branch prefixes?
    Feature branches? [feature/]
    Bugfix branches? [bugfix/]
    Release branches? [release/]
    Hotfix branches? [hotfix/]
    Support branches? [support/]
    Version tag prefix? []
    Hooks and filters directory? [/Users/mylxsw/codes/work/e-business-3.0/.git/hooks]

#### 功能分支

    git flow feature
    git flow feature start <name>
    git flow feature finish <name>
    git flow feature delete <name>

    git flow feature publish <name>
    git flow feature track <name>

功能分支使用例子：

    $ git flow feature start material-add
    Switched to a new branch 'feature/material-add'
    
    Summary of actions:
    - A new branch 'feature/material-add' was created, based on 'develop'
    - You are now on branch 'feature/material-add'
    
    Now, start committing on your feature. When done, use:
    
         git flow feature finish material-add
    
    $ git status
    On branch feature/material-add
    nothing to commit, working directory clean
    $ git flow feature publish
    Total 0 (delta 0), reused 0 (delta 0)
    To http://dev.oss.yunsom.cn:801/yunsom/e-business-3.0.git
     * [new branch]      feature/material-add -> feature/material-add
    Branch feature/material-add set up to track remote branch feature/material-add from origin.
    Already on 'feature/material-add'
    Your branch is up-to-date with 'origin/feature/material-add'.
    
    Summary of actions:
    - The remote branch 'feature/material-add' was created or updated
    - The local branch 'feature/material-add' was configured to track the remote branch
    - You are now on branch 'feature/material-add'
    
    $ vim README.md
    $ git add --all
    $ git commit -m "modify readme file "
    [feature/material-add 7235bd4] modify readme file
     1 file changed, 1 insertion(+), 2 deletions(-)
    $ git push
    Counting objects: 3, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (3/3), done.
    Writing objects: 100% (3/3), 303 bytes | 0 bytes/s, done.
    Total 3 (delta 2), reused 0 (delta 0)
    To http://dev.oss.yunsom.cn:801/yunsom/e-business-3.0.git
       0d4fb8f..7235bd4  feature/material-add -> feature/material-add
    $ git flow feature finish
    Switched to branch 'develop'
    Your branch is up-to-date with 'origin/develop'.
    Updating 0d4fb8f..7235bd4
    Fast-forward
     README.md | 3 +--
     1 file changed, 1 insertion(+), 2 deletions(-)
    To http://dev.oss.yunsom.cn:801/yunsom/e-business-3.0.git
     - [deleted]         feature/material-add
    Deleted branch feature/material-add (was 7235bd4).
    
    Summary of actions:
    - The feature branch 'feature/material-add' was merged into 'develop'
    - Feature branch 'feature/material-add' has been locally deleted; it has been remotely deleted from 'origin'
    - You are now on branch 'develop'
    
    $ git branch
    * develop
      feature-fulltext
      feature-vender
      master
    
#### 预发布分支

    git flow release
    git flow release start <release> [<base>]
    git flow release finish <release>
    git flow release delete <release>
    
#### hotfix分支

    git flow hotfix
    git flow hotfix start <release> [<base>]
    git flow hotfix finish <release>
    git flow hotfix delete <release>

#### git-flow 备忘清单

参考[git-flow 备忘清单](http://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html)

## 总结

如果上面内容太多记不住，也没有关系，作为开发人员，刚开始的时候只要知道以下几点就足够了，其它的可以在碰到的时候再深入学习：

- 所有的新功能开发，bug修复（非紧急）都要从`develop`分支拉取新的分支进行开发，开发完成自测没有问题再合并到`develop`分支
- `release`分支发布到测试环境，由开发人员创建`release`分支（需要测试人员提出需求）并发布到测试环境，如果测试过程中发现bug，需要开发人员`track`到该release分支修复bug，上线前需要测试人员提交`merge request`到`master`分支，准备上线，同时需要合并回`develop`分支。
- 只有紧急情况下才允许从`master`上拉取`hotfix`分支，`hotfix`分支需要最终同时合并到`develop`和`master`分支（共两次merge操作）
- 除了`master`和`develop`分支，其它分支在开发完成后都要删除

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。

