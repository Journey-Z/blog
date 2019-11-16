---
title: "使用 Hugo 搭建个人博客"
date: 2019-11-15T10:55:49+08:00
categories:
- Golang
tags:
- Hugo
keywords:
- hugo
thumbnailImagePosition: left
thumbnailImage: img/gopher-hero.svg
---

  以前我没得选，现在我想做个勤奋的人，正所谓好记性不如烂笔头，因为懒，很多技术点以及解决思路都没有做笔记的习惯，虽然偶尔会存在印象笔记中，但作为一个程序员，觉得不整个博客来装下逼，就愧对了这个职业，刚好最近在学习 Golang，看了些框架，惊叹于语言的性能优越的同时，发现了 Hugo，顺便就用来搭个博客...
<!--more-->

### Hugo 安装

  Hugo 的官网提供了三种安装方式，具体请参考 **[Hugo 官网](https://gohugo.io/getting-started/installing/)** 

1. 二进制文件安装 

     在 [Hugo Releases](https://github.com/gohugoio/hugo/releases) 中找到适合自己系统的版本下载，进行安装，安装好需要配置好系统的环境变量
 
    ![avatar](http://zhongyue618.com/image/blog/hugobinary_install.jpg)
 
      因为我用的是 zsh，直接修改 .zshrc 文件
 
      ```
      sudo vim ~/.zshrc
      
      # 添加如下内容
      export HUGO=/usr/local // 此处改成你安装的 Hugo 路径
      
      # 在 PATH 后添加如下内容
      export PATH=HUGO/bin
      
      # 执行
      source ~/.zshrc
      ```
   
2. Homebrew 安装(因为我是 mac os，所以这里只说明下 Homebrew 的安装方式，linux 或者 windows 请移步官网查看) 

     如果尚未安装 homebrew，请参考**[官网](https://brew.sh/)**进行安装

     ```
     brew install hugo
     ```

3. 源代码安装，请确保系统已经搭建好 Go 环境

    ```
     go get -u github.com/gohugoio/hugo.git
    ```
     
     
### 使用 Hugo 创建网站

1. 创建网站
  ```
   hugo new site blog
  ```
  其具体结构如下：
  此时会生成一个 blog 项目，其具体结构如下：
  ```
  .
  ├── archetypes
  ├── config.toml
  ├── content
  ├── data
  ├── layouts
  ├── static
  └── themes
  ```
 - **archetypes：**
    archetypes 是存放默认的内容模版的，修改这里的内容，在使用 hugo new posts/my-first-blog.md 等命令创建出来的文件内容，和 archetypes 里面是一致的，可以根据自己的习惯对 archetypes 的内容进行修改

 - **content：**
  contents 存放的是 blog 的所有文件内容，例如 hugo new posts/my-first-blog.md 所创建的文件就在 content/posts 目录下

 - **data：**
  这个我暂时没用到，没有研究太多，这里不做赘述

 - **layouts：**
  存放以 html 形式结尾的文件，这些文件觉得网页的内容将以何种样式展示在静态网站中

 - **static：**
  存放所有的静态资源，包括 css、 js、images 等

 - **themes：**
  存放 blog 的主题，目前已经有很多开源的主题提供到 Hugo 建站中使用，我现在使用的主题是：**[hugo-tranquilpeak-theme](https://github.com/kakawait/hugo-tranquilpeak-theme)** ，更多主题请移步至 **[Hugo Themes](https://themes.gohugo.io/)**

2. 下载自己心仪的主题

 ```
  cd blog
  git init
  git submodule add git@github.com:kakawait/hugo-tranquilpeak-theme.git themes/hugo-tranquilpeak-theme
 ```
    
      将 blog/themes/hugo-tranquilpeak-theme/exampleSite 下的 content、static、config.toml 复制到 blog 根目录下，将原来的文件或目录覆盖
  
 ```
  hugo server -D
 ```

      启动本地的 hugo server，然后访问 http://localhost:1313/ ，就能预览到当前站点内容了，按照自己需要撰写的博客内容，开始发挥你的小宇宙吧！


### 发布

  这里发布主要是发布到 Github Pages 上面，至于为什么发布到 Github Pages 上，主要还是因为穷。

  关于 Github Pages 更多的详细介绍，请参考**[官网](https://pages.github.com/)**，按照 Github Pages 的教程，创建一个 repository，专门用来存放静态页面。

1. 将自己的 blog 项目，部署到 GitHub 上面去

```
git init
git add .
git commit -am "my first blog commit"
git remote add origin git@github.com:Journey-Z/blog.git
git push -u origin master
```

2. 将静态文件上传到 GitHub Pages和生成 blog 项目静态资源文件

    首先将 GitHub Pages 的项目克隆到本地

    ```
    git clone git@github.com:Journey-Z/journey.github.io.git
    ```

    到 journey.github.io 的 GitHub 页面，点击 setting，查看对应的域名地址

    ![avatar](http://zhongyue618.com/image/blog/hugogithub_page_setting.jpg)

    下拉找到 GitHub Pages 相关展示页面，如下图所示：

    ![avatar](http://zhongyue618.com/image/blog/hugogithub_pages.jpg)

    回到 blog 项目，在终端中执行以下命令：

    ```
    hugo --baseUrl=https://journey-z.github.io/
    ```

    此时会在 blog 根目录下生成一个 public 目录，里面所有的文件，都是该 blog 项目的静态文件，将 public 目录下的所有文件，复制到 journey.github.io 项目中

    ```
    git add .
    git commit -am "add static resources"
    git push
    ```

    将所有文件 push 到 GitHub 上，此时访问 https://journey-z.github.io/ ,就会出现的博客内容了。


以上是记录一下简单使用 Hugo 搭建个人博客的一些心得记录，更骚的操作的话，本人就没有研究得很深了，需要研究的话，还是得花不少时间和精力去通读下 Hugo 的官方文档的。

附上本人的 **[config.toml](https://github.com/Journey-Z/blog/blob/master/config.toml)** 文件内容