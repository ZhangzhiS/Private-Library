---
title: "Git Pull Request"
date: 2020-01-25T16:31:34+08:00
categories: ['历史文章']
tags: ["历史文章"]
draft: false
---

## 前言

Pull-Request是[GitHub](https://github.com/)上对开源项目贡献代码的工作流程。

## 1. Fork原项目

![Fork](https://help.github.com/assets/images/help/repository/fork_button.jpg)

如上图，在项目的右上角，可以看到`Watch`、`Star`、`Fork`，三个按钮，点击`Fork`就可以了，成功之后在个人的仓库列表中可以看到你Fork的项目。

## 2. 将项目clone到本地

在克隆项目的时候，克隆的是自己仓库的代码，而不是源项目的。比如你fork了user-A的项目`project-a`，源项目地址为`https://github.com/user-A/project-a`，在你仓库中的项目地址为`https://github.com/your-username/project-a`。



```
git clone https://github.com/your-username/project-a
```

## 3. 创建自己的分支

新建一个自己的分支，后续开发在自己分支中开发，创建分支的时候，在分支名字中表达出这个分支是进行BUG修复，或者是开发新功能。具体参考源项目的命名规范。注意创建分支的时候选择的上游分支，看项目发布规则，是在dev分支开发或者是master分支进行开发。

```
git branch fixbug_dosomethings [上游分支]
```

## 4. 在自己的分支上进行开发

开发过程中的话，需要注意的就是代码规范了，最好是延续源项目的风格，当然，规范的开发风格都一样。经常提交代码是一个好习惯，避免异常状况发生之后造成损失。有权限的人可以看到你每次的提交

## 5. 发送一个Pull-Request

在源项目新建一个Pull-Request，就在选择分支按钮的旁边，`new pull request`，base repository，选择源项目，head repository选择你自己的仓库，选择好对应的分支就可以提交。

## 6. 后续

有权限的项目参与者能够看到你创建的合并，进行code review，或者进行测试，当他们认为可以合并，那么接受本次提交并合并到源项目的目标分支中。

**注意：本次提交没有关闭，你后续所有在这个分支上进行的修改，commit，都会进入此次pull-request中**。

