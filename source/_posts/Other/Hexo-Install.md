---
title: 我的Hexo搭建方案
date: 2018-02-03 20:42:08
categories: Other
tags:
 - blog
---

看了别人博客目录，感觉别人成长真的迅速，跟飞一样，说真的，对自己很打击，自己的状态也不应该如此。
在曾经也懵懵懂懂学了一些东西，但是现在都回想不起具体的内容，只有刻画在脑子才能依稀想起。
所以想接下来把自己的所学或者积累和经验书写在自己的博客里面，看看自己的博客就能鞭策自己，也方便自己复习以前的知识。
当然能给到大家帮助，那是最好的。
开心就好

<!-- more -->

记录一下自己搭建博客的整个过程，方便以后自己复现。
参考了这篇文章的博客架构：[阿里云VPS搭建自己的的Hexo博客](https://segmentfault.com/a/1190000005723321)

**2018/6/24更新**
## 博客架构
![blog架构图](2018-06-24-20-27-32.png)

> 整个流程就是本地将 `*.md` 渲染成静态文件，然后Git推送到服务器的repository,服务器再通过 `git-hooks`同步网站根目录。


所以，我们需要将本地计算机配置`Hexo的运行环境`，将自己的VPS配置成`GIT服务器`和`网站服务器`。


## 本地计算机配置
### 环境部署
我的系统是`Win10`，所以直接官网下载安装包安装即可。

Hexo基于Node.js，所以我们必须安装运行环境。[Node.js Download](http://nodejs.cn/download/)
在本地我们也需要使用Git推送到VPS的Repository。[Git Download](https://git-scm.com/download/win)
 - 如果第一次使用git，可以按照[这个流程做Git的基本配置](http://www.cnblogs.com/superGG1990/p/6844952.html)
 - 注意：`~/.ssh/id_rsa.pub`文件后面会使用到

### Hexo建设
可以先看一遍[Hexo官方文档](https://hexo.io/zh-cn/docs/index.html#%E5%AE%89%E8%A3%85-Node-js)
以下配置只需要右键打开`Git Bash`逐行输入即可，当然需要注意 **Blog所在路径**
```python
# npm是随同NodeJS一起安装的包管理工具，可以通过它从npm服务器直接安装各种包
npm install -g hexo-cli # 安装Hexo
# 初始化Hexo（之后的本地上的操作都要到该目录下进行，所以慎重选择目录所在的位置）
cd /h/blog
hexo init # Hexo会在当前目录初始化

# 安装必要Hexo插件
# 之后的操作都在初始化的目录下进行执行
npm install hexo-deployer-git --save
npm install hero-server

# 设置ghexo-deployer-git参数
vi _config.yml # 在我文件最后增加以下这段,repo换成VPS的IP和对应目录即可

deploy:
  type: git
  repo: git@1.1.1.1:/var/git/blog.git
  branch: master
  message: '站点更新:{{now("YYYY-MM-DD HH/mm/ss")}}'
```
`hexo-deployer-git` 可以自动化的将本地的代码`push`到自己的VPS上，结合`git-hooks`钩子，就是在本地一打命令，VPS上的站点就更新了，贼方便
`hero-server` 就是一个服务器一样，执行 `hexo s` ，在访问给出的url，就可以看见自己博客的样子。
当然以上配置做完之后，建议`hexo s`，访问URL，查看Hexo是否正常运行。

### 主题美化
默认主题比较丑，当然需要找好看的主题啦！之前使用了[archer](https://github.com/fi3ework/hexo-theme-archer)，由于`archer`更新后变得不喜欢，所以直接换了NEXT主题
如果有选择了`NEXT`,可以参考这篇[主题美化文章](https://reuixiy.github.io/technology/computer/computer-aided-art/2017/06/09/hexo-next-optimization.html)




## 服务器建设
我VPS是`CentOS6`，所以下面的服务器安装部分会有所不同，`google`即可

### 环境部署

```python
yum install git  # 安装git
curl -sL https://rpm.nodesource.com/setup | bash - # 安装node.js
yum install -y gcc-c++ make # 安装node.js
```

### git设置
因为要把VPS变成一个Git服务器来使用，所以需要创建Git用户，并且需要将本地的公钥放到服务器中。
```python
# 创建git用户
adduser git
passwd git # 设置密码

# 设置公钥
cd /home/git/
mkdir .ssh
# 复制自己GIT公钥进入，一行一个
vi .ssh/authorized_keys # 这边所说的公钥，是本地计算机中的~/.ssh/id_rsa.pub中的内容

# 初始化git仓库，拿来当repo
cd /var
mkdir git
cd git
git init --bare blog.git # 创建一个裸库

# 设置git-hook
# git-hook的作用是当本地计算机push到VPS上时，会触发git-hook，更新nginx发布的站点
vi blog.git/hooks/post-receive # 输入下面两行，然后参数换成自己的

#!/bin/sh
git --work-tree=/var/blog --git-dir=/var/git/blog.git checkout -f

# 权限设置
chmod +x blog.git/hooks/post-receive
chown -R git:git /var/git

vi /etc/passwd # 让git用户不能shell登录

/bin/bash # 将git用户的/bin/bash替换为下面字符串
/usr/bin/git-shell

```

### 站点设置
创建站点目录，拿来发布站点
```python
mkdir /var/blog
chmod 777 /var/blog
```

### 安装nginx
```python
# 引用别人的过程
# 安装 nginx
# 第一步，在/etc/yum.repos.d/目录下创建一个源配置文件nginx.repo：

cd /etc/yum.repos.d/
vim nginx.repo

# 填写如下内容：

[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1

# 保存，则会产生一个/etc/yum.repos.d/nginx.repo文件。

# 下面直接执行如下指令即可自动安装好Nginx：

yum install nginx -y


# 配置 nginx
# /etc/nginx/nginx.conf文件中http节点下，增加以下代码
    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /var/blog;
        index index.html

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

# 重启
service nginx restart

# 访问自己站点，直接浏览器输入IP
# 部署正常的话，应该会出现nginx的翻译界面，因为博客还没有push上来
```

## 使用流程
配置完以上流程后，那么最后只剩在本地计算机中`push`渲染后的静态站点了。
到Blog目录中打开`Git Bash`，输入`hexo clean && hexo g -d`。(如果生成Git公钥的时候，不是空密码，那么会弹出输入密码界面)
如果没有问题的话，访问自己VPS的IP地址，即可看见站点正常运行。

在回顾一下流程：
根据本地计算机中的`.md`文件渲染成静态站点，在利用Hexo命令`push`到自己的Git服务器，Git服务器接收后会根据`git-hooks`的设置，自动将`push`来的站点同步到网站根目录。


## 博客编写

完成以上内容，那么你的站点已经成功的可以运行，那么需要注意的就是 **怎么去写文章**?
Hexo是使用[Markdown](http://xianbai.me/learn-md/article/about/readme.html)需要来编写`.md`文件的，Markdown语法，可以看文章学一下即可。
但是编写Markdown文章当然需要顺手的编辑器，如果你已经有很顺手的编辑器了，那么可以直接在原有的编辑器安装插件即可，顺手当然好。
如果还没有，下面推荐两个编辑器

-----------------------

在`notyeat大佬`推荐下使用`atom`本地编写，并且安装了下面插件，具体`atom`如何安装插件可以`google`一下
```python
atom-simplified-chinese-menu # 中文包，理解不了转换下看看
language-markdown # 代码高亮
markdown-preview-enhanced # 可视化+同时滚动
markdown-table-editor # 表格
markdown-image-paste # 贴图，记得把_config.yml中的post_asset_folder设置为true，new文章会自动创建一个文件夹，图片需要放在文件夹中
```
重装电脑后在安装的时候出了问题在`git bash`中执行`apm -v`会报错，在powershell或cmd可以执行。在atom中无法安装插件，报错信息中提示运行`apm install --check`,运行了就可以安装了。

**2018/6/24更新**
由于`atom`的一些问题，又在`gokoucat大佬`的推荐下用了`VS Code`。总体感觉`VS Code`上手更快，所以第一次用，推荐使用它。
同时也安装了一部分插件
```python
Chinese Language Pack # 中文包
Bracket Pair Colorizer # 花括号
Markdown Paste # 图片复制，同时也注意记得把_config.yml中的post_asset_folder设置为true，new文章会自动创建一个文件夹，图片需要放在文件夹中
Markdown Preview Enhanced # 可视化预览
vscode-hexo # F1可以直接输入hexo命令

当然还有主题、图标等等，这些按照当然按照自己来选择啦
```



## 备份Blog
因为Hexo是建设在本地的，也就是源文件在本地，如果自己电脑gg，就不好说了，所以我自己做了一步操作，就直接把源文件和主题`push`自己的Github里面，做个简单备份。

**注意**
配置文件也会暴露出来了!!!



## 问题解决
### [DEP0061] DeprecationWarning: fs.SyncWriteStream is deprecated.
Hexo 命令都会触发这个Error
[解决思路](http://rangerzhou.top/2017/07/27/Hexo%E5%8D%9A%E5%AE%A2%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9/)
遇见这个问题是因为hexo-admin中的lib库hexo-fs没更新及时。
注释掉里面的被弃用的代码即可。

### 修改文章生成的title格式
因为默认是 文章标题 · 站点名称，想把符号改成|，然后看官方配置，好像没这个，直接去他部署文件把规则改了就行。
修改`blog\themes\archer\layout\_partial\base-head.ejs`文件`title`标签下的代码。

### 更改archer样式
因为archer中的代码显示，注释有点难看清，然后我又比较喜欢写注释，就比较难受，然后就直接修改他的样式，自自己舒服就好
[archer二次开发文档](https://github.com/fi3ework/hexo-theme-archer/blob/master/docs/develop-guide.md)
文档写的很详细，但是我懒得安装编译插件，所以直接改了两处地方
`archer\source-src\scss\_partial\_code.scss`中.comment 样式
`archer\source\css\style.css` 中的`.comment` 样式
然后只把斜体改了，原本想改个亮色，但是不怎么会配色，以后看见舒服在改
**2018/6/24更新**
很多主题是提供用户自定义样式接口！！！