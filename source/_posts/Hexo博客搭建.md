---
title: Hexo博客搭建
date: 2017-04-30 16:39:01
tags: 
- Hexo
- Github
- Git
- Nodejs
categories: 博客搭建
---
## 摘要
网上已经有很多使用 ***Hexo*** 搭建博客的教程了，这些教程都是非常详细的。我本人也是按照这些教程搭建出属于自己的博客，不过一方面实际操作中会因为生产环境的不同，搭建博客会遇到很多坑，一方面也是自己这几天学习的总结，还是在这里总结并分享自己的经验，毕竟从同是小白的我眼中更容易关注到新手会遇到的问题。首先是基于 ***Hexo+Github*** 的方式搭建。
![Hexo+Github](http://ogvm9vr0q.bkt.clouddn.com/blog/20170430/hexo.png)
<!-- more -->
## 准备
搭建博客用到的技术有：
-   Nodejs
-   Git
-   Markdown

如果只是实现博客的搭建，不了解这些技术问题也不是很大，只需要了解就行。Hexo博客是基于Nodejs的，简单理解就是生产环境，Git作用是代码管理以及后续使用Github部署；Markdown则是用来写博客的，非常简单。当然为了更好的学习，前两项是基础，我还得继续深入学习。
## 本地博客
### Git安装
从[这里下载](https://git-for-windows.github.io/ "Git下载")windows版Git，一路默认安装即可。软件自带命令行工具 ***Git Bash*** 和GUI界面（鼠标右键菜单）。
-   [Git安装教程](https://jingyan.baidu.com/article/020278117cbe921bcc9ce51c.html "Git安装")
-   [Git学习教程](http://iissnan.com/progit/ "Pro Git")

![右键菜单](http://ogvm9vr0q.bkt.clouddn.com/blog/20170430/git_menu.png)

安装后可以鼠标右键使用Git Bash输入以下命令查看Git版本测试是否正确安装
```bash
git --version
```
还可以设置自己的信息
```bash
git config --global user.email "username@xx.com"  # 用户邮箱
git config --global user.name "username"          # 用户名
```
>Git每次commit提交时，要记录提交人身份信息，包括用户名`name`和邮箱`email`。因此需要使用以上命令设置全局的用户信息。用户信息只是表示身份的字符信息，并不需要验证，与下文的Github注册邮箱也可以不一样。这里的信息可以理解为昵称。

### Nodejs安装
从[官网下载](https://nodejs.org/en/ "官网下载")LTS版本，或者[中文官网](http://nodejs.cn/ "中文官网")下载最新版，一路安装即可。
-   [Nodejs安装教程](http://www.runoob.com/nodejs/nodejs-install-setup.html "Nodejs安装教程")

为了在任何目录下使用命令行，需要配置环境变量。一般windows下默认会配置环境变量，安装参考教程。安装完毕后，在命令行中（win+R,输入“cmd”或者使用 ***git bash***）查看安装node版本
```bash
node -v
```
如果输出如下面等字样说明安装正确
```bash
v6.9.4
```
### Hexo本地部署
#### 安装Hexo cli
Hexo是Nodejs的一个包或一个模块，因此使用`npm`命令安装。任意目录下鼠标右键选择`Git Bash`，安装Hexo命令行工具
```bash
npm install -g hexo-cli
```
> npm是Nodejs的包管理器，安装包会从官方镜像下载，速度非常慢。安装 ***Hexo*** 需要用到这一命令，因此需要切换国内镜像，提高下载速度。我们可以使用 ***cnpm*** 代替npm，或者安装 ***nrm*** 镜像管理工具。

-   [cnpm](https://npm.taobao.org/ "cnpm")
-   [nrm](https://cnodejs.org/topic/5326e78c434e04172c006826 "nrm")

这里我们使用最简单的方法，设置安装包的镜像来源为淘宝镜像，这样以后安装包均是从淘宝镜像下载
``` bash
npm config set registry https://registry.npm.taobao.org
npm install -g hexo-cli
```
#### 创建本地项目
Hexo安装完成后，要生成一个本地Hexo项目，名字叫`blog`，执行以下命令，Hexo会在指定文件夹中创建博客需要的文件
```shell
# 创建blog目录，并初始化hexo项目
hexo init blog
cd blog
# 根据package.json，安装hexo依赖包
npm install
```
启动本地服务器预览
```shell
hexo server
```
或使用简写
```shell
hexo s
```
正确输出
```bash
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```
Hexo默认端口是4000，在浏览器打开`localhost:4000`可以看到预览效果如下：
![Hexo预览图](http://ogvm9vr0q.bkt.clouddn.com/blog/20170430/demo.PNG)

> 一般在这里最容易出现的坑就是4000端口被占用，导致浏览器始终无法显示正确结果。如果不能正确预览，可以试试使用`-p`参数指定其他端口

```bash
hexo server -p 5000
```
或者使用命令查找占用4000端口的进程
```bash
netstat -aon|findstr "4000"
```
最右列是进程ID，可以看到PID为15112和19640。
![查找端口](http://ogvm9vr0q.bkt.clouddn.com/blog/20170430/port.PNG)

调出任务管理器查看进程可以看到分别是福昕阅读器和chrome占用，关掉福昕进程重新启动服务能正常显示

![任务管理器](http://ogvm9vr0q.bkt.clouddn.com/blog/20170430/PID2.PNG)
#### 创建新文章
```bash
hexo new "First Post"
# 启动本地服务，查看效果
hexo server
```
使用`new`创建新文章，在/source/_posts目录下就生成了一个`First Post.md`文件。`.md`表示markdown文件，可以使用Sublime Text等工具编辑，或者使用MarkdownPad2等专门的软件编写。
Hexo的其他用法以及主题更换可以参考官方或中文文档

-   [Markdown入门](http://www.jianshu.com/p/1e402922ee32/ "Markdown入门")
-   [Hexo中文](https://hexo.io/zh-cn/ "Hexo中文文档")

#### 生成静态文件
使用Hexo的静态编译命令
```bash
hexo generate   # 或使用简写 hexo g
```
这时候会在blog目录下生成一个public目录，这里面就是编译后的静态文件。这里面就是我们的博客页面，可以直接本地用浏览器打开里面的html文件。后续要做的只是找一个服务器托管这个文件夹。
## 部署到Github
本地部署成功了，我们使用Github提供的免费服务Github Pages托管我们的博客，同时也可以管理我们的代码。
### 注册Github账号
首先我们需要一个Github账号，进入[官网](https://github.com/ "官网")注册账号。下文用`username`代替注册的用户名。
### 创建项目仓库
点击主页上的[new repository](https://github.com/new "新建仓库")创建一个新的仓库，这个仓库是用来管理托管我们博客静态文件。
>Github中每个账号都拥有一个个人主页，这个主页只能是静态文件，并且通过一个特殊的仓库来存放，这个特殊的仓库名字必须是**username.github.io**。这个仓库的托管的网站，可以直接通过**username.github.io**访问，也可以绑定自己的域名。
>此外，每个仓库可以创建一个gh-pages分支，该分支原来是用来作为该仓库项目静态文件预览的，默认通过**username.github.io/projectname**。也就是说如果我们新建一个**blog**仓库，再创建gh-pages分支托管public文件夹下面的静态文件，那么在浏览器访问**username.github.io/blog**也能访问我们的博客。

我们使用第一种方式，如下图(*图片来自网络*)，创建好自己的仓库。

![创建仓库](http://ogvm9vr0q.bkt.clouddn.com/blog/20170430/repoCreate.png)

-   [Github pages](https://pages.github.com/ "Github pages")

### 配置SSH Key
用Github管理项目时，可以使用https和SSH两种协议方式，为了更好的体验每次pull、push不需要验证，配置好SSH Key后Github通过公钥对用户授权。

> Linux/Mac系统在 `~/.ssh` 下，win系统(我的是win10)在`c/user/username/.ssh`下
使用Git Bash查看ssh文件
```bash
ls ~/.ssh
```
查看是否已经有id_rsa.pub和id_rsa两个文件，id_rsa.pub是公钥，id_rsa是私钥。如果有可以跳过下一步生成SSH Key,或者直接生成覆盖。
使用`sh-keygen`命令生成Key
```bash
# 生成SSH Key
ssh-keygen -t rsa
```

> 回车确认后，接着会提示输入两次密码，该密码是私钥的密码，在使用密钥的时候需要使用，如push的时候需要输入该密码。如果填写了，请记住。也可以一路回车，则默认密码为空。

```bash
Enter passphrase (empty for no passphrase):
# Enter same passphrase again:
```
接着复制公钥id_rsa.pub的内容，使用编辑器或者命令：
```bash
clip < ~/.ssh/id_rsa.pub
```
登录Github账号，从用户的Setting -> 菜单栏SSH and GPG Keys -> New SSH Key 进入，粘贴复制的key添加成功。有多台电脑可以使用同样的方法添加多个，并且可以在`title`栏填写区分

![SSH Key](http://ogvm9vr0q.bkt.clouddn.com/blog/20170430/ssh.png)

配置后可以输入以下代码测试
```bash
ssh -T git@github.com
```
如果最后输出一下内容表示成功配置了
```bash
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```
> 如果设置SSH Key的时候设置了密码，中间也需要密码

### 部署
终于到部署这一步了，是不是很期待呢？
在blog项目中的_config.yml文件是配置文件，我们按以下方式填写
````bash
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
# 仓库名字 
  repo: git@github.com:username/uername.github.io.git
  branch: master
````
> repo填写Github创建的仓库名字，点击仓库页面的**Clone or download**处复制SSH形式url

![仓库地址](http://ogvm9vr0q.bkt.clouddn.com/blog/20170430/repo_url.png)

配置好后执行部署命令：
````bash
hexo generate # 生成静态页面，或者hexo g
hexo deploy   # 部署到Github，或者hexo d
````
如果出现下面的提示表示成功
````
INFO Deploy done: git
```
这时候浏览器访问`uername.github.io`，就可以访问自己的博客了。到此博客就算基本搭建好了。
>如果出现404页面可能是因此第一次部署有延误，稍等一会就好了；如果报错，那么很可能就是配置文件的冒号 `":"` 后面少了空格

### 绑定自己的域名
如果有自己的域名(或者自己去各大云等购买域名)，在项目下的`source`文件夹新建一个`CNAME`文件（没有后缀），添加自己的个人域名，如`mydomain.com`，然后重新生成静态文件并部署

```bash
hexo g
hexo s
```
最后在你购买域名的xx云的控制台处找到你的域名点击解析，添加一条记录，记录类型选择`CNAME`，主机记录就是域名前缀，根据需要选择，如果想通过`mydomain.com`直接访问，那么填写`@`即可，想通过`blog.mydomain.com`访问就填写`blog`。其中记录值填写我们的github博客域名`"uername.github.io"`。最后如图添加记录，直接在浏览器里输入`"blog.mydomain.com"`就能直接访问我们的博客了。

![CNAME](http://ogvm9vr0q.bkt.clouddn.com/blog/20170430/host2.png)

## 结束
以上就是简单的使用Hexo搭建博客的过程，限于篇幅，暂时写到这里。后面还涉及到云服务器的使用，博客源码的管理以及自动部署问题。第一次写博客，而且自己水平有限，如果有错误，请指出。