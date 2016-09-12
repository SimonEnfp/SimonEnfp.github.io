---
title: Hexo+Github搭建个人博客
date: 2016-05-09 15:19:45
tags:
categories: 技术分享
---
## hexo + github搭建属于自己的博客
*前言*：拥有一个属于自己的技术博客，在这里可以展现你自己的个性，发挥你自己的优点，可以做一些技术分享，作为一个程序员，能够养成一个常写技术博客的习惯，乐于分享，不仅能让你自己的技术更加精湛(很多技术或许你会，但当你能把别人说会时，理解绝对是更深一层)，久而久之，逼格档次O(∩_∩)O，好了，废话不多说，进入正题
### 1.github账号申请与创建仓库

 - 账号的申请我就不多说了，相信大家都OK，这里着重说一下创建仓库以为搭建我们的博客备用。
 - 进入代码库创建页面，你的仓库名**必须是** *yourname.github.io*,这里的yourname就是你github的用户名(形如yourname.github.io这样的可访问的网站，每个用户名下面只能建立一个)，其他的默认就行了。如图:
 ![create-repo][1]
完成后等待备用。
### 2.必备软件安装

 - [git][2] 安装(一路下一步即可)。 
 - [Node.js][3]安装(一路下一步即可，环境变量会自动配置)。
 - 完后可以通过命令行查看下安装是否成功
 - 打开命令行分别输入：
    `node -v`
    `npm -v`
    `git --version`

    如果出现如下所示，说明安装成功：
![版本号][4]

### 3.安装配置Hexo：
在你自己喜欢的地方创建一个文件夹Hexo(名字随意),然后进入这个文件夹，在此文件夹内打开命令行窗口(在文件夹空白处按住Shift+鼠标右键，然后点击在此处打开命令行窗口)。
在命令行输入：
    `npm install -g hexo-cli`
这里会出现一些安装信息，等待完成即可

 - 完成后输入hexo -v如下：
![hexo版本信息][5]
 
 - 接下来我们可以初始化hexo了,输入:
    `hexo init`
这是下载一些的文件，完成后再看我们的这个Hexo文件夹，与之前比多了很多文件。
 - 然后再输入：
    `npm install`
npm将会自动安装一些需要的组件。
 - 这时候可以体验下本地搭建的博客，输入:
    `hexo g`(hexo g是hexo generate的缩写，生成文件)
    `hexo s`(hexo s是hexo server的缩写，本地预览效果)
这时候你到浏览器输入localhost:4000，就可以看到你刚刚搭建博客的效果了。但是很多信息都不是我们自己的，下面通过修改站点的_config.yml文件改一些配置
 - 打开Hexo下_config.yml文件
```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/
# Site
title: simonenfp的博客 #网站标题
subtitle:
description: 痛点就是起点 #侧边栏的描述信息
author: simonenfp #网站作者
language: zh-Hans #站点语言
timezone: #默认系统时间
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://yoursite.com #可以绑定域名(我暂时没买)
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false #新建一篇博客的时候是否同时创建资料文件夹
relative_link: false 
future: true
highlight:  #代码高亮设置
  enable: true #是否启用
  line_number: true #是否显示代码的行号
  auto_detect: false
  tab_replace:

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10 #每页显示多少篇文章
pagination_dir: page

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: landscape

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/simonenfp/simonenfp.github.io
  branch: master
```

 - 强调一点比较容易出错的地方：

    deploy:
      type: git
      repo: https://github.com/simonenfp/simonenfp.github.io
      branch: master

_对于没接触过yamllint很容易入坑，你可以通过[yamllint][6]来保证yaml语法的正确性，这里的所有 : 后一定要有个空格，type，repo，branch前必须两个空格(这个相信大家应该能明白),反正千万不要忽略该有的空格，否则各种报错，本人就在此踩了很多坑，repo后是你一开始创建的github仓库地址_

 - 修改完之后我们这时候可以把我们的本地站点部署到github上了:
这里先执行一句话:
`npm install hexo-deployer-git --save`(因为在Hexo 3.0 后server被单独出来了，需要安装server)
 - 然后输入:
 `hexo d`(hexo deploy的缩写)
随后按照提示，输入Github的账号密码，等待上传。
这样你就可以通过http://yourname.github.io/来访问自己刚刚上传的网站了。
 - 发布一篇新文章:
 输入：`hexo n "我的第一篇博客"`(我的第一篇博客为文章的标题)，执行了此命令后你会发现在Hexo/source/_post中生成一个我的"第一篇博客.md"文件，用编辑器打开编写即可，在此推荐一个编辑器[MarkDown][7]用于编写网站，关于MarkDown语法介绍有很多，在此再推荐一个[MarkDown语法介绍][8]
 - 然后输入
hexo g(生成文件)
hexo s（可以先本地预览确认无误后上传）
hexo d(上传站点)
即可完成发布

__当然这时你会发现，你的界面看起来很low，并不是像别人博客那样看起来那么让人赏心悦目，别着急，这个平台是属于你自己的，你可以做主，下面介绍一下主题的配置__。

### 4.博客主题配置

 - 你可以在[***这里***][9]选择你喜欢的主题，你可以点击预览，仔细挑选，本人选择的是***[Next][10]***这一款。下面介绍一下配置方法：
 1.我们把主题包下载下来后，放置到Hexo/themes下，你会发现除了我们刚下载的主题，之前还有一个landscape，这是默认的主题。
 2.更改我们自己站点_config.yml中的theme字段，设置为刚刚下载的主题包名next,即可使用此主题，详细配置请看[next主题使用文档][11]


 
### 5.结束语

 - hexo还支持很多第三方，可以查看[官方文档][12]去了解，我在自己博客里集成了[多说][13]，感兴趣的可以尝试，至于发表博客插入图片，本人也为此纠结很久，最后觉得[七牛][14]还不错，注册就送很多流量与存储空间。
 - 至此，搭建一个属于自己的简易博客应该没问题了吧，当然这里面还有很多功能很多特色，恕小弟才疏学浅，无法面面俱到，有写的不好的地方，希望大家能一起多多交流，分享学习才是宗旨！

 

 


    


  [1]: http://7xtufe.com1.z0.glb.clouddn.com/create-repo.png
  [2]: https://git-scm.com/downloads
  [3]: https://nodejs.org/en/
  [4]: http://7xtufe.com1.z0.glb.clouddn.com/v.png
  [5]: http://7xtufe.com1.z0.glb.clouddn.com/hexo-v.png
  [6]: http://www.yamllint.com/
  [7]: https://www.zybuluo.com/mdeditor
  [8]: http://wowubuntu.com/markdown/basic.html
  [9]: https://github.com/hexojs/hexo/wiki/Themes
  [10]: https://github.com/iissnan/hexo-theme-next
  [11]: http://theme-next.iissnan.com/
  [12]: https://hexo.io/
  [13]: http://duoshuo.com/
  [14]: https://portal.qiniu.com/