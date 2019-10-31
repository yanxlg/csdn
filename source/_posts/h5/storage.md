---
title: storage
date: 2019-10-29 14:59:56
tags:
    - h5
    - storage
    - 跨窗口通信
categories:
  - h5
  - storage
top: true
---
## 前言
  随着前端业务的复杂化，前端对于数据敏感度越来越高，对于本地化存储数据的需求越来越多，本文主要介绍目前前端中可用于数据存储的方式

## 本地化方案
1. cookie
    + cookie 作用
        服务端通常使用cookie+session来进行验证、授权等权限控制操作。
        cookie可以临时存储小量数据供客户端使用，例如加密秘钥等
        客户端可以将部分数据同时cookie来实现长期存储，或者传递给服务端
    + cookie 大小：不超过4KB
    + cookie 限制
    cookie 在传统web中起到非常重要的作用，因为传统web仅能通过cookie在本地存储一些数据，常用与进行账号验证，token下发，授权等。cookie存在很多限制问题，具体如下：
    > 一、浏览器允许每个域名所包含的cookie数：
  　&emsp;Microsoft指出InternetExplorer8增加cookie限制为每个域名50个，但IE7似乎也允许每个域名50个cookie。
    &emsp;&emsp;Firefox每个域名cookie限制为50个。
    &emsp;&emsp;Opera每个域名cookie限制为30个。
    &emsp;&emsp;Safari/WebKit貌似没有cookie限制。但是如果cookie很多，则会使header大小超过服务器的处理的限制，会导致错误发生。
    &emsp;&emsp;注：`“每个域名cookie限制为20个”将不再正确`！
    二、当很多的cookie被设置，浏览器如何去响应。
    &emsp;&emsp;除Safari（可以设置全部cookie，不管数量多少），有两个方法：
    &emsp;&emsp;最少最近使用（leastrecentlyused(LRU)）的方法：当Cookie已达到限额，自动踢除最老的Cookie，以使给最新的Cookie一些空间。Internet Explorer和Opera使用此方法。
    &emsp;&emsp;Firefox很独特：虽然最后的设置的Cookie始终保留，但似乎随机决定哪些cookie被保留。似乎没有任何计划（建议：在Firefox中不要超过Cookie限制）。
    三、不同浏览器间cookie总大小也不同：
    &emsp;&emsp;Firefox和Safari允许cookie多达4097个字节，包括名（name）、值（value）和等号。
    &emsp;&emsp;Opera允许cookie多达4096个字节，包括：名（name）、值（value）和等号。
    &emsp;&emsp;Internet Explorer允许cookie多达4095个字节，包括：名（name）、值（value）和等号。
    注：多字节字符计算为两个字节。在所有浏览器中，任何cookie大小超过限制都被忽略，且永远不会被设置。
    
    + cookie 的属性
        ①. HttpOnly：该属性限制cookie仅能在http请求中被使用，通过javascript无法读取，可以有效防止`XSS`攻击
        ②. Secure：该属性限制仅在SSH链接时才会向服务端发送cookie，即尽有https协议中才会附带客户端cookie
        ③. Path：cookie生效路径，cookie仅在当前路径下可以被访问，该限制对javascript和http请求都生效，必须以`/`结束，是一个目录路径
        ④. Domain：cookie生成域名，cookie尽在该域名下可以被访问，同时对javascript和http请求生效
        ⑤. MaxAge：cookie失效时间，单位秒，`如果是整数，则在maxAge之后失效，如果是负数表示是临时cookie，关闭浏览器即会失效，同时会清除，如果是0表示删除该cookie，默认是-1`
        ⑥. Name： cookie键名
        ⑦. Value：cookie值
        ⑧. Comment：cookie使用说明
        ⑨. Version：cookie版本号
    + cookie 生命周期
        cookie通常在http请求服务器时，服务器通过`Set-Cookie`设置到客户端中，客户端与服务端通信时，http请求中会自动带上服务端可以访问到的cookie，服务端通过cookie进行验证处理，当cookie失效时，浏览器并不会立即清除cookie,会在窗口关闭后清除
    + cookie 应用范围
        仅存在于http协议中，其他TCP/IP 协议中是不存在的，虽然Websocket握手是基于Http协议的，仅可以在握手拦截器中获取到cookie去进行验证,通信过程中无法获取cookie
    + cookie 客户端使用方式
        读取：`document.cookie`，返回所有可以访问的cookie键值对字符串，以`；`连接
        设置：`document.cookie`，通过document.cookie将需要设置的cookie进行属性覆盖
        删除：通过`document.cookie`设置expire过期时间实现删除
2. sessionStorage
    sessionStorage与session是两个完全不同的概念，session是一个会话连接，通常配合cookie实现会话保持机制，作用在服务端；sessionStorage是客户端一个数据存储机制，以文件形式存在于浏览器中
    - 大小：通常浏览器中大小在`5M`左右，不同的浏览器略微不同，在2.5MB 到 10MB 之间
    - 操作：
        + 存储：sessionStorage.setItem(key:string,value:string|object);// 高版本`chrome`浏览器支持直接存储json对象，其他浏览器会自动将对象进行toString处理
        + 读取：sessionStorage.getItem(key:string); //`chrome`中如果设置的是对象，则读取出来的就是对象，其他浏览器必定是字符串 
        + 删除：sessionStorage.removeItem(key:string);
        + 清空：sessionStorage.clear();
        + 检查：sessionStorage.key(key:string);//判断是否有key
        + 键值对数量：sessionStorage.length
    - 阻塞：storage操作是同步操作，会阻塞上下文
    - 作用域：
        sessionStorage受同源策略影响，不可以跨域访问，sessionStorage受窗口影响，仅在当前窗口可以访问，即`同源同窗口`，窗口关闭后自动清除，刷新不会清除
    - 生命周期：
        `临时存储`，sessionStorage仅存在于当前窗口，窗口关闭即刻回收
3. localStorage
    localStorage于sessionStorage基本类似，可以参照sessionStorage去理解， 仅作用域、声明周期和事件存在区别
    - 作用域：
        localStorage受同源策略影响，不可跨域访问，但可以跨窗口访问，这是`跨窗口通信`的一种常用方法的实现原理
    - 生命周期：
        `永久存储`，如果不手动清除，数据会永久存储，不会自动回收
    - 事件：`storage`
        localStorage相比于sessionStorage多一个事件监听，这个事件在localStorage发生改变时会触发，这也是实现`跨窗口通信`的基础。此事件在`同Context(不能说同窗口)`中不会触发，在同一个上下文环境中执行监听和更新操作，是无法监听到事件触发的，只有在不同的上下文（包括不同窗口或者iframe）中分别进行更新个监听，才能正常触发该事件
4. [IndexDB](../database)
5. [WebSql](../database)
