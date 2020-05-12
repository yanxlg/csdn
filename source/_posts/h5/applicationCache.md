---
title: application cache
tags:
  - h5
  - cache
  - optimize
  - deprecated
categories:
  - h5
  - optimize
  - cache
abbrlink: 58a73389
date: 2019-06-01 11:43:33
---
## 前言
 随着H5的发展，前端业务也来越复杂，页面中资源逐渐庞大，优化逐渐成为一个重要的关注点，而缓存在所有优化中占据重要的地位，本文主要介绍web缓存机制中的Application Cache。

## 基本介绍
 顾名思义，Application Cache就是应用缓存，首次访问时会对需要缓存的资源进行下载并且缓存到浏览器的缓存区，再次访问时会优先访问本地缓存，如果没有才会访问服务器，该方式在一定程度上较少了不必要的请求，但容易引起强缓存问题，目前该规范已被W3C废弃，纳入了不推荐使用行列，但是其[兼容性](https://caniuse.com/#search=application%20cache)目前看非常好。

## 使用方式
1. 首先对于需要缓存的html创建对应的appcache文件，etc `index.appcache`
```appcache
CACHE MANIFEST
# version xx.xx.xx
CACHE:
test.css
http://img95.699pic.com/photo/50109/8980.jpg_wh860.jpg

NETWORK:
*

FALLBACK:

```
其中具体格式如上，首先需要通过CACHE MANIFEST声明这是一个缓存配置文件，其次CACHE下定义哪些文件需要进行缓存，NETWORK中定义哪些文件不需要进行缓存，FALLBACK定义当资源无法访问时，浏览器会使用某个页面或资源
2. 配置规则：
    - 支持资源全路径，etc `http://img95.699pic.com/photo/50109/8980.jpg_wh860.jpg`
    - 支持资源相对路径，etc `test.css`
    - 支持通配符`*`
    - `不支持`局部通配符，如`*.css`
3. html文件中引入：
```html
<html manifest="index.appcache">

</html>
```

## 优点
- 配置简单
- 支持跨域缓存，可以缓存其他域名下的资源
- 支持通配符，可以对其他文件进行全部缓存或不缓存配置

## 体验
 到这里，application cache 基本使用就已经结束了，打开网站会发现第一次访问时在浏览器请求资源的同时，application cache也再次请求需要被缓存的资源，并缓存到浏览器缓存区中。再次访问时页面资源完全从本地读取，Network中会发现资源的Size列显示（disk cache）：
 {% asset_img img-left from_cache.png load from cache generator style %}
 资源无需从服务器下载，速度非常快，感觉非常美滋滋~

## There are pits everywhere
 如果到此处你以为就完了那你就真的完了，犹如标题一样，受[更新机制](#更新机制)影响，你会发现到处都是坑：
 - 配置application cache的html文件必定会被缓存，以后当html发生改变时不会立即生效，甚至永远不会生效，如果只想缓存css、js、png等静态资源而不想缓存html，目前来说是个大坑
 - 默认appcache文件也同时会被缓存，如果该文件也被缓存，那么除非手动清除浏览器中的缓存，否则不会更新
 - 每个html都需要添加manifest属性
 - 只要有文件缓存失败则整个缓存事务回滚，不会缓存任何文件
 - 根据Application Cache的加载机制，如果仅仅修改资源文件的内容（没有修改资源文件的路径或名称），浏览器将直接从本地离线缓存中获取资源文件。所以在每次修改资源文件的同时，需要修改manifest文件，以触发资源文件的重新加载和缓存。这其中，最有效的方式是修改manifest文件内部的版本注释（所以说那句注释相当重要）
 - 如果资源没有被缓存，在而没有设置NETWORK的情况下，将会无法加载（浏览器不会去网络上进行加载），所以需要使用通配符来表明除了CACHE中确定的资源以外，其他资源都需要去网络上加载
 - 当浏览器检测到资源发生更新（如manifest文件更新或者文件名发生改变）时并不会立即使用服务器中的资源，而是从缓存中读取，同时application cache从服务器获取新的资源并缓存，默认情况下下次访问时才会使用新的缓存。
 - 首次访问，两次请求资源，`加大了首次访问的负担`
 
## 更新机制
1. 在解析html DOM树时，如果检查到html节点配置了manifest属性，会先下载对应的appcache文件
2. 解析manifest文件并根据规则将页面中用到的资源和appcache中配置的需要缓存的资源重新从服务器获取，并缓存到缓存区
3. 当manifest发生更新或者资源名或路径发生改变时，进入页面后会重新从服务端获取新的资源并缓存，但当前应用的仍然是缓存中的资源
4. 缓存更新后下次进入则会从缓存区获取最新的资源并应用

## Filling pits
1. appcache文件自身必须要配置到NETWORK中，防止客户端将其缓存后服务器端更新失效
2. 有人提出通过`iframe进行缓存`，防止当前html被缓存，即manifest属性设置到iframe对应的html中，仅缓存iframe的页面而不缓存主页面，这样做你会发现缓存中真的没有该文件，但是刷新后css、js等资源并没有从缓存中读取，而是从服务器获取了。因此该方法并没有实际作用，缓存应该和主文件有关联，哪个主文件下缓存的资源仅在该主文件下有效。
3. 通过application cache提供的API进行手动刷新缓存
    ```js
    //手动更新 window.applicationCache.update();
    applicationCache.onchecking = function(){
       //检查manifest文件是否存在
    }
    
    applicationCache.ondownloading = function(){
      //检查到有manifest或者manifest文件
      //已更新就执行下载操作
      //即使需要缓存的文件在请求时服务器已经返回过了
    }
    
    applicationCache.onnoupdate = function(e){
      //返回304表示没有更新，通知浏览器直接使用本地文件	  
    }
    
    applicationCache.onprogress = function(){
      //下载的时候周期性的触发，可以通过它
      //获取已经下载的文件个数
    }
    
    applicationCache.oncached = function(){
      //下载结束后触发，表示缓存成功
    }
    
    applicationCache.onupdateready = function(){
      //第二次载入，如果manifest被更新
      //在下载结束时候触发
      //不触发onchched
        applicationCache.swapCache();// 得到最新版本缓存列表，并且成功下载资源，更新缓存到最新  
        location.reload();// 刷新页面
    }
    
    //清单 5 手动更新缓存
    
    if (window.applicationCache.status == window.applicationCache.UPDATEREADY){
        window.applicationCache.update(); 
    }
    
    applicationCache.onobsolete = function(){
      //未找到文件，返回404或者401时候触发
    }
    
    applicationCache.onerror = function(){
    
    }
    ```
    刷新后需要reload页面，用户体验极差。

## Results
 虽然该机制在浏览器中兼容性极好，包括低版本的浏览器和移动端webview都支持，但是已经被W3C废弃，后续版本肯定会移出，W3C官方也不建议使用，同时上述的pits也无法有效解决，解决不了更新问题，那带来的是更加严重的问题。
