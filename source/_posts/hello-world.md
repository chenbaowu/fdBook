---
title: Hello Hexo
date: 2017-05-07 17:07:07
categories: 多学
tags: hexo
---

1.下载安装[Git](https://git-scm.com/downloads)（一路next）
2.下载安装[Node.js](https://nodejs.org/en/)（一路next）
3.下载hexo： 命令行 输入
``` bash
$ npm install -g hexo-cli
```
<!-- more -->

4.验证软件是否成功安装
``` bash
$ git --version
$ node -v
$ npm -v
$ hexo –v
```
5.新建GitHub版本库 
注意工程名字一定是你的githua的名字 + github.io，也是你博客的域名

6.配置ssh
（1）设置Git的user name和email：
``` bash
$ git config --global user.name "chenbaowu"
$ git config --global user.email "1534598088@qq.com"
```
（2）生成密钥
查看是否已经有了ssh密钥：cd ~/.ssh
如果没有密钥则不会有此文件夹，有则备份删除
生成密钥：
``` bash
$ ssh-keygen -t rsa -C “1534598088qq.com”
```
按3个回车，密码为空。
最后生成生成一个目录.ssh，里面有两个文件：id_rsa , id_rsa.pub；在github的Settings的SSH and GPG keys点击new ssh key 把 id_rsa.pub的内容复制进去就可以了

7.初始化hexo
（1）在你的电脑新建一个文件夹用来当hexo的工程目录，在里面右键打开
git bash，输入
``` bash
$ hexo init
```
安装生成器 ：
``` bash
$ npm install
```
然后就可以新建文章了
``` bash
$ hexo new name
```
8.运行和部署hexo
（1）本地查看 ：
``` bash
$ hexo g && hexo s –p 5000
```
浏览器输入localhost:5000,就可以在本地看到你的个人博客了 
（2）上传git：
下载插件：
``` bash
$ npm install hexo-deployer-git --save
```
修改工程目录下的_config.yml文件
#发布设置
deploy: 
 		type: git
  		repository: git@github.com:chenbaowu/chenbaowu.github.io.git
  		branch: master
运行 ：
``` bash
$ hexo g && hexo d
```
浏览器输入 http://chenbaowu.github.io/  就可以访问了

9.主题更换
（1）在工程目录下载主题 $ git clone https://github.com/litten/hexo-theme-yilia.git    themes/yilia
（2）修改工程目录下的_config.yml文件 ：      theme:   yilia

10.添加分类和标签
md文章头直接加就可以了
categories: fdbook
tags: fdbook

11.如何显示图片
``` bash
（1）本地：在工程目录下的source下新建文件夹images，使用例子： ![](/images/fd.jpg) 
（2）网络：![图片描叙](网路地址)
```

12.修改网站小图标
在主题目录下修改_config.yml文件 ：favicon: /images/favicon.jpg

13.截取文章显示
（1）在主题目录下修改_config.yml文件 ：excerpt_link: more
（2）在文章需要截取的地方加上 
``` bash
 <!-- more -->
```
14.加访客统计和文章统计

[参考链接](https://crane-yuan.github.io/2016/03/25/Hexo-05-add-site-statistics/)

15.评论功能
!index 表示非主页中才显示

[参考链接](http://moxfive.xyz/2016/01/02/hexo-comments/)

    
