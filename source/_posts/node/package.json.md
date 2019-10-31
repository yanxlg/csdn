---
title: package.json
date: 2019-10-31 16:56:53
categories:
    - node
tags:
    - h5
    - node
    - package.json
---
## 概述
 本文主要介绍package.json一系列配置文件中你所需要掌握的细节。注意package.json必须是一个完全的JSON，而不是Javascript对象字面量；该文件描述受npm-config中的配置影响，下面主要介绍每个字段的含义和用法。
## 字段
1. `name`  
    name字段是`必须字段`，这个字段指定npm包的包名，如果不指定则无法被安装也无法发布，包名在一个仓库中必须唯一。  
    规则：  
      - 长度必须小于214字节
      - 不能以`.`或`_`开头
      - 不能包含大写字母和中文
      - 不能有非URL安全的字符（特殊字符）
      - 可以使用[@scope](https://docs.npmjs.com/misc/scope)来创建私有包，例如`@yanxlg/rtl-tool`
2. `version`  
    version字段是`必须字段`，这个字段指定当前包的版本，version和name组成包的唯一标识，version应与包内容是同步的。
    规则：
      - 组成：`major.minor.patch-prerelease`
      - major：主版本号，通常有重大更新，如设计变动、模块重构等，与前一大版本不兼容时升级
      - minor：次版本号，通常表示一些大的版本更改，比如API变动时升级
      - patch：修补版本号，用于bug-fix或者优化时升级
      - prerelease：预发布版本号，可能存在一些问题，但是其他环境需要使用到该包的时候指定，表明非正式发布
3. `description`
    包的描述，是一个字符串，对于npm仓库搜索引擎有效，可以方便开发者找到这个包
4. `keywords`
    keywords表明包的关键字，是一个字符串数组，对于npm仓库搜索引擎有效，可以方便开发者找到这个包
5. `homepage`
    包的官网url，是一个合法uri地址
6. `bugs`
    当遇到bugs反馈地址及邮箱，通常指定github的issues地址和开发者邮箱，例如：
    ```json
      {
         "url": "https://github.com/yanxlg/csdn/issues",
         "email": "project@hostname.com"
      }
    ```
