本文提供了基于阿里云ECS服务器环境（ Ubuntu Server 镜像）的Jekyll静态网站环境搭建过程，核心组件包括Jekyll, Nginx, Git。网站用于个人博客分享，使用Jekyll生成静态网页能够满足需求。

搭建环境：
- 系统：Ubuntu 16.04.5版本 。查看系统版本可使用如下命令
```shell
 lsb_release -a
```
-  客户端：putty或者xshell都行，都支持ssh登陆和sftp
- 安装前在root权限下通过apt update和apt full-upgrade跟新ubuntu的程序包到最新

基本思路：
1、

## 一、准备工作
参考资料：https://git-scm.com/book/en/v2/Git-on-the-Server-Setting-Up-the-Server

### 1、在服务端新建用户 tedblog (本文使用该用户名作为新用户)
```shell
adduser tedblog	#新建用户
su tedblog #切换到新建用户

cd ~ #切换到新建用户tedblog的home目录
pwd #确认当前是否在新建用户tedblog的同名目录下（ubuntu新建用户后会在home目录下新建同名文件夹）
```

### 2、在服务端添加.ssh 认证目录和认证文件
```shell
pwd #确认当前是否在tedblog的Home目录
输出为： /home/tedblog

mkdir .ssh && chmod 700 .ssh  #新建.ssh文件夹，修改权限为tedblog只读写执行
touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys #新建公钥文件，修改权限
```
### 3、在客户端生成密钥对，将公钥推送到远端
秘钥对生成借助git命令行工具来实现，git内部已集成openssl工具，也可以自行在系统安装openssl来实现。
未安装git需要先在服务端和客户端安装git。

使用git bash的好处是可以在任意平台使用ssh-copy-id命令来远程推送public RSA key，省去了手动配置的环节。

- 生成密钥对
```shell
ssh-keygen -t rsa
```

- 推送公钥到服务端
```shell
ssh-copy-id  -i  id_rsa.pub  user@serverURL
```
- 测试ssh登陆服务端
```shell
ssh user@serverURL
```


## 二、配置jekyll
jekyll作为轻量高效的静态网站生成引擎，基于Ruby开发，拥有较为丰富的插件环境，GitHub Page默认使用jekyll作为网页生成工具，托管在github.io上。

- 参考资料:  https://jekyllrb.com/docs/installation/ubuntu/

当前用户为tedblog，如果是root用户可以先切换大新建用户。不在root下安装的原因在于root安装的工具包有些无法在非root用户访问到。需要root权限的时候，随时sudo切换即可。

---
### 1、安装jekyll包和依赖

- #### 1.1 安装完整的ruby包及其常用的依赖工具，例如ruby gem包管理器

```shell
sudo  apt install ruby-full build-essential zlib1g-dev

#安装完后可以查看ruby和Gem版本信息确认安装成功
ruby -v # ruby 2.3.1p112 (2016-04-26) [x86_64-linux-gnu]
gem -v # 2.5.2.1
```

如果sudo提示 "tedblog is not in the sudoers file.  This incident will be reported." ，则需要提升当前用户的权限。常用方法有两种：在root下将用户添加到sudo/admin用户组，adduser username sudo或者adduser username admin；另一种方法为修改/etc/sudoers权限配置文件。具体方法参考:https://www.tecmint.com/fix-user-is-not-in-the-sudoers-file-the-incident-will-be-reported-ubuntu/
{: .notice--warning}

 -  #### 1.2  修改通过gem安装ruby工具包的默认路径为当前tedblog用户路径，实现方法为修改~/.bashrc文件

```shell
echo  '# Install Ruby Gems to ~/gems'  >> ~/.bashrc 
echo  'export GEM_HOME="$HOME/gems"'  >> ~/.bashrc 
echo  'export PATH="$HOME/gems/bin:$PATH"'  >> ~/.bashrc 
source ~/.bashrc
```

- #### 1.3 通过gem安装Jekyll

```shell
gem install jekyll bundler

ls ~/gems/bin #可以看到gem已安装包，说明安装路径修改成功
```

至此，jekyll的工具环境就安装完成，下面来通过Jekyll简单配置默认的博客环境，用来测试

---
### 2、搭建jekyll的测试网站
- #### 2.1 在指定路径下面新建博客站点，这里我们建在/home/tedblog下面，博客名为myblog
```shell
cd /home/tedblog
jekyll new myblog
```
- #### 2.2 开启本地测试服务器
```shell
cd myblog
bundle exec jekyll serve
```
- #### 2.3  查看jekyll serve开启服务信息

```shell
jekyll serve
```
```
#能看到以下信息
Configuration file: /home/tedblog/myblog/_config.yml
            Source: /home/tedblog/myblog
       Destination: /home/tedblog/myblog/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
       Jekyll Feed: Generating feed for posts
                    done in 0.526 seconds.
 Auto-regeneration: enabled for '/home/tedblog/myblog'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```

通过ssl远程链接配置的时候，可以配合一些http抓包工具来分析


## 三、配置Nginx服务器
Nginx是一款功能强悍的轻量web服务器程序，我将使用Nginx来为jekyll生成的静态网站提供web服务。

- ### 3.1 安装nginx
```shell
sudo apt install nginx
```
- ### 3.2 配置nginx
nginx的配置文件路径为 /etc/nginx/sites-enabled/default
```shell
sudo vim /etc/nginx/sites-enabled/default
```
修改以下一个关键地方：
> - root: /home/tedblog/myblog/_site     修改网站根目录
> - server_name: www.palelight.cn  修改网站域名




