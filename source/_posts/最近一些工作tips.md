---
title: 最近一些工作tips
author: Liu Yuze
avatar: 'https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg'
authorLink: liuyuze.site
authorAbout: 好少年光芒万丈
authorDesc: 好少年光芒万丈
categories: 技术
comments: true
mathjax: true
date: 2019-11-06 10:31:19
tags:
    - tips
    - 技术
    - 学习笔记

keywords: work_tips
description: Tips for work
photos: https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/123829.jpg
---

# 在Mac 上使用VS Code进行单步调试
#### 系统
MAC OS : Catalina, Version 10.15

Xcode : Version 11.2
#### 问题 
目前按照VS Code的official tutorial，在mac上无法对代码进行单步调试。
#### 可能原因
Xcode更新后， 缺少文件lldb-min
#### 解决方案
* 方案1 （未验证）： 如果有老版本的Xcode，并可以定位到lldb-min文件，那么在launch.json文件中添加一行环境变量：
  ` "miDebuggerPath": "${path to lldb-min}" `
  例如： 
  `"miDebuggerPath":"/Applications/Xcode.app/Contents/Developer/usr/bin/lldb-mi"`

* 方案2（目前可行）：在VS Code中搜索并添加extension ：[Codelldb](https://github.com/vadimcn/vscode-lldb/blob/v1.4.0/MANUAL.md)，并按照下面简单配置launch.json文件

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "test",
            "type": "lldb",
            "request": "launch",
            "program": "${workspaceFolder}/helloworld.out",
            "args": [],
            "cwd": "${workspaceFolder}"
        }
    ]
}
```

# sublime Text 3 中文乱码解决方案
Download and install ConvertToUTF8 package

# git command on Windows
```
1. Install git from official website, if you are in China due to the GFW, you can used cdn accelerate.
2. use `git config --global user.name "XXX" ` and `git config --global user.email "XXX` to configerout the global username and git email address.
3. use command line `ssh -keygen -t ras -C "YOUR E-MAIL ADDRESS"` to generate your public and personal key. Just keep typing `Enter` if anything pop up on the comand window.
4. On your gitLab profile setting page, upload your ssh-key public simply by copying and pasting from your `id_ras.pub` file.
5. Some tips to work with git control :
    * create new dir that you would like to put the project.
    * `git clone git@XXXX` to clone the project from the gitlab to your local dir.
    * `git branch "name your new branch"` to create a new brach from master locally.
    * `git switch "new branch"` or `git checkout "new branch"` to switch to the new branch.
    * `git push origin "new branch"` push the new created branch to the master, this will not merge two of them.
    * `git --set-upstream-to origin/new Branch` will set up the new branch to track of the original master branch.
     
```

# Upload new hexo article for my blog

```
    1. `cd` to the dir that store the hexo file (On my mac it is : `cd Github/BlogSakuraTheme/`.
    2. `hexo new "Name_of_your_new_article"`
    3. `hexo generate`
    4. `hexo serve` give temporary link to checkout the blog
    5. `hexo deploy` deploy the new blog online.
```

