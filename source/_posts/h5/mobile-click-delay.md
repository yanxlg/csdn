---
title: mobile click 300-350ms
date: 2019-09-29 16:54:09
tags:
    - h5
    - fastclick
    - click delay
categories:
  - h5
  - mobile
top: true
---
  做过m站开发的小伙伴都清楚，在移动端，通过webview打开的网页中，js点击事件感觉比在PC浏览器中触发慢，这是因为移动端默认情况下点击事件会存在300-350ms延迟，很多人不清楚300ms的来源，只知道有这么一回事，本文主要介绍其来源及常用解决方案。
## 产生原因
  在移动端浏览器中存在一个默认行为，那就是双击放大，因为移动端没有鼠标滚轮功能，因此设计成双击放大。浏览器底层在接收到用户点击事件时会等待一定的时间，判断用户是不是要进行双击放大操作，而这个时延===300-350ms;清楚这个原因后就可以通过其寻找解决方案。
## 解决方案:
   - touch.js：通过移动端的touch事件封装一套全新的事件类型，例如tap，longTap等。该第三方库能够有效解决300ms延迟问题，但是此库会存在touch的点透问题（点击上层元素会触发下层元素的事件），相关点透的概念此处不作具体普及
   - fastclick.js：该库全面修改了js的click事件模型，在不改变使用方式的情况下实现延迟处理，该库没有点透问题，但是其移动端手势事件未封装，仅处理了click事件，并且fastclick相对于touch.js来说体积比较大，两库选择需要注意适用环境
   - viewport width：很多人不清楚这种解决方式，如果阅读过Google开发文档的小伙伴就会发现，其在2014年Chrome 32版就针对该问题做了处理，并提供了解决方案，其就是设置viewport 的width=device-width。目前该方式在Firefox及Safari(IOS 9.3)中完全兼容
   - viewport use-scale：同样的可以通过设置viewport 的禁止缩放来直接取消系统放大判断，禁止缩放有两种形式，如下：
      ```html
        <meta name="viewport" content="user-scalable=no"/>  
        <meta name="viewport" content="initial-scale=1,maximum-scale=1"/> 
         //如果设置initial-scale=1.0，在chrome上是可以生效，但是Safari 不会
      ```
   - touch-action none：css属性touch-action设置为none，表示在该元素上的操作不会触发用户代理的任何默认行为，就无需进行300ms的延迟判断。但是其兼容性很不友好，很多浏览器不兼容。
 
  通过上面的描述，应该对该问题有了清晰的认识，目前主流应用中，仅设置meta viewport即可，不需要单独引用其他的第三方库。如果兼容版本非常低，则需要考虑其是否支持viewport设置，如果不支持则需要引入第三方库，使用touch.js还是fastclick根据项目情况而定。
