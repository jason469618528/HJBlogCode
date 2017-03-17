---
title: 使用SourceTree&Github 碰到的问题
date: 2017-03-02 14:40:15
tags:
---
### 使用github 创建一个空白仓库时，使用sourcetree克隆下来的分支里面找不到master分支，错误提示：fatal: Not a valid object name: 'master'
*解决方案 :*
	1. `git init`  初始化这个仓库
	2. `git --bare init`  清空这个仓库
这时打开SourceTree，**未暂存文件**里面会出来多个文件，提交这些文件后，master 就正常显示了。
**注：不知道是不是只有我碰到这类问题**
