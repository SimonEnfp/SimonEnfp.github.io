---
title: hexo博客多电脑协同发布
date: 2016-09-13 10:33:50
tags:
categories: 技术分享
---
公司，个人各有一台电脑，一般是下班回家发布博客，可有时候工作时候想临时修改点东西，或者灵感来了，这时候如果公司电脑和家里电脑能协同发布就好了。

这里前提是个人电脑的博客环境已搭好，搭建方法：[请看我这一篇博客][1]
    但是现在想要另一台电脑也可以协同发布，这就需要将此电脑博客源文件推送到github上保存，可以发现根目录有个为我们写好的.gitignore文件，里面忽略了不需要上传的文件
    在博客根目录打开命令行，会发现现在的分支是在master上，先切出个分支，名字随意
    
        git branch hexo
然后切换到此分支：

        git checkout hexo
此时应在hexo分支上,将次分支推送到远程分支hexo上
        
        git push origin -u hexo
稍等推送完成。这时候去github会发现远程多了个hexo分支，里面是源文件，此时在github项目setting中把github默认分支改为hexo分支，因为后续所有的提交源码，发布博客等操作都只是在这个分支上进行。
这样以后发布的时候需要先同步远程分支hexo到本地(更新最新博客内容)
然后发布完新的博客后，记得提交源文件到远程分支。
前面部署好以后，在另一台电脑就可以直接clone远程hexo分支到本地，
        
        git clone 项目地址
clone成功后，会发现本地已有博客源文件了，在根目录打开命令行，会发现在分支默认hexo上，如果这台电脑环境也都已配置完成，依次执行下列指令：

        npm install hexo
        npm install
        npm install hexo-deployer-git
这样就可以发布博客了，发布完成后记得提交原始文件，以便另一台电脑协同发布博客，就像项目开发多人协作一样。


  [1]: http://simonenfp.github.io/2016/05/09/Hexo-Github%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/