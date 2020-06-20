---
layout: post
title: github+jekyll构建自己的博客
categories:
- software
tags:
- git jekyll
- 
---
###jekyll+github建立博客过程

1.环境windows7

2.安装railsinstaller-3.0.0.exe 包含了Rails4.1.8 ruby2.0.0 git version 1.9.4.msysgit.2

一键安装完成，还是很清爽的。

3.配置git和github的连接，在gitbash 中输入自己的帐号后会生成一个ssh

在github setting中的key填入就可以连接了。

4.安装jekyll gem install jekyll

5.安装一个gem包，gem install rdiscount

6.下载了一个模版（https://github.com/kejinlu/kejinlu.github.com），删除以前博主的文章，

把discuz的shortname换成自己的账户（这个需要在disguz上面申请shortname）

7.开始写自己的博客

8.对了，写的博客记得保存为以UTF_8无BOM格式编码，不然会有中文格式问题

9.jekyll serve启动本地服务器查看

#10.强调第8条，使用notepad++进行编辑就可以了，当中有把格式改成UTF_8无BOM格式编码的功能。

11.markdown可以在图片7牛上面生成外部链接再加上那个，还有代码前面加上tab键就可以成了代码块样式了。

	img src="http://7xrmn9.com1.z0.glb.clouddn.com/project121.png" style="width: 50%; height: 50%"

12.点击链接的方法【链接名字】（链接）

附上一些常用命令

    $ git clone git@github.com:username/username.github.com.git //本地如果无远程代码，先做这步，不然就忽略
    $ cd .ssh/username.github.com //定位到你blog的目录下
	$ git pull origin master //先同步远程文件，后面的参数会自动连接你远程的文件
	$ git status //查看本地自己修改了多少文件
	$ git add -A 更加好 git add . //添加远程不存在的git文件  
	$ git commit -a更加好git commit * -m "what I want told to someone"
	$ git push origin master //更新到远程服务器上
	
13.每次添加或者修改程序之后要运行jekyll serve  编译整个程序

14.为了这个错误熬到了凌晨3点 如果文档命名的时间在系统时间之后上传到reposity中，但是博客中不会显示该博客的内容