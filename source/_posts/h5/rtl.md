---
title: h5 RTL
tags:
  - h5
  - rtl
categories:
  - h5
  - ill8
top: true
abbrlink: 5213db69
date: 2019-02-25 16:25:15
---
本文分享关于前端中实现正反排排版的技术方案，介绍相关应用场景及成熟方案
## 前言
   前端国际化产品中不仅涉及到多语言的解决方案，还会涉及到正反排版的问题，多语言通常对应的框架都有i18n的技术方案，而正反布局很少有比较好的解决方案，因此正反排版问题是前端工程师一直以来比较头疼的问题。
## 前期调研
   web 开发中rtl涉及到文字及布局，而在html中有html元素的dir属性及css中direction属性去控制文本的左右排版问题，direction属性只隐性存在css中，无法控制布局显示，因此使用html标签元素的dir属性来控制左排还是右排。同时通过该属性控制不同的样式显示
## 一般方案
1. css
    - css 中直接编写通用样式类及对应的ltr和rtl类，例如
        ```css
       .test{
          background-color:red;
          color: blue;
       }
       [dir=ltr] .test{
           padding-left: 10px;
      }
      [dir=rtl] .test{
          padding-right: 10px;
      }
        ```
     - 功能可以正常实现，但是增加了大量的开发难度，源码比较复杂，项目变得臃肿
     - 不可通用，引用的第三方库需要重新定义一份ltr和rtl样式
2. less
    - 通过less中的mixin + when + when not 通过函数的方式去判断属性是否需要进行ltr和rtl区分，如果需要则针对该属性生成单独的ltr和rtl样式类
    - 该方法主要复杂度在mixin的实现上，mixin中需要判断所有位置相关的属性，并且需要支持css复合属性，根据不同规则去拆解计算，实现复杂度较大
    - 该方法最终生成的css类过多，一个初始样式类中所有匹配的属性各生成两个样式类，如果不适用clean插件最终生成的样式文件臃肿，降低cssom渲染效率
    - 不可通用，外部引入的三方库中样式基本是css,无法支持
3. sass
    - 通过sass中的@mixin + @include + @if +@else去实现类似less中的判断。
    - 负责度也等同于less方案
    - 最终生成的样式也和less一样臃肿
    - 不可通用，外部引入的三方库中样式基本是css,无法支持
    
## 优化解决方案
* 以上方案都是建立在原生css及预编译的基础上的，存在很多局限性，复杂度也比较高，通用性低
* 前端开发规范性要求source code保持简洁性，这种转换和目前大量使用的px2rem一样，既然px2rem存在大量的工程化插件支持自动转换，那么rtl也能同样实现
* 样式文件使用的工程化插件一般分为less-loader插件、sass-loader插件和postcss插件，为了通用性，我们开发postcss插件，支持所有预编译方案
* 模块依赖：rtlcss(目前非常成熟的主流rtl转换模块，支持所有css复合属性及background-position等，支持ltr、rtl相同属性不同值设置，支持属性忽略设置，支持样式忽略设置，支持rtl动态新增属性设置等），具体相关语法可参考[官网](https://rtlcss.com/)
* postcss 插件开发
    - 熟悉postcss原理及AST语法树
    - 插件代码如下：
        ```javascript
         const postcss = require('postcss');
         const postcssPluginRtl = require('rtlcss');
      
         module.exports = postcss.plugin('postcss-plugin-rtl', opts => {
             const {exclude, include} = opts || {};
             return root => {
                 const file = root.source.input.file;
                 if (exclude && exclude.test(file) || include && !include.test(file)) {
                     return ;
                 }
                 let extraRulesList = [];
                 root.walkRules(rule => {
                     if(rule.parent.type==="atrule"){
                         return;
                     }
                     const css = rule.toString().replace(/;?}$/,";}");
                     const parseCss = postcssPluginRtl.process(css, {
                         useCalc: true,
                     });
                     const prevNode = postcss.parse(css);
         
                     const nextNode = postcss.parse(parseCss);
                     const selector = rule.selector;
         
                     const rtlRule = postcss.rule({ selector: "[dir='rtl'] " + selector});
                     const ltrRule = postcss.rule({ selector: "[dir='ltr'] " + selector});
                     let rtlRuleHasChildren = false;
                     let removeList = [];
                     // 可能不止一个nodes
                     for (let i = 0 ; i < prevNode.nodes.length; i++) {
                         let cur = prevNode.nodes[i].nodes[0]; // 初始值
                         let _cur = nextNode.nodes[i].nodes[0];
                         // 游标遍历
                         while (cur) {
                             if (cur.type === "decl") {
                                 while ( _cur.type !== "decl") {
                                     _cur = _cur.next();
                                 }
                                 // compare
                                 // fix trim
                                 if (cur.prop !== _cur.prop || cur.value.trim() !== _cur.value.trim()) {
                                     ltrRule.append({
                                         prop: cur.prop,
                                         value: cur.value,
                                     });
                                     rtlRule.append({
                                         prop: _cur.prop,
                                         value: _cur.value,
                                     });
                                     removeList.push(cur);
                                     rtlRuleHasChildren=true;
                                 }
                                 _cur = _cur.next();
                             }
                             cur = cur.next();
                         }
                     }
                     removeList.map((decl) => decl.remove());
                     if (rtlRuleHasChildren) {
                         extraRulesList.push(rtlRule);
                         extraRulesList.push(ltrRule);
                     }
                     // 转成Node后进行比较
                     rule.replaceWith(prevNode);
                 });
                 root.append(extraRulesList);
             };
         });
        ```
    - 最终生成的样式做了优化，针对每一个父样式类，仅会生成一个对应的ltr和rtl样式，其内所有属性均生成在一个类中,如下所示
       {% asset_img img-left image2019-9-25_11-21-27.png rtl generator style %}
    - 使用方式，在webpack的postcss plugin中添加
    - 最终业务样式完全按照原有设计图开发即可，如有定制按照rtlcss提供的directives去配置
    - hacks:
        * AtRule：如@keyframes，@keyframes动画不支持父子选择器模式，该插件对于AtRule不作处理，因此@keyframes如果想要实现ltr与rtl不同则需要定义两个不同的动画，然后通过rtlcss directives语法指定rtl专有属性值
        * sass：压缩模式下行`注释`必须使用`/*!xxx*/`格式，非压缩模式下可以使用/**/。压缩模式会将最后一个属性的结束符“;”自动删除，会影响directives 解析，因此插件内部做了自动补全
        {% asset_img ex_1.png sass注释用法 %}
        * less：directives完全按照官网方式配置
    - next：
        * 可能存在部分未适配的case，后续遇到后做相应处理
## 发布 & 版本
<span class="inline-image hide-alt">[![npm](https://img.shields.io/npm/v/@yanxlg/postcss-rtl.svg?style=flat-square)](https://www.npmjs.com/package/@yanxlg/postcss-rtl)[![npm](https://img.shields.io/npm/l/@yanxlg/postcss-rtl.svg?style=flat-square)](https://www.npmjs.com/package/@yanxlg/postcss-rtl)[![npm](https://img.shields.io/npm/dt/@yanxlg/postcss-rtl.svg?style=flat-square)](https://www.npmjs.com/package/@yanxlg/postcss-rtl)[![npm](https://img.shields.io/npm/dm/@yanxlg/postcss-rtl.svg?style=flat-square)](https://www.npmjs.com/package/@yanxlg/postcss-rtl)[![Known Vulnerabilities](https://snyk.io/test/github/mbrevda/@yanxlg/postcss-rtl/badge.svg)](https://snyk.io/test/github/mbrevda/@yanxlg/postcss-rtl)[![wercker status](https://app.wercker.com/status/51bfd9b8aa6e52acf77310e17f00aff4/s/master "wercker status")](https://app.wercker.com/project/byKey/51bfd9b8aa6e52acf77310e17f00aff4)</span>
