---
title: 'How I Built This: vultr+hexo+nginx 建站记录'
date: 2021-02-16 12:44:04
excerpt: A brief record of building the blog site. Based on Vultr cloud compute, utilizing nginx, git, and hexo
---

这下大概是把博客网站搭好了，折腾挺久，作为第一个post，稍微记录一下过程。

#### 1. Cloud Compute选购

物理位置在大陆的云主机，用户响应时间会短一些，但是价格普遍挺高，而且腾讯云和阿里云的学生优惠感觉好复杂。真正劝退的是之后的认证备案... 所以我还是使用最最熟悉的vultr吧。

vultr的地址选在亚洲，JP Tokyo，Server Size是1 CPU，1024MB Memory，1000GB Bandwidth。系统是CentOs 8 (x64)。这台主机本来是用来跑vmess的，但是nginx的静态服务好像不很耗资源，那就一起来吧。毕竟现在流行的各种带域名伪装的ladder，都是这么一套。

#### 2. 设置nginx

（安装vmess后nginx似乎已经在那儿了，一键脚本都干了什么...）

要安装，先运行

```bash
sudo yum install yum-utils
```

安装前置需求。之后运行

```bash
sudo yum install nginx
```

注意在Ubuntu下，需要使用apt。

运行时，nginx有一个master进程和若干个worker进程，master负责全局调控，而worker负责请求的执行。

使用nginx服务静态网页，需要关心的内容只有两个：

- 通常在`/etc/nginx/`下的配置文件`nginx.conf`，此文件会被master进程读取，如果语法正确的话，会根据其中的命令对整体进行调度。对于静态网页来说，需要设置的就是html文件的位置。
- 选定一个位置作为网页的部署地点。例如`/home/Kincaid/blog`。也就就是`nginx.conf`里设置的html文件的位置，或者说根目录。

`nginx.conf`的具体格式与设置请看[这里](http://nginx.org/en/docs/beginners_guide.html#static)。

之后运行命令：

```bash
nginx -c /etc/nginx/nginx.conf # 用于指定配置文件位置
nginx -s reload
```

来启动nginx。注意nginx的运行用户必须拥有此位置的读取权限，使用root用户启动这些进程是不错的选择。

现在，如果在部署地点添加任何静态文件，如index.html，都可以通过浏览器，使用ip地址进行访问。

#### 3. 搭建Git仓库

在远程主机上搭建Git仓库，并写好Git Hooks，就可以将本地push到云主机仓库的内容，自动按照hooks里的脚本指示完成一些动作。

对于博客网站来说，hooks的动作无外乎是从git库中将最新的静态html拷贝到部署地点，这样nginx就可以服务最新的内容了。

首先安装git，`sudo yum install git`。

添加名为git的用户，并转到git用户：

```bash
adduser git
passwd git # 设置git用户的密码
su git # 以git作为当前用户
```

注意Ubuntu下密码不需要显示设置，在创建新用户后会直接要求输入密码。

在`/home/git/`中新建`.ssh`文件夹，在其中新建文件`authorized_keys`，内容是本地的公钥，与`id_rsa.pub`一致。这样本地主机就可以通过ssh使用git用户的权限连接到云主机上。

在git用户有权限的地方，新建`repo.git`文件夹，作为git库。进入其中执行

```bash
git init --bare
```

建立裸仓。

此时在本地gitbash，新建文件夹并进入，执行

```bash
git clone git@_ip_address_:/home/git/repo.git
```

即可clone一个空仓库到本地，本地也可以push内容到远程。

最后配置git hooks，在`repo.git/hooks`中，建立文件`post-receive`，内容如下：

```bash
echo BEGIN HOOKS
git clone /home/git/repo.git /home/kincaid/blog
# 将push上去的内容clone一份到部署地点
echo END HOOKS
```

注意`/home/kincaid/blog`必须是git仓库，且git用户必须拥有读写权限。建议将部署地点设置在git用户的目录下。

最后，执行如下命令赋予hooks执行权限。

```bash
chmod +x post-receive
```

#### 4. 本地hexo

本地hexo需要注意的地方只有一个，那就是`_config.yml`中的部署选项：

```yaml
deploy:
  type: 'git'
  repo: git@_ip_address_:/home/git/repo.git
  branch: master
```

每次发布新文章，执行下面的命令：

```bash
hexo new "post title" # 创建新post
hexo g -d # 生成静态文件，并部署（push到远程git库中）
```

END