---
title: adjust
date: 2019-09-30 08:56:04
tags:
  - RN
  - adjust
top: true
---
## 前言
移动端开发中，设计到不同设备分辨率的UI大小适配问题，该问题不仅存在于H5中，RN中同样会出现该问题

## 解决方案
- UI设计按照750标准进行效果图设计
- RN中处理方法：
    1. 全局缩放：
       在RN的根节点上通过样式设置当前视图的缩放比例，对视图区域进行整体缩放，使得自适应变得更加简单
       ```typescript
           import {PixelRatio, Dimensions, StyleSheet} from 'react-native';
           const dp2px = dp=>PixelRatio.getPixelSizeForLayoutSize(dp);
           let designSize = {width:750,height:1280}; //假设设计尺寸为：750*1280
           let pxRatio = PixelRatio.get();
           let win_width = Dimensions.get("window").width;
           let win_height = Dimensions.get("window").height;
           let width = dp2px(win_width);
           let height = dp2px(win_height);
           let design_scale = designSize.width/width;
           height = height*design_scale;
           let scale = 1/pxRatio/design_scale;
           
           const styles = StyleSheet.create({
               page: {
                   width: width,
                   height: height,
                   transform: [
                       {translateX: -width * .5},
                       {translateY: -height * .5},
                       {scale: scale}, 
                       {translateX: width * .5}, 
                       {translateY: height * .5}
                   ]
               }
           });
       ```
       将styles.page添加到RN页面根节点上，页面中完全按照设计稿给定的大小进行布局即可
       > 存在问题：
          > 1.RN官方自定义组件或第三方原生组件的大小受到缩放影响，例如下拉刷新，此时大小会变得不正常
       <hr>
    2. 通过计算转换比例去实时计算：
       在样式文件中通过一个转换函数去将UI设计大小转换成实际大小
       ```typescript
           import {StyleSheet, Dimensions} from 'react-native';
           let width = Dimensions.get("window").width;// dp
           // UI 默认给图是 750
           const uiWidthPx = 750;
           const uiRatio = width / uiWidthPx;
           const styles = StyleSheet.create({
               infoHeader:{
                   position:"absolute",
                   width:"100%",
                   height:84 * uiRatio,
                   flexDirection:"row"
               },
           });
       ```
       在指定大小的属性后面通过"* uiRatio"的方式转换成实际大小
       > 存在问题：
          > 1.所有样式需要手动使用表达式或者函数去进行转换，工作量增加， 比较繁琐
          > 2.动态获取View视图大小并进行处理的时候不需要进行转换，例如onLayout或者measure方法中获取的大小，这两者都是实际大小，直接进行计算即可
       <hr>
    3. 通过StyleSheet提供的全局代理转换拦截器进行处理
       StyleSheet类中提供了一个设置属性`getter`拦截器的方法`setStyleAttributePreprocessor`，可以通过这个方法设置全局属性拦截器进行自动转换
       ```typescript
           import {StyleSheet, Dimensions} from 'react-native';
           let width = Dimensions.get("window").width;// dp
           let height = Dimensions.get("window").height;// dp
           // UI 默认给图是 750
           const uiWidthPx = 750;
           const ratio = width / uiWidthPx;
           const ratioKeys = {
               fontSize: true,
               paddingHorizontal: true,
               paddingLeft: true,
               paddingRight: true,
               padding: true,
               marginHorizontal: true,
               marginRight: true,
               marginLeft: true,
               margin: true,
               borderRadius: true,
               borderWidth: true,
               right: true,
               left: true,
               width:true,
               paddingVertical: true,
               paddingTop: true,
               paddingBottom: true,
               marginVertical: true,
               marginTop: true,
               marginBottom: true,
               height: true,
               minHeight: true,
               lineHeight: true,
               top: true,
               bottom: true,
           };
           Object.keys(ratioKeys).forEach(key => {
               StyleSheet.setStyleAttributePreprocessor(key, (value) => {
                   return ratio * value;
               });
           });
       ```
       > 存在问题：
          > 1.拦截器api处于尝试截断，后面如果修改，是否会废弃无法确定
          > 2.所有值都会进行比例缩放，包括现有组件中指定的样式大小，影响极大，出现很多超出预期的结果
          > 3.动态获取视图大小并计算后赋值给组件新的属性时会再次被拦截器转换，此时会出现不可控问题
          > 4.大部分属性支持字符串设置大小，这些属性可以使用自定义格式，例如px格式去进行拦截器处理，该方法可以规避上述2，3问题，但无法完美解决font-size、line-height等不支持字符串类型的属性，当然这两个属性也可以强制使用字符串，typescript下需要扩展类型

## 总结
 >上述三个方案中
    >- 方案一最简单，不需要考虑太多，并且复杂度最低，但是无法解决原生模块的大小问题；
    >- 方案二最是复杂，所有代码都需要进行手动调用转换，但是可配型强，哪些需要转换，哪些不需要完全可以定制，同时不影响原有组件；
    >- 方案三不太稳定，官方API暂时处于试用阶段，可能会被废弃，并且影响比较大，容易造成不良影响
    >- 建议使用方案二，开发中简单化往往带来更复杂的问题，复杂化有时候更能满足需求
