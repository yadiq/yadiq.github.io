---
title: GitHub Pages + Hexo搭建个人博客网站
date: 2024-11-29 20:26:36
tags:
---

 GitHub Pages + Hexo搭建个人博客网站

## 概述
本文基于 github 项目 [yadiq.github.io](https://github.com/yadiq/yadiq.github.io)。
通过GitHub Pages + Hexo实现了个人博客网站的搭建。


## 1.GitHub Pages

### 1.1 概述
GitHub Pages是GitHub提供的一个网页托管服务，可以用于存放静态网页，包括博客、项目文档。
一般GitHub Pages的网站使用github.io的子域名，但是用户也可以使用第三方域名。

### 1.2 配置
在GitHub上创建一个新的代码仓库用于保存我们的网页。
注意仓库名，格式必须为<username>.github.io。
在GitHub settings/pages目录中可以看到我们主页的地址为 https://yadiq.github.io/

## 2.Hexo博客框架

### 2.1 概述 doing
[Hexo](https://hexo.io/)是一个快速、简洁且高效的博客框架，支持Markdown。
基于Node.js编写的，所以需要安装NodeJS和npm工具。
创建者是台湾人，文档友好，提供丰富[主题](https://hexo.io/themes/)。

### 2.2 配置
1. 安装Hexo
npm install -g hexo-cli  //全局安装
hexo v //查看版本
2. 建站
cd yadiq.github.io //切换至博客文件夹
hexo init //在当前文件夹建立网站
npm install //安装模块
hexo server //本地启动，网址 http://localhost:4000
3. 更换主题
默认的主题不太好看，官方提供了[数百种主题](https://hexo.io/themes/)供用户选择。
博主使用的是[yuzu](https://github.com/Cerallin/hexo-theme-yuzu)
+ 使用pug代替ejs作为模板引擎
npm i hexo-renderer-pug
+ 安装主题
cd yadiq.github.io/themes
git clone https://github.com/Cerallin/hexo-theme-yuzu
+ 修改站点配置文件，根目录下的 config.yml
将theme 字段改为 hexo-theme-yuzu
将主题文件夹下 _root_config_example.yml 的内容添加到 config.yml
4. 创建文章
hexo new FirstPost //新建文章，默认post布局，路径source/_posts
hexo new page about //新建文章，page布局，路径source

5. 在 GitHub Pages 上部署
+ 将博客文件夹中的文件 push 到储存库的默认分支
+ 修改储存库配置
前往 Settings > Pages > Source ， 将 source 更改为 GitHub Actions
+ 在储存库中建立 .github/workflows/pages.yml
详见 https://hexo.io/zh-cn/docs/github-pages
+ 部署完成后，前往 yadiq.github.io 查看网页
