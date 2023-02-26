---
title: 如何使用Hexo在Github上搭建个人博客
date: 2021-01-30 19:50:55
tags: Hexo
---

第一篇就顺手记录下自己的Github博客搭建的过程，这里以mac环境来做说明

# 安装 Node 以及 Git
首先在电脑上安装好[Node.js](https://nodejs.org/en/)和[Git](https://git-scm.com/)

查看 Node 和 npm（Node 自带的包管理工具，需要使用它来安装 Hexo）以及 Git 是否安装完成

``` bash
$ node -v
$ npm -v
$ git version
```
使用 npm 安装 Hexo

``` bash
$ sudo npm install -g hexo-cli
安装完成后创建一个存放我们blog的文件夹，然后初始化一下
```

``` bash
$ cd ~
$ mkdir myGitBlog
$ cd myGitBlog
$ hexo init #初始化博客目录
$ hexo s    #预览 
```
接下来我们可以在 myGitBlog 的根目录找到我们这个博客项目的配置文件 _config.yml
可以配置一些我们博客的基本信息，诸如博客的网站标题、网站描述、作者、时间格式、网站地址（即我们即将部署的自己的github主页地址，比如 https://xiamengfan.github.io/ ）等等信息
更详细的安装以及配置请参阅[Hexo官方文档](https://hexo.io/zh-cn/docs/)

至此，我们已经可以在本地写博客了：

``` bash
$ hexo new "我的第一篇博客"     #生成在 myGitBlog/source/_post 目录下
$ hexo g        #生成静态网页
$ hexo server   #启动一个本地服务器
```

接下来介绍如何将我们本地的博客项目部署到自己的Github主页上

# 部署到 Github
在你准备部署到Github上之前，请先注意一些事项（尤其是正用Github管理站点目录的用户）：
- 当执行 hexo deploy（hexo d） 时，Hexo 会将 public 目录中的文件和目录推送至 _config.yml 中指定的远端仓库和分支中，并且**完全覆盖**该分支下的已有内容
- 由于 Hexo 的部署默认使用分支 master，所以如果你同时正在使用 Github 管理你的站点目录，你应当注意你的部署分支应当不同于写作分支
- 一个好的实践是将站点目录和 Pages 分别存放在两个不同的 Github 仓库中，可以有效避免相互覆盖
- Hexo 在部署你的站点生成的文件时并不会更新你的站点目录。因此你应该手动提交并推送你的写作分支

如果你已充分知晓以上注意事项或者第一次尝试使用Github管理自己的站点目录，那么我们就开始吧

## 创建你的Github站点
首先需要创建一个你的 Github 账户，并且创建一个特殊的 repository，这个 repository 的名字必须为 username.github.io (username 为你的 Github 用户名，比如我的就是 xiamengfan.github.io)
Github 官网对此的介绍为：Websites for you and your projects. 可以写一些静态页面展示给别人看，拿来写博客可以说是再合适不过了
别人只需要访问 username.github.io 就可以看见你的博客主页了，比如[我的](https://xiamengfan.github.io/)

## Git用户信息以及ssh配置
配置当前项目的 Git 用户信息
如果之前使用 –global 参数进行过全局配置，且无需更改用户信息则不用配置

``` bash
$ cd ~/myGitBlog
$ git config user.name "username"          #Git用户名
$ git config user.email "example@xxx.com"  #Git登陆邮箱
```
生成 ssh key
这里我原先已经有了一个用于公司项目的 ssh key，所以使用 -f 参数重命名生成了一个另外的 ssh key

``` bash
$ ssh-keygen -t rsa -C "email@gmail.com" -f ~/.ssh/id_rsa_myGit
```
配置公钥
输出并拷贝 id_rsa_myGit.pub 信息

``` bash
$ cat ~/.ssh/id_rsa.pub
```
进入你的Github主页，右上角个人信息处，点击 settings, 然后左侧栏点击 SSH and GPG Keys，点击 New SSH key，将拷贝的公钥信息复制，title 可随意填写

配置私钥
``` bash
$ ssh-add ~/.ssh/id_rsa_myGit
```
如果执行 ssh-add 时提示”Could not open a connection to your authentication agent”，可以先执行命令:

``` bash
$ ssh-agent bash
```
然后再重新运行ssh-add命令

修改配置文件
若本地 .ssh 目录下无 config 文件，那么创建：

``` bash
$ cd ~/.ssh
$ touch config
```
在 config 文件中添加以下内容：

``` plain
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ～/.ssh/id_rsa_myGit
    User username
```
测试ssh key 是否配置成功
``` bash
$ ssh -T git@github.com
```
输出“Hi username! You’ve successfully authenticated, but GitHub does not provide shell access.” 则代表成功了

## Hexo部署配置
``` bash
$ cd ~/myGitBlog
$ sudo npm install hexo-deployer-git --save
```
然后修改根目录配置文件 _config.yml 中的 deploy 部分：

``` plain
deploy:
  type: git
  repo: git@github.com:username/username.github.io.git
  branch: master
```
ok，大功告成，现在可以很方便地将Github作为你的博客部署站点了：

``` bash   
$ hexo g                        #生成静态网页
$ hexo d                        #部署到 github
```
生成静态网页和部署可以简写为一条命令：

``` bash
$ hexo g -d
```
更多 hexo 命令可参阅[官网](https://hexo.io/zh-cn/docs/commands)

# 安装博客主题
Hexo 支持各种各样的[主题](https://hexo.io/themes/)配置，在Github上已经有很多非常好看的主题项目可以下载，不同主题的配置方式也略有不同，可以查看各自的项目说明进行安装配置
这里以我安装的 hexo-theme-melody 为例

首先确认自己安装的Hexo版本，5.0上下安装此主题方式不同

``` bash
$ hexo -v
首先进入自己博客的工作目录进行安装
hexo版本 >= 5.0
```
``` bash
$ npm install hexo-theme-melody
hexo版本 < 5.0
```
``` bash
$ git clone -b master https://github.com/Molunerfinn/hexo-theme-melody themes/melody
```
设置 _config.yml
首先确认是否安装了 pug 以及 stylus 的渲染器插件
``` bash
$ npm ls --depth 0
```
如果没有，请下载安装：

``` bash
npm install hexo-renderer-pug hexo-renderer-stylus --save
```
安装完成后在工作目录下找到 _config.yml 文件，找到theme这行修改为 theme: **melody**

至此，主题已经配置完成，可以打开自己博客主页查看效果了，更多配置请参阅：[hexo-theme-melody官方文档](https://molunerfinn.com/hexo-theme-melody-doc/zh-Hans/quick-start.html)
