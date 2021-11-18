---
title: Git
---

# git

## git get the first commited branch
```
git name-rev commit-id
```

```
git describe --contains commit-id
git branch -a --contains commit-id
```

## git代码统计
### 查看git上的个人代码量：
```sh
git log --author="Felix Du" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -
```
结果示例：
```
added lines: 120745, removed lines: 71738, total lines: 49007
```

### 统计每个人增删行数
```sh
git log --format='%aN' | sort -u | while read name; do echo -en "$name\t"; git log --author="$name" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -; done
```
结果示例
```
Max-laptop    added lines: 1192, removed lines: 748, total lines: 444
chengshuai    added lines: 120745, removed lines: 71738, total lines: 49007
cisen    added lines: 3248, removed lines: 1719, total lines: 1529
max-h    added lines: 1002, removed lines: 473, total lines: 529
max-l    added lines: 2440, removed lines: 617, total lines: 1823
mw    added lines: 148721, removed lines: 6709, total lines: 142012
spider    added lines: 2799, removed lines: 1053, total lines: 1746
thy    added lines: 34616, removed lines: 13368, total lines: 21248
wmao    added lines: 12, removed lines: 8, total lines: 4
xrl    added lines: 10292, removed lines: 6024, total lines: 4268
yunfei.huang    added lines: 427, removed lines: 10, total lines: 417
³ö    added lines: 5, removed lines: 3, total lines: 2
```

### 查看仓库提交者排名前 5
```sh
git log --pretty='%aN' | sort | uniq -c | sort -k1 -n -r | head -n 5
```
### 贡献值统计
```sh
git log --pretty='%aN' | sort -u | wc -l
```
### 提交数统计
```sh
git log --oneline | wc -l
```
### 添加或修改的代码行数：
```sh
git log --stat|perl -ne 'END { print $c } $c += $1 if /(\d+) insertions/'
```
### 使用gitstats
[GitStats项目](https://github.com/hoxu/gitstats) 用Python开发的一个工具，通过封装Git命令来实现统计出来代码情况并且生成可浏览的网页。官方文档可以参考这里。

#### 使用方法
```sh
git clone git://github.com/hoxu/gitstats.git
cd gitstats
./gitstats 你的项目的位置 生成统计的文件夹位置
```

可能会提示没有安装gnuplot画图程序，那么需要安装再执行：
```sh
//mac osx
brew install gnuplot
//centos linux
yum install gnuplot
```

生成的统计文件为HTML：
![](https://image-static.segmentfault.com/167/520/167520797-58b7c2547b219_articlex)
### 使用cloc
```sh
npm install -g cloc
```

![](https://raw.githubusercontent.com/kentcdodds/cloc/master/other/screenshot.png)
