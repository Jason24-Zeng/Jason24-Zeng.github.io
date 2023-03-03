---
layout: post
title: "搭建 Hexo 博客"
date: 2022-01-14 18:14:26 +0800
comments: true
tags: 
- Hexo
categories: 
- Others
- Blog
copyright_author: "Jason Zeng"
cover: /img/hexo.png
top_img: /img/hexo.png
---

#### Mac 安装 Node.js

- 最常用的方法是通过 Homebrew 进行 Node.js 和 npm 的安装。常用指令

```shell
# 更新与安装 node
brew update
brew install node

# 检查安装版本
brew list
node -v
npm -v

# 删除 node
brew uninstall node
```

#### 安装 Hexo

- 使用 npm 命令安装 Hexo

```shell
npm install -g hexo-cli 
# 或者安装 懒人包
npm install -g hexo
```

#### 初始化 Hexo

1. 初始化一个 directory: `mkdir ~/hexo/dir && cd ~/hexo/dir`

2. 初始化博客：`hexo init`

3. 静态编译博客 : `hexo g` \ `hexo generate`

4. 离线检查博客效果：`hexo s`\ `hexo server` 可以通过访问  `http://localhost:4000/` 本地检查效果

5. 修改 config 文件，路径在 `~/hexo/dir/_config.yml`，需要修改
   
   ```yml
      - deploy:
       type : git
       repository: git@github.com:username/username.github.io.git
       branch: master
   ```

6. 推到线上 `hexo d` \ `hexo deploy (-m message)`

#### 写博客并推到线上

1. 新建博客：`hexo n "your title"`\ `hexo new "your title"`

2. 进入相应路径修改 md 文件(写博客)： `cd ~/hexo/dir/source/_posts/your title.md`

3. 静态编译博客 : `hexo g` \ `hexo generate`

4. 离线检查博客效果：`hexo s`\ `hexo server` 可以通过访问 `http://localhost:4000/` 本地检查效果

5. 推到线上 `hexo d` \ `hexo deploy (-m message)`

#### 个性化 Hexo 博客

- 参考：[Butterfly 安裝文檔](https://butterfly.js.org/posts/dc584b87/)

##### 如何对每一篇 post 做相应的修改？

可参考 [# Hexo butterfly 自定义文章封面 && 主页顶部图片更改](https://blog.csdn.net/qq_43857095/article/details/108306164)

比较重要的几个参数：

```yml
title: Hexo Blog
tags: [Hexo, Nodejs, git] # 或者用 - 隔行分隔
categories: Hexo
description: 对这篇 blog 的描述
top_img: /img/avocado.jpeg # 头部图片
comments: true # 是否现实评论
cover: /img/hashtag.jpeg # 封面图片
```

如果没有 pug 以及 stylus 的渲染器，请下载安装：

```shell
npm install hexo-renderer-pug hexo-renderer-stylus --save
```

##### 如何自动生成 categories 页或者 tags 页？

1. 创建 categories page 或者 tags page，这时会在 source 文件夹下生成相应子文件夹
   
   ```shell
   hexo new page categories
   hexo new page tags
   ```

2. 在相应的子文件夹中会生成 `index.md` 文件, 修改 index.md 中的头文件配置如下
   
   ```yaml
       title: categories
   date: 2022-01-14 20:10:25
   type: "categories"
   cover: /img/category.png
   top_img: /img/category.png
   ```

3. 修改 `_config.butterfly.yml` 文件中的以下 command
   
   ```yaml
   aside:
     enable: true
     hide: false
     button: true
     mobile: true # display on mobile
     position: right # left or right
     card_author:
       enable: true
       description:
       button:
         enable: false
         icon: fab fa-github
         text: My Github
         link: https://github.com/Jason24-Zeng/Jason24-Zeng.github.io/tree/main
     card_announcement:
       enable: false
       content: This is my Blog
     card_recent_post:
       enable: true
       limit: 5 # if set 0 will show all
       sort: date # date or updated
       sort_order: # Don't modify the setting unless you know how it works
     card_categories:
       enable: true
       limit: 8 # if set 0 will show all
       expand: false # none/true/false
       sort_order: # Don't modify the setting unless you know how it works
     card_tags:
       enable: true
       limit: 40 # if set 0 will show all
       color: false
       sort_order: # Don't modify the setting unless you know how it works
     card_archives:
       enable: true
       type: monthly # yearly or monthly
       format: MMMM YYYY # eg: YYYY年MM月
       order: -1 # Sort of order. 1, asc for ascending; -1, desc for descending
       limit: 8 # if set 0 will show all
       sort_order: # Don't modify the setting unless you know how it works
     card_webinfo:
       enable: true
       post_count: true
       last_push_date: true
       sort_order: # Don't modify the setting unless you know how it works
   ```

4. 执行以下执行更新 blog
   
   ```shell
   hexo clean && hexo g && hexo d
   ```
