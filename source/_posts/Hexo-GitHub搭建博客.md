---
title: Hexo + GitHub 搭建我的个人博客记录
tags:
- test
- Games
categories:
- [Diary, PlayStation]
- [Diary, Games]
reward: true
---
## 💙 搭建教程
参考了网上一些教程和官方文档：
- [如何用github page搭建博客？ - 工匠羅的回答 - 知乎](https://www.zhihu.com/question/59088760/answer/265741938)
- [Hexo文档](https://hexo.io/docs/)
- [用Hexo在码云上搭建个人博客](https://blog.csdn.net/qq_40780805/article/details/99559526)

对主题进行轻微改动，最终效果如下：
![效果图](001.png)  

## 📗 搭建过程

1. 在GitHub创建仓库**dreamfields.github.io**，并安装**Hexo、Git、Node**

2. 本地拉取仓库，创建新文件夹`blogs`，运行命令`hexo init`进行初始化，并运行`npm install`安装依赖
 ![初始化过程](002.png)
 ![初始化结果](003.png)

3. 获取主题**ayer**，在`themes`文件夹下多出一个文件夹，按照其GitHub上的教程进行相关插件的安装并设置相关页面—— [Shen-Yu/hexo-theme-ayer](https://github.com/Shen-Yu/hexo-theme-ayer)
   
4. 调整相关配置，包括主题的配置文件和hexo的配置文件

5. 回到GitHub仓库，添加分支`gh-pages`，第4步中在hexo的配置文件中已经为部署到该分支上，此时运行命令`hexo deploy`进行部署即可

## 📋 常用命令

#### 创建新博客

``` bash
$ hexo new "My New Post"
```

#### 创建新页面

``` bash
$ hexo new page newPageName
```

#### 生成静态文件

``` bash
$ hexo generate (可缩写为 hexo g)
```

#### 启动本地服务

``` bash
$ hexo server (可缩写为 hexo s)
```

#### 部署到远程网站
要先在`_config.yml`进行配置

``` bash
$ hexo deploy
```

## ❌ 问题与解决

#### 图片的添加

- 首先要在配置文件`_config.yml`中进行如下更改
```bash  
post_asset_folder: true
marked:
  prependRoot: true
  postAsset: true
```
- 在引用的过程中，直接使用`![图 1](001.png)  `来引用同名资源文件夹下的001.png
- 其它引用方式可见官方文档：[Hexo文档-资源文件夹](https://hexo.io/zh-cn/docs/asset-folders)

#### Markdown的编辑

用VS Code尝试编辑了.md文件，效果一般，后来安装了几个插件，最终能基本上达到CSDN的在线编辑效果：
- Markdown All In One：是一个组合包，一股脑把最常用的Markdown优化都给你装好
- Markdown Preview Github Styling：使用这个样式，在本地就能预览Markdown文件最终在Github Pages中显示的效果
- Markdown Shortcuts：绑定了快捷键，方便编辑
    ```Bash
    ctrl+L 添加链接
    ctrl+B 加粗
    ctrl+I 斜体
    ctrl+shift+L 为图片添加说明文字
    ctrl+M+C 代码块
    ctrl+M+I 内嵌行代码
    ctrl+M+M 组合键，显示所有编辑快捷选项
    ```
- Markdown Paste：用于在.md文件中直接粘贴图片，点击右键进行添加，同时将配置设置为`${fileBasenameNoExtension}`，表示将图片默认保存到与本.md文件同名且不包含扩展名的文件夹中。前提是Hexo的_config.yml中已经开启了`post_asset_folder: true`，这样每次添加新博客的时候就会自动创建与.md文件同名的资源文件夹。

#### 自动部署travis-ci

看到官方文档上写：`Travis CI 应该会自动开始运行，并将生成的文件推送到同一 repository 下的 gh-pages 分支下`，而我的博客仅在`master`上，况且配置的时候出了问题，干脆放弃挣扎...

目前代替的方式是每次写完博客直接使用命令行即可直接部署
```bash
$ hexo g -d
```

后来的后来，还是想尝试把自动部署弄好，按照官网的步骤来的（[将 Hexo 部署到 GitHub Pages](https://hexo.io/zh-cn/docs/github-pages)）但是卡到这一步：
![](error.png)

发现是yarn和npm包的问题，查了一些博客后，修改`.travis.yml`文件如下：
```bash
language: node_js
node_js: stable

install:
  - npm install

script:
  - hexo g

after_script:
  - cd ./public
  - git init
  - git config user.name "xxx"
  - git config user.email "xxx"
  - git add .
  - git commit -m "Update docs"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master  # 自动引用之前在travis官网设置的GH_TOKEN

branches:
  only:
    - master
env:
 global:
   - GH_REF: github.com/DreamFields/dreamfields.github.io.git # 我的仓库地址
```

于是乎，每次写完博客，就可以直接push到master分支（这个过程可以用SourceTree软件实现，方便一些），接着travis便会自动部署到gh-pages分支上：
![自动部署完毕](result.png)


#### 评论功能

**显示设置：**
去官网按照文档设置即可：[valine官方文档-快速开始](https://valine.js.org/quickstart.html)，接着在主题的配置文件下修改应用的id和key。

**评论管理：**
- 由于Valine 是无后端评论系统，所以也就没有开发评论数据管理功能，需要自行登录Leancloud应用管理。

- 具体步骤：登录 > 选择你创建的应用 > 存储 > 选择Class > Comment，然后就可以对评论管理。（ [Leancloud网站](https://console.leancloud.cn/apps)）
 ![测试评论](comments.png)

 ![管理评论](comment-admin.png)


## 🎈 标签和分类
博客的标签是随缘添加，格式如下：
```bash
tags:
- 标签1
- 标签2
```
博客的分类是参考官方文档，搬运如下：
```bash
categories:
- [日记, 日记下分类1]
- [日记, 日记下分类2]
- [技术博客]
```

## 🔵 展望
- [x] 评论系统完善
- [ ] 自动部署travis-ci
- [ ] 首页图片加载过慢优化
- [ ] 在线.md文件编辑功能
- [ ] “about关于我”界面，使用模板打造个人简历
- [ ] ......