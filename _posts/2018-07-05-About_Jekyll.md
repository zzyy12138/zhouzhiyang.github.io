---
layout: post
title: "windows下如何使用jekyll"
date: 2018-07-05 
description: "windows下使用jekyll"
tag: Jekyll  

---

### 步骤

本文只是示范了在window7环境下Jekyll的安装调试

1.安装Ruby

2.安装DevKit

3.安装Jekyll

4.启动jekyll

5.故障诊断


### 安装Ruby

1.前往[http://rubyinstaller.org/downloads/](http://rubyinstaller.org/downloads/)
2.在`RubyInstllers`部分,选择某个版本指定下载
>例如，Ruby 2.3.3-p222 (x64) 是适于64位 Windows 机器上的 Ruby 2.3.3 x64 安装包。
>

3.通过安装包安装
>1.最好保持默认安装路径`C:Ruby23-x64`,如果要修改路径的话 路径中不要出现中文
>2.勾选`Add Ruby executables to your PATH`,这样会自动添加到PATH中不用为添加路径头疼.
>

4.打开命令行输入命令检查是否安装成功
>ruby -v
>

### 安装DevKit

1.前往[http://rubyinstaller.org/downloads/](http://rubyinstaller.org/downloads/)
2.下载Ruby对应版本的DevKit安装包例如，DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe 适用于64位 Windows 系统上的 Ruby 2.3.3 x64。
3.运行安装包解压到一个文件夹路径中 推荐直接在`C:\Ruby23-x64\ `文件夹中添加一个`devkit`文件夹
4.在打开命令行窗口传递到`C:\Ruby23-x64\devkit `中输入下面命令
>ruby dk.rb init
>notepad config.yml
>

5.在打开的记事本窗口中 文末添加`- C:/Ruby23-x64 `保存并退出
6.回到命令行输入下面命令
>ruby dk.rb review
>ruby dk.rb install
>

### 安装Jekyll

1.确保gem已经正确安装,输入下面命令,会输出版本号
>gem -v
>

2.安装Jekyll和Bundler gems
>gem install jekyll bundler
>

3.确保 jekyll gem 已经正确安装,输入下面命令,会输出版本号
>jekyll -v
>

4.确保 bundler gem 已经正确安装,输入下面命令,会输出版本号
>bundle -v
>

### 启动Jekyll

具体可以参照[官方文档](http://jekyllcn.com/)使用

### 故障诊断
1.当输入`jekyll build`报以下错误的时候: (单线框和双线框分别对应包和版本号)
![错误1](/images/posts/About_Jekyll/1.jpg)
解决方法如下:(单线框和双线框分别对应包和版本号)
![解决1](/images/posts/About_Jekyll/2.jpg)

2.当输入`jekyll build`报以下错误的时候:
![错误2](/images/posts/About_Jekyll/3.jpg)
解决方法如下:
![错误2](/images/posts/About_Jekyll/4.jpg)
只需要在命令前加`bundle exec`

3....未完待续


<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [windows下如何使用jekyll](http://zhouzhiyang.cn/2018/07/About_Jekyll/) 



