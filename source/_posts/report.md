---
title: 个人博客配置过程记录
date: 2021-05-20 22:42:37
tags:
categories: 
- 各种配置
---
# 个人博客的配置和搭建过程

## 一、前言

​	之前就萌生过搭建个人博客的想法，只是因为自己技术能力的不足就一直拖延着，直到最近查资料的时候发现了github pages和hexo这两个宝藏之后，又看到大佬们漂亮的博客，一咬牙，就开始捣鼓怎么搭建一个自己的个人博客。上网查了挺多资料，其实很多都不太一定正确，中途踩了很多坑，为了方便朋友和自己以后再次配置，故记录下来配置过程。

## 二、使用的工具和简介

### 1.hexo

​	Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

### 2.git

  Git是目前世界上最先进的分布式版本控制系统，可以有效、高速的处理从很小到非常大的项目版本管理。我们这次个人博客的搭建就是基于github pages。

### 3.nodejs

​	*Node.js* 是一个基于 Chrome V8 引擎的 JavaScript 运行环境。*Node.js* 的包管理器 npm,是全球最大的开源库生态系统。我们所使用到的hexo，就是基于nodejs编写的，需要使用到nodejs的包管理器npm。

## 三、工具的安装及配置

### 1.git

  git的安装网上已经有很多，安装过程也没有什么坑，这里就不再赘述了，详细可以看官网的说明

{% links %} - site: #git下载与安装  owner: #git官网  url: #https://git-scm.com/downloads  desc: #git官网安装指导  color: "#9b5b8b" {% endlinks %}

  或者查看廖雪峰老师的教程：

{% links %} - site: #git  owner: #廖雪峰  url: #https://www.liaoxuefeng.com/wiki/896043488029600/896067074338496  desc: #廖雪峰git教程  color: "#9b5b8b" {% endlinks %}

==注意== 

  安装好git之后，还需要进行用户名、邮箱、ssh的配置

#### 1）配置用户名、邮箱

```text
git config --global user.name "yourname"
git config --global user.email "youremail"
```

yourname对应你的用户名，youremail里对应你的github邮箱

使用以下两条命令可以检查你有没有输入正确的用户名和邮箱

```text
git config user.name
git config user.email
```

![git-userconfig](git-userconfig.png "像我这样查看")

#### 2）配置github的ssh key

还是在git的命令行，输入这个命令来产生ssh-key：

```text
ssh-keygen -t rsa -C "youremail"//youremail换成你的github邮箱
```

生成过程中它会问你生成的路径之类的，通常默认就好（一路回车）

如果成功的话应该是这样的：

![git-ssh1](git-ssh1.png "网络图片，出处(https://www.cnblogs.com/hafiz/p/8146324.html)")

它会提示你公钥生成的位置（上图中的“your public key has been saved in xxx”），打开这个公钥文件（.pub后缀）==复制里面的内容到github中个人资料->ssh keys->new ssh keys->key里面，点击添加，即可添加成功。

```text
ssh -T git@github.com//执行这条命令看有没有成功配置ssh
```

![ssh-success](ssh-success.png "如果成功，应该像我这样")

### 2.nodejs

直接上官网下载安装就好，一般没有什么坑。

完成以后在命令行输入以下命令查看是否安装成功

```text
node -v//Vxx.xx.x
npm -v//Vxx.xx.x
```

### 3.hexo

​	配置hexo比较的简单，配置过程中遇到问题参见hexo官方文档。

```
$ npm install -g hexo-cli
$ hexo -v//版本，能查看表示安装成功
$ hexo init 文件夹的名字
$ cd 文件夹的名字
$ npm install //安装依赖

$ hexo g //生成静态页面
$ hexo server //这个时候就可以在localhost:4000查看自己的博客雏形了
ctrl+c将服务关闭
```

## 四、github创建个人仓库

​	==前提：有一个GitHub账户==

​	如果没有，可以去官网上创建一个，然后走步骤3的git配置ssh。

### 	1.新建仓库：

![newRepos](newRepos.png)	

++创建一个和你用户名相同的仓库，在后面加上github.io++

:::info

这样部署到github pages上的时候才会被识别，如xxx.github.io

:::

![githubRepoName](githubRepoName.png)

:::warning:

必须是公开仓库

:::

## 五、将hexo部署到github Pages

### 1.修改配置文件

​	++打开我们根目录下的_config.yml，翻到最后++

```text
deploy:
  type: git
  repo: https://github.com/YourgithubName/YourgithubName.github.io.git//你的github项目地址，也可以用ssh地址
  branch: main
```

### 2.安装deploy-git

```text
npm install hexo-deployer-git --save
```

:::info

--save的意思是只在这个项目安装这个包

:::

### 3.进行部署

在根目录执行以下命令

```text
hexo clean
hexo generate//也可以是hexo g
hexo deploy//也可以是hexo d
```

:::info

Clean命令用来清除我们之前生成的文件

generate命令生成静态文件

deploy命令部署,部署时可能会需要输入账号密码

:::

成功时应该是这样的：

![deploySuccess](deploySuccess.png)

过一会你就可以在https://yourname.github.io上面看见你的博客了

## 六、设置个人域名（可选）

去阿里云，注册一个账户然后买一个个人域名，com比较贵一点，59首年，cn的便宜一些，29首年，纯粹看个人喜好。

### 1.添加解析

​	购买之后，去域名控制台，点击域名解析

![address1](address1.png)

添加如下记录![addAddress](addAddress.png)

### 2.修改GitHub pages

登录GitHub，进入我们之前的仓库->设置->pages

在custom domain中输入你购买的域名,点击save

![inputAddress](inputAddress.png)

### 3.修改博客配置文件

在博客文件的source目录下新建一个CNAME文件（==不要后缀==）内容写上你的域名

### 4.重新部署

最后，走一遍5.3中的步骤，重新部署上去，过一段时间就可以通过自己的域名访问了

:::info

而且本来国内访问GitHub巨慢的，绑定域名之后快了非常多

:::

## 七、配置主题的问题

在hexo的官方中theme中找到自己喜欢的主题，根据主题作者的教程配置就好，这里涉及到一些配置文件的配置，每个主题都不一样，就不展开说了

## 八、错误问题及解决方法

### 1.mac下npm命令报EACCES权限错误

​	处理方法见：[Resolving EACCES permissions errors when installing packages globally | npm Docs (npmjs.com)](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally)

​	不要乱用sudo覆盖权限，而是要能正常npm install 某个包才算是搞定了权限

### 2.mac下hexo使用hexo d报权限错误

​	修改文件权限，使用chown修改文件夹权限后即可。