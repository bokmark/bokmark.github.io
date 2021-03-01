---
title: "sed"
date: 2020-11-14T12:20:31+08:00
draft: false
summary: "Sed 使用方法"
---

> 最近有个问题就是 测试发过来的 log 一大堆里面很多不是我们自己应用的log。我们就从去除无用log的角度来分析sed怎么用

sed全名叫stream editor，流编辑器，用程序的方式来编辑文本[官方文档](http://www.gnu.org/software/sed/manual/sed.html)

> 如果你和作者一样是mac用户那么恭喜你了。mac上的sed 与linux的不同。你需要安装gnu的sed套件。
> ```
> brew install coreutils
> brew install gnu-sed
> ```
> 这时候你可以使用gsed来代替sed命令
> 当然你可以通过添加这个
> ```
> export PATH="/usr/local/opt/gnu-sed/libexec/gnubin:$PATH"
> ```
> 到zsh或者bash的配置文件中来，来使用sed 代替gsed



我们主要是通过官方文档来进行sed的学习

## sed hello world
  大致的格式类似于`sed SCRIPT INPUTFILE ...`
  ```
  sed 's/hello/world/' input.txt > output.txt
  ```
  这一句话的作用就是将input.txt 中的每一行中的第一个hello替换成world 然后输出到output.txt。

### sed的大致格式
  `sed OPTIONS... [SCRIPT] [INPUTFILE...]`
  作者将分别简单介绍 OPTIONS SCRIPT INPUTFILE

#### SCRIPT
  首先一个SCRIPT的大致格式为 `[addr]X[options]`


##### addr
  addr 表明了file中的哪些行会被执行command。先看一些例子，我们再简单的来归纳一些规则
  `'144s/hello/world/'` ： 只有144行执行 替换hello到world的命令
  `'s/hello/world/'` : 每一行都执行 替换hello到world的命令
  `'/apple/s/hello/world/'` ：只有包含apple的行才执行 替换hello到world的命令
  `'4,17s/hello/world/'` : 4到17行 执行 替换hello到world的命令
  `'/apple/!s/hello/world/'`:不包含apple的行才执行 替换hello到world的命令
  `'4,17!s/hello/world/'`:不是4到17行 执行 替换hello到world的命令

  规则如下：
  - 如果是数字选行不需要用`/`
  - `4,17`选4到17的range
  - - *`$`代表最后一行*
  - - `1~3` 从1开始每隔3行
  - addr之后添加`!` 进行反选
    `/apple/!` 不包含apple的行
    `4,17!`不包含4到17的行
  - 如果是非数字选行，那么需要用`//`包起来。包起来的内容是一个正则表达式
  - `/bash$/` 匹配以bash结尾的行
  - and so on 更多的请直接到开篇提到的官网进行查询


##### X
  - s replace
  - p print
  - d delete

##### options
  根据各个关键词进行添加


#### INPUTFILE

  如果你没有输入inputfile 或者inputfile 为 `-`。sed将会把当前的输入视为inputfile。所以以下三种方式是等价的
  ```
  sed 's/hello/world/' input.txt > output.txt
  sed 's/hello/world/' < input.txt > output.txt
  cat input.txt | sed 's/hello/world/' - > output.txt
  ```
  - 多文件
    sed 将会把多文件 视为一整个长流
    ```
    sed -n  '1p ; $p' one.txt two.txt three.txt
    ```
    这句话将会输出第一个文件的第一句话和最后一个文件的最后一句话。使用-s 反转这个列表

#### OPTIONS
- `-i`

  sed 使用 `-i` 来代替输出。
  ```
  sed -i 's/hello/world/' file.txt
  ```
  这句话将会把file.txt 文件中的第一个 hello 替换成world。

- `-n`
  sed 默认会输出所有的输入。我们可以使用`-n`来压制这些默认的输出
  ```
  sed -n '45p' file.txt
  ```
  这里将会print出file.txt 文件的第45行。（p就是print的意思）


- `-e` `-f`
  在没有-e 或者-f的情况下，sed将会把把第一个非option参数作为script 下一个非option的参数作为inputfile。
  如果有-e或者-f 那么剩下的非option参数作为inputfile。
  ```
  sed 's/hello/world/' input.txt > output.txt

  sed -e 's/hello/world/' input.txt > output.txt
  sed --expression='s/hello/world/' input.txt > output.txt

  echo 's/hello/world/' > myscript.sed
  sed -f myscript.sed input.txt > output.txt
  sed --file=myscript.sed input.txt > output.txt
  ```
  上面的这一些语句都是等价的。（当然，myscript.sed 里面写着 s/hello/world）

## 开始我们的探索
  别忘了我们的目的，是为了在整体的log文件中，找出我们自己的应用的log
  - 找出我们的应用的进程号 比如我这里 32085
  ```
  sed '/32085/!d' main.log > main.log.1
  ```

### 其他用法等下次用到 我们再加
