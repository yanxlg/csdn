---
title: git
tags:
  - tools
  - git
abbrlink: 518e617c
date: 2020-03-17 09:48:45
---
## 引导
在开发过程中，我们最常用的团队协同工具就是git，因此git常用操作必须要熟练掌握

1. 多个git仓库推送：在公司开发或者个人开发一个模块，如果需要同步到个人或者公司的git仓库，这时候就显得尤其重要
```shell
git remote set-url --add origin git@gitee.com:teamemory/myH5.git   //给origin添加一个远程push地址，这样一次push就能同时push到两个地址上面
git remote -v //查看是否多了一条push地址（这个可不执行）
git push origin master -f // 推送到仓库

git remote set-url --delete origin git@gitee.com:teamemory/myH5.git   // 删除方法
```


