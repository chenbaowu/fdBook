---
title: git
date: 2017-05-13 14:47:06
categories: 多学
tags: git
---

1.下载安装[Git](https://git-scm.com/downloads)（一路next）
2.设置Git的user name和email：
``` bash
$ git config --global user.name "chenbaowu"
$ git config --global user.email "1534598088@qq.com"
```
<!-- more -->

3.生成密钥
查看是否已经有了ssh密钥：cd ~/.ssh
如果没有密钥则不会有此文件夹，有则备份删除
生成密钥：
``` bash
$ ssh-keygen -t rsa -C “1534598088@qq.com”
```
按3个回车，密码为空。
最后生成生成一个目录.ssh，里面有两个文件：id_rsa , id_rsa.pub；在github的Settings的SSH and GPG keys点击new ssh key 把 id_rsa.pub的内容复制进去就可以了

4.初始化git,创建本地仓库
``` bash
$ git init
```
5.将文件加入暂存区
``` bash
$ git add . //将目录下所有文件加入暂存区
$ git add my_file,other_file //将目录下所有文件加入暂存区
```
6.提交本地仓库
``` bash
$ git commit -am "初次提交"
```
7.推送到远程仓库
``` bash
$ git remote add origin git@github.com:用户名/项目名.git  // 添加远程仓库 origin 
$ git push origin master  
```
8.实用命令
``` bash
$ git clone git@github..com:用户名/项目名.git // 克隆github的代码
$ git remote add origin git@github.com:chenbaowu/fdBook.git // 添加远程仓库地址
$ git remote rm 远程仓库名 //  删除远程仓库
$ git remote -v // 查看当前远程仓库地址
$ git fetch origin master  // 取回origin的master分支
$ git merge origin master //将origin merge 到 master 上
$ git push -u origin master // 客户端首次提交 ，以后直接 git pull ，相当于 fetch 加上 merge
& git status  // 查看状态
```
