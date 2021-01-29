+++
title = "我的git配置"
description = ""
tags = [
    "review",
    "git",
]
date = "2020-04-02"
categories = [
    "others",
    "tools"
]
menu = "main"
+++

```
# This is Git's per-user configuration file.
[user]
# Please adapt and uncomment the following lines:
	name = xxx
	email = xxx
[filter "lfs"]
	clean = git-lfs clean -- %f
	smudge = git-lfs smudge -- %f
	process = git-lfs filter-process
	required = true
[alias]
	co = checkout
	cp = cherry-pick
	cm = commit
	ba = branch -a
	bh = branch
	amend = commit -a --amend
	lg = log
  lo = log --oneline
	st = status
	df = diff
	review = !sh -c 'git push origin HEAD:refs/for/$1' -
	up = pull --rebase

```
