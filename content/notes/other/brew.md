---
title: "brew"
date: 2020-11-14T12:20:31+08:00
draft: false
summary: "brew 更换阿里的镜像"
---
> [转载](https://blog.csdn.net/weixin_36139431/article/details/103361751)

你可能有过这样糟糕的经历，当你满心欢喜的敲下 “brew install 应用名称”，静静的等待安装结果的时候，Homebrew在 Updating Homebrew卡死了。


[Homebrew](https://brew.sh/)是一款Mac OS平台下的软件包管理工具，拥有安装、卸载、更新、查看、搜索等很多实用的功能。简单的一条指令，就可以实现包管理，而不用你关心各种依赖和文件路径的情况，十分方便快捷。

### 使用 阿里云 的 Homebrew 镜像源进行加速

如果你没有更换过镜像源，执行 brew 命令安装应用的时候，跟以下 3 个仓库地址有关：
- brew.git
- homebrew-core.git
- homebrew-bottles

通过以下操作将这 3 个仓库地址全部替换为 阿里云 提供的地址

#### 更换 brew.git
```
cd "$(brew --repo)"

git remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git
```
#### 更换 homebrew-core.git

```
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"

git remote set-url origin https://mirrors.aliyun.com/homebrew/homebrew-core.git
```

> 执行上面两个命令之后请执行  `brew update`

#### 更换 homebrew-bottles

如果你使用zsh那么
```
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles' >> ~/.zshrc

source ~/.zshrc
```

如果你使用 bash。 那么
```
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles' >> ~/.bash_profile

source ~/.bash_profile
```
