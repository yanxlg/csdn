---
title: package.json
categories:
  - node
tags:
  - h5
  - node
  - package.json
top: true
abbrlink: 14f889f2
date: 2019-10-31 16:56:53
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
      - 可以使用[@scope](https://docs.npmjs.com/misc/scope)来创建`私有包`，例如`@yanxlg/rtl-tool`
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
7. `license`    
    版权，需要指定一个license让使用者知道他们的使用权利和限制，默认是`"ISC"`，常用的有`MIT`、`ISC`、`UNLICENSED`、`Apache-2.0`等，可以查看[SPDX许可证标识符的完整列表](https://spdx.org/licenses/)选择自己适合的许可证标识，推荐使用经过OSI核准的标识符，如果你的包在多个许可证下被授权，可以使用[SPDX许可证表达式语法v2.0](https://www.npmjs.com/package/spdx)(`OR`、`AND`、`WITH`)，例如
    ```json
    { "license" : "(ISC OR GPL-3.0)" }
    ```
    如果使用的许可未被授予SPDX吧标识符，或者使用自定义许可证，可以使用在当前包目录下创建一个<filenname>eg:LICENSE的文件，然后使用下列语法指定：
    ```json
    { "license" : "SEE LICENSE IN LICENSE" }
    ```
    如果不想让别人以任何方式使用该包，可以使用`UNLICENSED`标识符表明是`私有包`，对于其他人员不予授权
8. `author`、`contributors`  
    author表示作者，包的拥有者；contributors表示贡献者、参与者；author可以是一个对象或字符串，contributors可以是对象数组或字符串，对象格式如下：
    ```json
    {"author": {"name": "xlyan","email": "xlyan@email.com","url": "xlyan@home.com"}}
    ```
    其中email和url可以省略，当只有一个对象时可以简写成字符串，以`Space`隔开，author可以直接写成：
    ```json
    {"author": "xlyan xlyan@email.com xlyan@home.com"}
    ```
9. `files`  
    指定哪些文件被包含在包中，可以指定具体文件或文件名，是一个字符串数组，该字段在publish时比较重要，限制了哪些文件会被添加到发布的package中，如果不进行配置则表示所有文件均包括在包中，当然还有另一种方法忽略项目文件，就是使用`.npmignore`配置文件，ignore文件优先级相对较高，如果其中忽略了某个文件，不管files中是否包含都会被忽略
10. `main`  
    main指定模块的入口程序文件，是一个按照CommonJS规范生成的文件，通常指定为lib/index.js
11. `module`  
    module指定模块入口程序文件，是通过ES Module格式生成的文件，在导入时使用的`tree shaking`等特性，从而提升了打包性能，优先级高于main，目前尚未被加入到规范中，但编译工具都已支持，通常指定为es/index.js，文件中使用ES6特性，未被babel转换
12. `typings`  
    指定模块typescript声明文件，该方式是全局指定，也可以在每个js文件的同级目录创建一个d.ts文件，与js文件相关联
13. `bin`  
    可执行文件命令名和文件映射表，在安装包时会将可执行文件添加软连接到系统中，如果是全局安装则会关联到prefix/bin中，如果是本地安装则会链接到./node_modules/.bin中。全局安装后可以在全局使用指定的命令，本地安装仅可在当前安装目录下使用，设置方式:
    ```json
    {"bin": {"bin1": "./bin/bin1","bin2": "./bin/bin2"}}
    ```
14. `man`  
    指定一个单一的文件名或一个文件名数组来让man程序使用。通常指定一个文档数组，很少使用
15. `directories`
    CommonJS Packages规范说明了几种你可以用directories对象来标示你的包结构的方法。如果你去看[npm's package.json](https://registry.npmjs.org/npm/latest)，你会看到它标示出出doc、lib和man。在未来，这个信息可能会被用到，暂时没有实际意义。
16. `repository`
    代码托管位置，对于想要源码的人很有帮助，通常指定type和url两个字段或者使用缩写，例如
    ```json
    {"repository": {"type": "git","url": "https://github.com/npm/npm.git"}}
    ```
17. `scripts`
    脚本命令，配置可运行的npm命令，在前端或node项目中非常有用，支持`pre`和`post`命令前后钩子，如：
    ```json
    {"scripts": {"prestart": "./prefix.js","start": "react-dev-tools start","poststart": "./clear.js"}}
    ```
    优先执行pre命令在pre命令执行完成后执行指定的命令，执行完成后执行post命令，到此，你运行的命令才会结束
18. `config`
    一个配置对象，用来配置包脚本中跨版本参数，如指定port等包的配置参数，也可以使用`npm config`或`yarn config`命令配置，如：
    ```shell
        npm config set foo:port 8001
    ```
    这就是指定foo模块的config中port值为8001
19. `dependencies`
    指定依赖的包名和版本号列表，用于存放生产使用和非测试的包，在该包被安装时所有dependencies都会同步被安装，具体版本号写法可以参考下面[版本说明](#版本说明)
20. `devDependencies`
    如果需要使用到某个包，但是不需要使用其外部测试和文档，则将其放在devDependencies中比较合适，通常建议与`dependencies`的区分为，工具和测试类package放在`devDependencies`中，最终build需要的源码package放在`dependencies`中，仅供参考，不强制要求。
21. `peerDependencies`
    peerDependencies是做为一个插件的功能被使用，意思是`如果你安装我，那么你最好也安装X,Y和Z`。其他依赖配置中的包都会被安装在当前包的node_modules下，而不是安装在项目根目录到额node_modules下，因此直接使用会存在问题：
    ```dir
    MyProject
    |- node_modules
       |- PackageA
          |- node_modules
             |- PackageB
    ```
    PackageA中未在peerDependencies中引用PackageB就会出现上面的目录结构，导致项目中无法直接使用PackageB，而如果使用peerDependencies则会变成：
    ```dir
      MyProject
      |- node_modules
         |- PackageA
         |- PackageB
    ```
    此时在项目中可以直接使用PackageB
22. `bundledDependencies`
    在发布包时，包名的数组会被打包进去。会添加到dependencies中
23. `optionalDependencies`
    如果一个依赖项可用，但希望在这个依赖项无法被找到或者安装失败时npm还能继续处理(不中断)，那么你可以把它放在optionalDependencies中。和dependencies一样，optionalDependencies是一个包名和版本号或url的映射。区别在于optionalDependencies中的依赖构建失败时不会导致npm整体安装失败
24. `engines`
    你可以指定node的工作版本：  
    ```json
    { "engines" : { "node" : ">=0.10.3 <0.12" } }
    ```
    和dependencies类似，如果你不指定一个node版本(或者你用'*'指定)，则任何一个node版本都可以。  
    如果你指定了一个'engines'字段，则npm将会在某处包含这个node版本。如果忽略'engines'字段，则npm只会仅仅假设这个包工作在node下。  
    你还可以使用'engines'字段来指定可以安装这个包的npm版本，举个栗子：
    ```json
    { "engines" : { "npm" : "~1.0.20" } }
    ```
    请注意，除非用户设置了`engine-strict`标记，否则这个字段只是一个建议值。
25. `os`
    指定模块运行的操作系统，是个字符串数组，可以使用`darwin`、`win32`、`linux`或者可以加`!`实现黑名单
26. `cpu`
    指定模块只能在特定的cpu架构上运行，如：
    ```json
    {"cpu": ["x64", "ia32"]}
    ```
    当然，也可以使用`!`的方式实现黑名单
27. `preferGlobal`
    如果你的包是一个需要作为全局安装的命令行应用，则需要指定该属性，在拒不安装是会给与警告，但不会阻止用户拒不安装。
28. `private`
    第三种声明包为私有包的方式，是一个boolean值，true表示是私有包，false表示非私有
29. `publishConfig`
    发布配置项，在发布时如果npm publish后面需要加其他配置参数，可以将其配置在该项中，然后直接使用npm publish即可，通常用来配置access、tag、registry等
    
<hr>

## 版本说明
   dependencies中的包支持多种方式声明：
   1. `npm 仓库方式`：直接指定版本号，按照版本号格式进行配置
   2. `URLs 做为依赖项`：指定一个压缩包的url，在install时会自动下载该压缩包并安装
   3. `Git URLs 做为依赖项`：指定一个git仓库的地址为依赖路径，可选形式如下:
      ```
      git://github.com/user/project.git#commit-ish
      git+ssh://user@hostname:project.git#commit-ish
      git+ssh://user@hostname/project.git#commit-ish
      git+http://user@hostname/project/blah.git#commit-ish
      git+https://user@hostname/project/blah.git#commit-ish
      ```
        commit-ish可以是任何tag、sha或者branch，并作为一个参数提供给git进行checkout，默认值是master。
   4. `GitHub URLs 做为依赖项`:从1.1.65版本开始，你可以引用Github urls作为版本号，比如"foo": "user/foo-project"。也可以包含一个commit-ish后缀，举个栗子：
        ```json
        "dependencies": {
            "express": "visionmedia/express",
            "mocha": "visionmedia/mocha#4727d357ea"
        }
        ```
   5. `本地路径 做为依赖项`：从版本2.0.0开始你可以提供一个包的本地路径。本地路径可以在你使用npm install -S或npm install --save时被保存，具体形式如下：  
       ```
        ../foo/bar
        ~/foo/bar
        ./foo/bar
        /foo/bar
       ```
        这个特性有助于当你不想从一个外部服务器安装npm包的情况，比如本地离线开发和创建测试，但最好不要在发布包到公共registry时这样使用。
        
<hr>

## 版本格式
   1. version 必须确切匹配这个version
   2. \>version 必须大于这个version
   3. \>=version 必须大于等于这个version
   4. < version 必须小于这个version
   5. <=version 必须小于等于这个version
   6. ~version 大约相当于version，参考[semver](https://docs.npmjs.com/misc/semver)，会优先升级`patch`版本号，但是不会升级`minor`版本号
   7. ^version 与version兼容，参考[semver](https://docs.npmjs.com/misc/semver)，会升级`minor`版本号，但不会升级`major`版本号
   8. 1.2.x 可以是1.2.0、1.2.1等，但不能是1.3.0
   9. \* 匹配任何版本
   10. ""(空字符串) 匹配任何版本，和\*一样
   11. version1 - version2 相当于 >=version1 <=version2
   12. range1 || range2 range1或range2其中一个满足时采用该version
   13. tag 一个以tag发布的指定版本，参考[npm-tag](https://docs.npmjs.com/cli/tag)
