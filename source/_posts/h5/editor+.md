---
title: 可视化编辑器
abbrlink: 2ed7d141
date: 2020-06-01 10:18:52
tags:
password: mikemessi
message: 本文暂不开放，需要密码才可阅读.
wrong_pass_message: 密码错误，请输入正确的密码.
---
## 前言
基于目前后台项目的逐渐增多，去coding化也逐渐变得重要，早期，活动页这种频繁变动的页面很多公司会开发一个活动编辑器，基于可视化编辑器UEditor等扩展而成，目前React和Vue更加热门，为了后台项目的快速搭建，我们准备开发一个基于React 和 antd系列的可视化编辑器，目前设计仅用于后台网站，后期丰富前台网站和m站。

## 设计
{% asset_img editor.png editor设计图 %}
一期需要支持以下内容：
- 组件列表 `待开发`
    - antd
        - 容器类布局组件
        - 非容器类
    - 自定义
        - 容器类布局组件
        - 非容器类
    - element
        - 容器类布局组件
        - 非容器类
    - 控制器列表
        - 遍历器
        - 条件器
    - 动画组件：可能会考虑放在组件层统一封装
    - html标签
- 可视化区域 `待开发`
    - PC样式
    - mobile样式
- 属性栏 `待开发`
    - 组件属性支持
    - html属性支持
- 插件层设计 `待开发`
- 预览 `待开发`

二期扩展： `待开发`
- DOM Tree：参考react-devtools，但仅显示组件相关，不想管不显示
- 插件
- 使用demo 封装webview壳，解决移动端样式兼容问题
- 模版标准开发
- redux支持
- 服务端渲染

## 参考
### 参考项目
1. https://github.com/brick-design/react-visual-editor
2. https://github.com/alitajs/yucca/tree/master/src
3. umi ui

### 参考网站
1. https://yucca.alitajs.com/5ed4786d35e8ae000a265976
2. https://brick-design.github.io/react-visual-editor/

### 参考库
1. antd 系列
2. element
3. 动画（antd 系列）


基于前端+server端开发，支持electron 和web两种方式部署


## 痛点
代码格式化太多，怎么兼容多样化写法实现源码修改