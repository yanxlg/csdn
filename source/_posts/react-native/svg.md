---
title: svg
date: 2019-11-26 11:30:31
tags:
    - react-native
---
## 前言
`react-native`中图片适配一直来说是个比较麻烦的问题，需要使用不同分辨率的图片，相对来说比较麻烦，对比与web端矢量图的应用，react-native中对于适量图svg的需求也越来越广泛。

## 前提
`react-native`有一个svg的插件`react-native-svg`，该插件可以使用本地svg，但是旧版本在android生成release包时图片会不显示，因此svg的使用不尽人意，网上提供的方式是将svg通过脚本生成一个js文件存放svg中的xml，在配合`SvgXml`组件使用，该方法可行，但是每次添加图片时都需要执行脚本。

## 方案：
`react-native-svg` 新版本发布后，支持配合`metro`或者`babel`直接使用svg图片的方案，其最终编译时也是将svg图片转换成xml字符串。[具体参考插件](https://www.npmjs.com/package/react-native-svg)
