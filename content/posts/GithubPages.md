---
title: "Github Pages服务"
description: ""
tags: [
    "go",
    "golang",
    "templates",
    "themes",
    "development",
]
dat: "2014-04-02"
categories: [
    "Development",
    "golang",
]
menu: "main"
---

# GithubPages

---

> 作为一个程序员需要有一个记录知识点的地方，也需要一个做记录整理自己的知识体系的地方。GithubPages 作为影响力最大的博客网站，当我们需要进行博客搭建的时候，当然是选择他！




### 准备

1. [Pages](https://pages.github.com/) 这是github pages 提供的帮助文档。
我们按照他提供的步骤一步步的搭建好仓库。比如我取名就是bokmark.github.io
2. [主题](https://themes.gohugo.io/) 这是主题列表。我们使用hugo 来将markdown文件转化为html文件


### 开始

1. 第一步创建hugo的站

   ```
   hugo new site mysite
   ```

2. 然后再选一个好看的theme，我这里推荐我使用的这个主题

   ```
   cd mysite
   git clone https://github.com/jugglerx/hugo-whisper-theme.git themes/hugo-whisper-theme
   ```

3. 修改config.toml

   ```
   baseURL = "/"
   themesDir = "themes"
   publishdir = "docs"
   theme = "hugo-whisper-theme"
   ```
   这里面最重要的是 `publishdir = "docs"`

4. 创建你要的页面

   ```
   hugo new about.md
   // 会在content页面 创建about.md 文件
   hugo new notes/other/GithubPages.md
   // 会在 content/notes/other 目录下创建 GithubPages.md
   ```

5. 运行

   ```
   hugo
   // 如果想要在本地看一下效果可以这样
   hugo server
   // localhost:1313
   ```
   这时候在docs文档下面将会生成我们需要的html文件

6. 上传
  将整个库上传到github，包括`docs`文件夹。你可以使用git命令 也可以用github desktop 都可以。

7. 修改pages settings
  - 来到你的仓库( https://github.com/{username}/{username}.github.io )的settings页面。
  - 找到GitHub Pages栏
  - 找到 Source 栏
  - select folder 选择docs
  - 点击save

8. 查看效果
  https://{username}.github.io

9. 亲测有效哦，毕竟你都看到这个页面了。
