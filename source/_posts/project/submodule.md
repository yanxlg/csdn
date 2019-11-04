---
title: submodule
categories:
  - project
tags:
  - node
  - npm
  - h5
  - git
  - submodule
  - project framework
top: true
abbrlink: 6a58ae0d
date: 2019-10-29 15:11:11
---
## 前言
 随着前端的发展，前端项目越来越庞大、臃肿，模块化的思想越来越普及，不同的项目公用的模块需要进行统一管理，由此提出来组件库的思想。
## 组件库构建方式
 - npm 仓库：
    直接使用npm官方仓库，通过注册npm官方企业账号或个人账号，开发npm模块代码，并使用`npm publish`或者`yarn publish` 将模块发布到npm仓库，使用时直接在package.json的`dependencies`或者`devDependencies`中配置模块及版本，并且使用`npm install`、`yarn add`或者`yarn isntall`安装指定模块或所有模块
 - 私有仓库：
    * cnpm: 通过阿里开源的cnpm进行部署企业内网npm仓库，[配置教程](https://www.febugs.com/npm-private-library-build-and-config/)
    * Nexus：[配置教程](https://www.cnblogs.com/cheyunhua/p/10763370.html)
    * npm + git：npm install 可以和git进行结合，支持从git仓库下载并安装模块，`package.json`中配置如下
      ```json
          "dependencies":{
            "moduleName": "git+ssh://git@ip/project.git#version",
          }
      ```
    * etc ...
 - git submodule
    git 支持子模块配置功能，可以将git上其他项目添加到当前项目中做为子模块管理，常用命令为：
    ```shell script
       git submodule init  # 初始化当前项目中的.gitmodules文件
       git submodule update # 更新当前项目.gitmodules配置文件中配置的子项目
       git submodule add <submodule_url> <submodule_path> # 向当前项目添加子项目，同时会更新或创建.gitmodules文件
       git clone --recurse-submodules <main_project_url>  # clone主项目的同时下载所有子项目
    ```
   通过git的submodule功能同样能够管理项目依赖，实现项目关系架构设计。
## 总结
   以上方案没有明显的优缺点，根据需要进行取舍，常规企业会拥有自己的私有npm服务器，同时在项目依赖时也会用到git submodule
