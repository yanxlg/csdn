---
abbrlink: 5c7548aa
title: React 模块化服务端渲染
tags: react
categories: react
date: 2018-9-20 13:20:41
password: ssr_yxlg_module
message: 本文暂不开放，需要密码才可阅读.
wrong_pass_message: 密码错误，请输入正确的密码.
---
## 前言
  随着MVC框架的兴起，其在搜索引擎和首屏渲染方面的缺陷也越来越明显，该方面的需求也越来越迫切。因此服务端渲染技术应运而生，服务端渲染本质上是在服务端拼装完成html字符串或流直接返回给前端显示，其不在该文章中具体介绍，详细了解可以参考{% post_link react/ssr ssr %}文章详细了解。React 服务端渲染可以使用的方式有很多，可以直接使用react-dom 提供的serverToString等原生方法自己搭建服务端渲染服务，也可以使用目前已经存在的相对较稳定的轮子`Next.js`、`react-ssr`等。但是不管是自己实现还是使用轮子都会遇到各种各样的问题。对于常见问题的解决即属于前端架构层面的设计

## 场景
  在自主开发服务端渲染时通常在每个路由对应的页面组件上挂载一个静态async loadData方法（类似于Next.js的getInitialProps）。这样，在服务端渲染的时候，会根据路由匹配出当前页面组件，调用loadData方法，获取初始状态数据，然后将其传递给store用于服务端渲染及同构。但这个页面内如果有子组件想要独立获取自身的数据，这样的设计就有问题了，因为只能根据路由匹配出当前页面组件，无法匹配出其内部组件，react-dom中也没有提供相应向下遍历方法，而且父组件中有哪些自组件仅在实例话阶段才可以明确。服务端渲染前还未实例话，因此无法获取到子组件中的loadData方法。

## 初级想法
  既然自组件无法获取，那么就将自组件需要用到的初始化数据放在页面组件的loadData方法中，页面在引入自组件时，将其loadData方法同时添加到页面组件的loadData中执行，然后页面组件再将子组件需要的状态通过props或者context(redux系列本质上就是context)传递给子组件。这种设计是最简单也是最常见的设计，目前流行的react服务端渲染轮子中基本都是这种设计，即仅能支持页面组件中预请求数据。但是这种设计存在很大的缺陷：
  - 违背了低耦合的原则，组件内的数据交互最好是由它自己独立完成。
  - 复用组件管理相当麻烦，每添加或删除一个子组件在页面组件中都要添加或删除其对应的loadData方法，增加开发管理难度。
  - 模块化不彻底，如果组件层级嵌套过深，整个项目架构会相当`deep`。

## 进一步思考
  在服务端可以获取到页面组件是由Router路由表中根据当前的url匹配出来的，如果将子组件按照一定的规则也同样配置到路由中，那么在服务端渲染的时候就可以根据设计的规则获取到所有匹配的组件，其中包括配置的子组件，这样也就能执行其loadData方法。但这种设计跟上述实现思路一样麻烦，每新增或删除一个子组件都要去修改路由。综合比较该方法与第一种方法没有明显区别。

## 配置式（约定式）设计
  在使用子组件时，都将用到的自组件关联到父级组件静态属性中，服务端渲染时根据该静态属性一层层向下遍历出所有对应的组件并调用其loadData方法，该设计方案总体来说改进了前两种方案，子组件仅跟父组件耦合，而不是所有自组件都和页面组件耦合，从架构设计来说要简化不少。但仍没有完全解决耦合性问题。

## 深思
  要想满足这样的场景需求，本质上就是要做到根据页面组件遍历出所有用到的子组件，但是在未执行或实例未初始化阶段无法做到获取其子组件列表，子组件跟页面组件在`元信息`（`IOC常用`）上并没有关联，因此我们需要想一些办法来获取到页面组件中的子组件，上述的方案都是基于手动配置来完成，目前，前端方面也逐渐工具化，常用的`webpack`、`gulp`、`fis3`、`grunt`、`babel`等，为了进一步解决上述问题，是不是可以在工程化方面做些优化？例如，第一种方案，通过AST树解析其render中用到的所有React组件，并筛选符合条件的自动添加其loadData方法到页面组件中；对于第二种方案，可以通过AST解析，自动添加到路由中；第三种方案将自组件构造函数或类自动添加到父组件的某个静态属性上；从分析来看，三种方案通过AST解析或者使用node开发一个文件扫描器应该都可以实现，相对来说第三种工程化相对简单，因此引出`第一种设计方案`：
  >在[第三种方案](#配置式（约定式）设计)的基础上开发一个工程化插件，在编译阶段自动解析子组件并将其添加到父组件的静态属性中。

## 进一步解耦
  `第一种设计方案`需要在编译阶段修改原有文件及原有React组件类，在开发阶段完全没有耦合性，但是编译生成的文件还是有耦合性的，其实只要我们知道其依赖关联，完全没有必要将这些关联添加到原有组件上，通过文件之间关联关系json我们就可以解析出引用了哪些文件（并不一定是组件，如果单独分析组件需要额外处理）；因此这个关联json的生成显得尤为重要。如果熟悉webpack的话，应该很容易就想要webpack中提供的`stats.json`其实就是我们想要的基础json，我们常用的`analyze`功能就是基于`stats.json`开发的可视化页面，其中详细的列出了每个页面及文件中的依赖文件，因此我们可以在此基础上加上处理逻辑，生成需要的`ssr_dep.json`。

## 扩展
  - [`stats.json`详解](https://www.webpackjs.com/api/stats/)
  - stats筛选：
    1. stats是一个复杂的json文件，常规项目如果按照默认配置生成stats.json，生成的stats.json会非常大，包含了大量该需求中不需要的信息，因此需要进行过滤，转换。
    2. stats生成配置
    ```json5
    "statsOptions":{
        source:false,// 删除源码，减少体积
        excludeModules:[/node_modules/,/\.less$/,/\.css$/,/\.umi-production/],// 过滤掉不需要的module
        children: false,
        context: '/',// 生成的stats中以绝对路径显示
        entrypoints: false,
        chunkGroups: false,
        assets: false,// 不显示assets
        chunks: true,// 需要通过chunks解析出路由，如果知道路由列表是配置的可以设置为false，根据配置的路由表去处理即可
        reasons: false// 删除不需要的reasons列表
    }
    ```
    3. 可以在编译时使用webpack内置插件bundle-analyzer 生成stats.json，也可以自定义webpack插件，在插件中接收stats对象进行处理，我在实际项目中是开发webpack插件进行处理的，主要是为了方便，由于插件发布在公司私有仓库，不便直接贴代码，下面只会讲解主要处理方式及部分`demo code`
  - json处理：
    1. json生成：使用bundle-analyzer插件生成或在自定义插件中接收，自定义插件不做介绍，自行扒贴
    2. flat 所需要的module列表：
    ```typescript
    const modules = stats.modules;// 取出stats中的modules列表，进行处理
    let flatModules =[];// 存储过滤后的module
    function flat(module) {
        if(module.name&&module.issuerName){
            // 过滤调不需要处理的module，根据需要，我们仅需要当前项目src下的module，不需要node_modules或css类型的module，因此可以全部过滤掉
            if(!/\.(less|css)/.test(module.name)||!/\.(less|css)/.test(module.issuerName)||!/\.umi/.test(module.issuerName)||!/node_modules/.test(module.issuerName)){
                flatModules.push({
                    name:module.name,// 模块全路径
                    issuerName:module.issuerName,// 引用该模块的文件全路径
                    issuerPath:module.issuerPath,// 引用tree列表，可以向上查询到入口文件
                })
            }
        }else if(module.modules){
            module.modules.map((_)=>{flat(_)});
        }
    }
    modules.map((_)=>{flat(_)});
    console.log(flatModules);// 所有处理过的modules
    ```
    3. 获取所有路由列表：`不做代码介绍`
        1. 可以手动配置，需要知道路由文件绝对路径，解析时根据绝对路径去解析
        2. 可以开发文件扫描器，解析出所有的配置式路由文件路径
        ```
     4. 生成每个路由对应的组件依赖
     ```typescript
        const tree = routes.map(route=>{
            let components = [];
            flatModules.map(item=>{
                if(item.issuerPath.find(_=>_.name===route)){
                    components.push({
                        id:item.id,
                        name:item.name,
                        issuerName:item.issuerName,
                        usedExports:item.usedExports
                    });
                }
            });
            return {
                route,
                components
            }
        });
    ```
    5. 将js中的组件静态方法暴露给Nodejs，可以使用装饰器的方式，或者wrap方式
    ```typescript
        function ssrPreload(target:any, key:any, descriptor:any) {
          if(!__isBrowser){// 判断在node环境才执行
            // @ts-ignore
            if(global.ssrPreloadList){
              // @ts-ignore
              global.ssrPreloadList.push(target);// 将所有的loadData方法添加到node中
            }
          }
          return descriptor;
        }
        class Components extends React.Component<any, any>{
          @ssrPreload
          public static async loadData(){
            console.log("1111");
          }
          render(){
            return <div>I want to test</div>
          }
        }
    ```
    6. 生成每个路由文件对应loadData方法映射
    ```typescript
    tree.map(({ route, components }) => {
      let routeAsyncList = [];
      components.map(({ name, id }) => {
        const component = global._inject(id);// `假设存在该方法可以从编译后的js中获取内部module`
        // 遍历所有属性，可能为default
        const keys = Object.keys(component);
        keys.map(key => {
          const target = component[key];
          global.ssrPreloadList.map(item=>{
              if(item === target){
                routeAsyncList.push(target);
              }
          });
        });
      });
      return routeAsyncList[];
    });
    ```
    7. 根据前端请求req的上下文，解析出对应的路由文件绝对路径，此处不做介绍
    8. 在服务端渲染请求过来时结合`6、7`获取需要预加载的方法集合并调用获取数据
  - 尽管上面的步骤磕磕碰碰，但总归是有了思路，当然不同项目中可能存在部分问题，包括stats.json解析，都需要进行联调修改，上述仅作为参考代码
  - 目前唯一存在的问题就是从服务端js中获取内部模块，如果无法获取内部模块，一切白搭，上述的`global._inject`其实就是webpack生成的server.js中`webpackBootstrap`里创建的`__webpack_require__`方法，因此需要想办法针对`webpackBootstrap`处理，通常处理`webpackBootstrap`的方法就是改生成的文件代码，因为`webpackBootstrap`也是字符串拼接生成的，想要对代码字符串进行修改就要保留代码的唯一性，即需要保留原始的`__webpack_require__`，因此server.js生成时需要关闭`最小化`。
    1. webpack生成的module代码
    ```js
        module.exports =
    /******/ (function(modules) { // webpackBootstrap
    /******/ 	// The module cache
    /******/ 	var installedModules = {};
    /******/
    /******/ 	// The require function
    /******/ 	function __webpack_require__(moduleId) {
    /******/
    /******/ 		// Check if module is in cache
    /******/ 		if(installedModules[moduleId]) {
    /******/ 			return installedModules[moduleId].exports;
    /******/ 		}
    /******/ 		// Create a new module (and put it into the cache)
    /******/ 		var module = installedModules[moduleId] = {
    /******/ 			i: moduleId,
    /******/ 			l: false,
    /******/ 			exports: {}
    /******/ 		};
    /******/
    /******/ 		// Execute the module function
    /******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
    /******/
    /******/ 		// Flag the module as loaded
    /******/ 		module.l = true;
    /******/
    /******/ 		// Return the exports of the module
    /******/ 		return module.exports;
    /******/ 	}
    /******/
    /******/
    /******/ 	// expose the modules object (__webpack_modules__)
    /******/ 	__webpack_require__.m = modules;
    /******/
    /******/ 	// expose the module cache
    /******/ 	__webpack_require__.c = installedModules;
    /******/
    /******/ 	// define getter function for harmony exports
    /******/ 	__webpack_require__.d = function(exports, name, getter) {
    /******/ 		if(!__webpack_require__.o(exports, name)) {
    /******/ 			Object.defineProperty(exports, name, { enumerable: true, get: getter });
    /******/ 		}
    /******/ 	};
    /******/
    /******/ 	// define __esModule on exports
    /******/ 	__webpack_require__.r = function(exports) {
    /******/ 		if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
    /******/ 			Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
    /******/ 		}
    /******/ 		Object.defineProperty(exports, '__esModule', { value: true });
    /******/ 	};
    /******/
    /******/ 	// create a fake namespace object
    /******/ 	// mode & 1: value is a module id, require it
    /******/ 	// mode & 2: merge all properties of value into the ns
    /******/ 	// mode & 4: return value when already ns object
    /******/ 	// mode & 8|1: behave like require
    /******/ 	__webpack_require__.t = function(value, mode) {
    /******/ 		if(mode & 1) value = __webpack_require__(value);
    /******/ 		if(mode & 8) return value;
    /******/ 		if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
    /******/ 		var ns = Object.create(null);
    /******/ 		__webpack_require__.r(ns);
    /******/ 		Object.defineProperty(ns, 'default', { enumerable: true, value: value });
    /******/ 		if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
    /******/ 		return ns;
    /******/ 	};
    /******/
    /******/ 	// getDefaultExport function for compatibility with non-harmony modules
    /******/ 	__webpack_require__.n = function(module) {
    /******/ 		var getter = module && module.__esModule ?
    /******/ 			function getDefault() { return module['default']; } :
    /******/ 			function getModuleExports() { return module; };
    /******/ 		__webpack_require__.d(getter, 'a', getter);
    /******/ 		return getter;
    /******/ 	};
    /******/
    /******/ 	// Object.prototype.hasOwnProperty.call
    /******/ 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
    /******/
    /******/ 	// __webpack_public_path__
    /******/ 	__webpack_require__.p = "/";
    /******/
    /******/
    /******/ 	// Load entry module and return exports
    /******/ 	return __webpack_require__(__webpack_require__.s = 0);
    /******/ })
    /************************************************************************/
    /******/ ({/**modules**/})
    ```
    2. 修改生成的js，将`__webpack_require__`暴露出来
    ```js
      const serverJsContent = fs.readFileSync('./dist/server.js');
      const updateJs = serverJsContent.replace(/__webpack_require__\.m/,"global._inject=__webpack_require__; __webpack_require__.m");
    ```
    3. 修改生成文件需要在编译完成后立即执行，因此可以配置脚本来进行build一系列步骤
  - 到这里，需求总算是实现了，但是相当麻烦，我们当初也仅仅是针对公司项目定制了模块，并不一定可以适配各种项目结构和类型，例如如果服务端js打包时不是生成一个文件，而是多个文件，上述实现方式要根据需要进行`大调整`，如果服务器渲染直接以dev的方式在服务器上运行暂时没有比较好的办法


## 总结
  - 具体项目是可以实现该需求，但是想要开发成适合所有项目的轮子那绝对是`大坑`，还不如使用配置的方式，在引用的地方通过静态属性添加引用关系，毕竟vue就是使用配置声明方式的啊