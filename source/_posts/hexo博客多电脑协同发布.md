---
title: hexo博客多电脑协同发布
date: 2017-09-13 10:33:50
tags: Hexo博客
categories: Hexo
---
公司，个人各有一台电脑，我在个人电脑搭建的hexo静态博客，有时候工作时候想临时修改点东西，或者灵感来了，这时候如果公司电脑和家里电脑能协同发布就好了，又或者电脑坏了，电脑重装系统了，我自己辛苦写的博客不能就这么没了。

前提是个人电脑的博客环境已搭好，搭建方法：[请看我这一篇博客][1]
    但是现在想要另一台电脑也可以协同发布，这就需要将此电脑博客源文件推送到github上保存，可以发现根目录有个为我们写好的.gitignore文件，里面忽略了不需要上传的文件
在博客根目录打开命令行执行:

        git init
关联远程库:

        git remote add origin 项目地址
然后要先将master分支内容提交到本地，

        git add --all
        git commit -m ""
再切出个分支，名字随意
    
        git branch hexo
然后切换到此分支：

        git checkout hexo
此时应在hexo分支上,将此分支推送到远程分支hexo上
        
        git push origin -u hexo
稍等推送完成。这时候去github会发现远程多了个hexo分支，里面是源文件，**`此时在github项目setting中把项目的默认分支改为hexo分支(即刚刚推送上来的分支)`**，因为后续所有的提交源文件，发布博客等操作都只是在这个分支上进行。
这样以后发布的时候需要先同步远程分支hexo到本地(更新最新博客内容)
然后发布完新的博客后，记得提交源文件到远程分支。
前面部署好以后，在另一台电脑就可以直接clone远程hexo分支到本地，
        
        git clone 项目地址
clone成功后，会发现本地已有博客源文件了，在根目录打开`git bash`，会发现在分支默认hexo上，如果这台电脑环境也都已配置完成，依次执行下列指令：

        npm install hexo
        npm install
        npm install hexo-deployer-git
这样就可以发布博客了，发布完成后记得提交原始文件，以便另一台电脑协同发布博客，就像项目开发多人协作一样，如果自己电脑重装系统，或者哪天换电脑了，都可以用这种方法来同步博客。


  [1]: http://simonenfp.github.io/2016/05/09/Hexo-Github%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/