---
author: Bokmark Ma
title: "Android 下载源码"
date: 2021-10-26T00:16:18+08:00 
slug: fwk/source
description: TODO
categories:
    - Android
tags:
    - Framework
    - Android
---

# Android Framework 下载源码

> [参考资料](https://lug.ustc.edu.cn/wiki/mirrors/help/aosp/)

## 准备repo
```bash
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
## 如果上述 URL 不可访问，可以用下面的：
## curl -sSL  'https://gerrit-googlesource.proxy.ustclug.org/git-repo/+/master/repo?format=TEXT' |base64 -d > ~/bin/repo
chmod a+x ~/bin/repo
```


## 修改源地址

如果提示无法连接到 gerrit.googlesource.com，可以编辑 ~/bin/repo，把 REPO_URL 一行替换成下面的：  

` REPO_URL = 'https://gerrit-googlesource.proxy.ustclug.org/git-repo'`

## 初始化仓库

```
python3 ~/bin/repo init --depth=1 -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-9.0.0_r3
```
相关解释：
- `python3 ~/bin/repo` 在mac系统下，python命令无法运行，可以直接用 python3 来跑repo 脚本。  
- `--depth=1` 只下载一层代码  
- `-b android-9.0.0_r3` 指定下载的版本，配合  `--depth=1` 使用。

## 同步源码树

```
python3 ~/bin/repo sync
```






 