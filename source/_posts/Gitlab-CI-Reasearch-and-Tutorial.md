---
title: Gitlab_CI_Reasearch_and_Tutorial
author: Liu Yuze
avatar: 'https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg'
authorLink: liuyuze.site
authorAbout: 好少年光芒万丈
authorDesc: 好少年光芒万丈
categories: 技术
comments: true
mathjax: true
date: 2020-02-12 13:28:26
tags:
 - tech
 - software
 - java
keywords:
 - CI
 - gitlab-runner
 - docker
description:
photos: https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/cyber5.jpg
---

# Gitlab CI Research and Tutorial
CI is short for Continuous Integration,是Gitlab内置的持续集成工具，只需在仓库根目录下创建.gitlan-ci.yml文件，并配置Gitlab Runner；每次提交的时候，gitlab将自动识别.gitlan-ci.yml文件，并使用Gitlab Runner执行该脚本。

Most projects use GitLab's CI service to run the test suite so that the developers get immediate feedback everytime they push or merge something.

In brief, the steps needed to have a working CI can be summed up to:
1. Add ```.gitlab-ci.yml``` to the root directory of your repository.
2. Configure a Runner

### 几个概念
#### Pipeline
一次pipeline相当于构一次任务，里面包含多个流程，比如安装依赖，运行测试，编译，部署测试服务器，部署生产服务器流程。任何提交merge request的合并都可以出发pipeline。
#### Stages
Stages表示构建阶段，也就是上文提到的流程，在一个pipeline里可以有多个流程，也就是多个stages，stages的特点如下：

* 所有stages严格按照顺序执行，当一个stage完成后，下一个stage才会开始。
* 一个pipeline所有的stages全部完成后，由这些stages构建的pipeline才会最终成功。
* 如果任何一个stage失败，后面的stage不会执行且构建任务的pipeline失败。

#### Jobs
Jobs表示构建工作，表示某一个stage里面执行的工作。一个stage里面可以定义多个jobs。jobs具有下面特点：

* 一个stage中的jobs会并执行。
* 一个stage中的jobs都执行成功后，该stage才会成功。
* 如果任何一个job失败，那么该stage失败，进而导致pipeline失败。

### GitLab Runner
#### Install
可以参考官方[安装文档](https://docs.gitlab.com/runner/install/)。这里摘录macOS的安装教程。
##### manual installation
```sudo curl --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-darwin-amd64```
```sudo chmod +x /usr/local/bin/gitlab-runner```
##### homebrew installation
Install the GitLab Runner
```brew install gitlab-runner```
Install the Runner as a service and start it.
```brew services start gitlab-runner```
#### Obtain a token
The official tutorial can be found [here](https://docs.gitlab.com/runner/register/index.html)
Go to Settings > CI/CD to obtain the token
#### Register
The register tutorial can be found [here](https://docs.gitlab.com/runner/register/index.html).
1. After install the runner, you can either register it as a shared one or a specific one. In our case we register it as a specific one.
2. Specific Runners are useful for jobs that have special requirements or for projects with a specific demand. If a job has certain requirements, you can set up the specific Runner with this in mind, while not having to do this for all Runners. Specific Runners process jobs using a FIFO queue.
3. To register under macOS:

```command
gitlab-runner register

#Enter your GitLab instance URL:
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
https://gitlab.com

#Enter the token you obtained to register the Runner:
Please enter the gitlab-ci token for this runner
xxx

#Enter a description for the Runner, you can change this later in GitLab's UI:
Please enter the gitlab-ci description for this runner
[hostname] my-runner

#Enter the tags associated with the Runner, you can change this later in GitLab’s UI:
Please enter the gitlab-ci tags for this runner (comma separated):
my-tag,another-tag

#Enter the Runner executor:
Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:
docker

```
* 打开GitLab中的项目页面，在项目设置中找到runners
* 运行 gitlab-runner register
* 输入CI URL
* 输入Token
* 输入Runner名字
* 选择Runner类型，简单起见虚选用shell
* 使用 ```gitlab-runner list``` 可以查看各个runner的状态。

### .gitlab-ci.yml
yml file is to configure what CI does with the project. It lives in the root of your repository.

```.gitlab-ci.yml``` is a YAML file so you have to pay extra extra attention to indentation. Always use spaces, not tabs.

```YAML
# 定义stages
stages:
 - build
 - test
# 定义jobs
job1:
 stage:test
 script:
  - echo "I am job1"
  - echo "I am in test stage"
# 定义jobs
job2：
 stage：build
 script：
  - echo “I am job2”
  - echo "I am in build stage"
```
### mac常用runner命令：
```
sudo gitlab-runner list
sudo gitlab-runner verify
sudo gitlab-runner verify --delete
sudo gitlab-runner run
```
### gitlab-runner with Docker
1. Docker官网下载，安装即可。
2. 在注册runner的时候选择，dock模式，就可以了。

### Install gitlab-runner and docker on Linux Server
工作时需要在公司服务器上跑CI测试，之前的安装教程是在mac os下安装的，现在把再Linux上安装的教程贴出来，踩了很多坑，以后可以避免。<br>

1. 首先查看Linux系统。<br>```sudo uname -r``` <br>公司的server使用的是 ```centos 3.10.0-1062.12.1.el7.x86_64```<br>
2. 更新环境。<br>```sudo yum update```<br>这个非常重要，开始没有更新导致执行```yum```命令的时候系统一直处于等待状态，同时```yum```命令又一直被挂起，进程用```kill```命令或者```sudo rm -f /var/run/yum.pid```都不能有效关闭，所以这里要提前更新一下。<br>
3. 卸载旧版本的docker<br>```yum - y remove docker docker-common docker-selinux docker-engine```
4. 设置```yum```源<br>```yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo 
yum-config-manager  --add-repo  http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo #阿里云yum源```<br>
1. 查看所有仓库中所有docker版本，并选择特定版本安装<br>```yum list docker-ce --showduplicates | sort -r```
2. 安装docker<br>```yum install -y docker-ce  #由于repo中默认只开启stable仓库，故这里安装的是最新稳定版17.12.0
yum install  -y <FQPN>  # 例如：sudo yum install docker-ce-17.12.0.ce```<br>
7. 启动docker并加入开机自动启动<br>```systemctl start docker
systemctl enable docker```<br>
1. ~~配置阿里云镜像加速<br>~~```mkdir  -p  /etc/docker
vi  /etc/docker/daemon.json```<br>~~添加以下内容~~<br> ```{"registry-mirrors": 
["https://5f2jam6c.mirror.aliyuncs.com", 
"http://hub-mirror.c.163.com"]
}```<br>**垃圾镜像不要用，不删留作警示！！！！！**
1. 重新加载配置文件<br>```systemctl reload  docker```<br>
2. 重启docker<br>
```systemctl restart docker```

#### 安装过程可能遇到的问题
1. 正在处理依赖关系 docker-ce-selinux >= 17.03.0.ce-1.el7.centos，它被软件包 docker-ce-17.03.0.ce-1.el7.centos.x86_64 需要 软件包 docker-ce-selinux 已经被 docker-ce-cli 取代，但是取代的软件包并未满足需求” 等一大串的问题<br>这时我们需要通过```yum install```安装一个rpm包,通过这个地址我们查看和我们安装docker版本一直的rpm包 https://download.docker.com/linux/centos/7/x86_64/stable/Packages/<br>通过<br>```yum install```<br> https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.0.ce-1.el7.centos.noarch.rpm <br>
2. 运行时```sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2```<br>卡死在：``` Loaded plugins: auto-update-debuginfo, fastestmirror, langpacks```<br>这也有可能是因为我之前没有运行```yum update```的缘故，不过也同样可以给出解决办法，就是把这几个插件的config文件修改，不启动即可，具体修改位置如下：<br>```sudo vim /etc/yum/pluginconf.d/fastestmirror.conf```<br>```sudo vim /etc/yum/pluginconf.d/auto-update-debuginfo.conf```<br>```sudo vim /etc/yum/pluginconf.d/langpacks.conf```<br>
  
### 缓存问题
关于maven工程的缓存问题，通过执行docker类型的runner，可以在配置runner的时候作如下处理，可以使得runner复用maven依赖：
```
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "mac_liuyuze_docker"
  url = "http://192.168.2.114/"
  token = "cfc078487550e3c6125d45101e2de2"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.docker]
    tls_verify = false
    image = "maven:3-jdk-8"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache","/root/.m2"]
    pull_policy = "if-not-present"
    shm_size = 0
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
```

  

### 文章参考
1. https://www.cnblogs.com/tylerzhou/p/10969072.html











