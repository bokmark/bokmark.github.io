---
title: "Gitee Pages服务"
date: 2020-11-14T12:20:31+08:00
draft: false
summary: "如何创建 github pages"
---

# GiteePages

---

> 作为一个程序员需要有一个记录知识点的地方，也需要一个做记录整理自己的知识体系的地方，本来想要用github的pages，但是由于众所周知的原因，所以选择Gitee 的pages 功能。

### 准备

1. [Pages](https://gitee.com/help/articles/4136) 这是gitee 提供的帮助文档
2. [主题](https://themes.gohugo.io/) 这是主题列表
3. 一个gitee账号
4. 一个同名的仓库 比如我的名字是bokmark 我申请的库名就是 bokmark 那么我的pages的地址为 bokmark.gitee.io
5. 一个用来保存md 文件以及 hugo的库

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
   theme = "hugo-whisper-theme"
   ```

4. 创建你要的页面

   ```
   hugo new about.md
   // 会在content页面 创建about.md 文件
   hugo new Notes/Other/GiteePages.md
   // 会在 content/Notes/Other 目录下创建 GiteePages.md
   ```

5. 运行

   ```
   hugo
   // 如果想要在本地看一下效果可以这样
   hugo server
   // localhost:1313
   ```

6. 部署

   ```
   hugo --baseUrl="http://bokmark.gitee.io/"
   ```

7. rm public/index.xml 文件

8. 将public的内容全部push到pages的库

9. 在gitee中的pages 服务启动
