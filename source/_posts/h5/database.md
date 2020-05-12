---
title: database
tags:
  - h5
  - database
  - webSql
  - indexDB
categories:
  - h5
  - database
top: true
abbrlink: c953062e
date: 2019-01-29 14:59:40
---
##前言
web端对于数据的存储需求逐渐增多，越来越多的网站开始考虑，将大量数据储存在客户端,减少从服务器获取数据，直接从本地获取数据。cookie、localStorage、sessionStorage只能存储少量数据，对于大量数据或者一些框架代码的存储来说显得无力，同时不提供搜索功能，不能建立自定义的索引，因此客户端数据库的概念被h5引入。
## Database
1. Web Sql：  
    Web Sql是W3C在H5规范中最早提出来的一个数据库理念及设计思想，但是由于后期规范完全按照SQLlite来定制，最后导致该标准被废弃，目前仅webkit部分浏览器进行了支持，主流的Chrome内核浏览器都进行了支持，Firefox等浏览器不支持该规范，因此`使用时需要考虑兼容性`,`移动端webview中兼容性较好`，`高版本IOS中兼容被放弃`，具体兼容性可以[查看](https://www.caniuse.com/#search=web%20sql)
    - 定位： 关系型数据库，支持Sql
    - 兼容性：兼容性不佳，`低版本移动端中兼容性较好，高版本IOS已放弃兼容`
    - 同源策略限制：受同源策略影响，不可跨域访问
    - 数据库大小
       部分文档中注明Web Sql数据库的大小是`50M`，但是并不准确，不同的浏览器存在差异，有些远远不止50M，并没有一个固定的标准，虽然数据库方面有这样的限制，但是可以通过创建不同的数据库实例来解决该问题，大小限制仅对单个实例有效
    - API
        + 创建数据库实例：openDatabase(name:string,version:string,description:string,size);// 可以指定数据库名称、版本号、描述、大小
        + 事务处理：transaction(execute:(ctx)=>void,onsuccess:()=>void,onerror:()=>void);// 第一个参数是事务内容的方法，在其内部调用executeSql去执行相关操作，第二个参数是事务执行成功的回调，第三个参数是事务执行失败的回调。当事务执行失败时会自动回滚到初始状态
        + 执行sql：executeSql(sql:string,values:string[],onsuccess:()=>void,onerror:()=>void); //  第一个参数是sql语句字符串（SQLlite语法），第二个参数是sql中用到的`?`替换值，第三个参数是执行成功回调，第四个是执行失败回调
    - 第三方库：[html5sql.js](http://kencorbettjr.github.io/html5sql/)
        html5sql.js对websql操作进行了友好性封装，使得前端人员在开发过程中使用更加方便
2. Index DB：  
    Index DB是H5后期发展过程中诞生的一个新的客户端本地数据库，可以存储大量数据，该数据库在主流浏览器中兼容性原来越好，仅低版本存在一些问题，相关兼容性如下[查看](https://www.caniuse.com/#search=Index%20DB)
    - 定位：非关系型数据库，类似于NoSQL
    - 兼容性：兼容性较好，主流浏览器都已支持，移动端兼容性较好，除非常低的版本意外全部兼容
    - 数据库大小：  
        部分文章中指出Index DB大小在250M以上，不同的浏览器存在差异，甚至无大小上线。即使存在上限，也可以按照Web Sql的方式通过多个实例解决上线问题
    - 特性：  
        ①. 键值对存储，采用对象仓库存放数据，可以直接存储对象  
        ②. 异步，数据库操作都是异步操作，放置大量数据操作阻塞浏览器，这与{% post_link h5/storage storage %}不同  
        ③. 事务，Index DB 同样支持事务操作  
        ④. 同源，Index DB受同源策略影响，不可跨域访问 
        ⑤. 支持二进制存储，Index DB支持对象存储，对象中包括ArrayBuffer和Blob，因此可以存储二进制文件
    - API：所有操作都是异步的，api返回的都是request
        + 创建（打开）数据库：indexDB.open(name:string,version:number);// 指定数据库名及版本号
        ```typescript
          const request = window.indexedDB.open("databaseName",1);// 打开或创建数据库
          request.onsuccess=function(event){
              const db = request.result;// db实例
          };
          request.onerror=function(event){
              // 创建失败回调
          };
          request.onupgradeneeded=function(event) {
              // 当创建数据库或打开数据库，但指定版本大于实际版本时，触发数据库升级回调
          }
        ```
        + 创建对象仓库（表）：db.createObjectStore(tableName:string,config:{keyPath?:string;autoIncrement?:boolean});// keyPath主键可以是一级属性，也可以是下一层属性，例如`foo.bar`。`该方法必须在onupgradeneeded事件中调用`
        ```typescript
          // 创建表必须在open的upgradeneeded回调中执行
          openRequest.onsuccess=function(event) {
              const db = openRequest.result;
              // 创建表时需要先判断该表是否存在
              if (!db.objectStoreNames.contains('tableName')) {
                  const objectStore = db.createObjectStore('tableName', { keyPath: 'id' });// 可以通过keyPath指定主键，也可以使用autoIncrement自动生成主键
              }
          }
        ```
        + 创建索引：objectStore.createIndex(indexName:string,keyPath: string | string[],config:{unique?:boolean;multiEntry?:boolean})// unique指定属性值是否包含重复的值，multiEntry如果为true，则会给keyPath数组中每一个元素创建一个索引，否则给整个数组创建一个索引
        ```typescript
          objectStore.createIndex('name', 'name', { unique: false });
          objectStore.createIndex('email', 'email', { unique: true });
        ```
        + 事务：db.transaction(storeNames: string | string[], mode?: "readonly" | "readwrite");// 需要指定涉及到的对象仓库（表），以及操作模式。整个事务成功需要在`oncomplete`中监听
        ```typescript
          const transaction = db.transaction(['person'], 'readwrite');
          transaction.onsuccess=function(event) {
            
          };
          transaction.onerror=function(event) {
                       
          };
          transaction.oncomplete=function(event) {
              // 事务成功在complete中监听，onsuccess之后仍然可能会失败
          }
        ```
        + 新增数据：objectStore.add(object,`key?:any`);// 对象仓库（表）直接使用add添加数据对象，需要在事务中使用，第二个参数标识某条记录的键，不指定默认为null
        ```typescript
          const request = db.transaction(['person'], 'readwrite').objectStore("person").add({name:"Wang"});
          request.onsuccess = function (event) {// add request 成功监听
              console.log('数据写入成功');
          };
          request.onerror = function (event) {
              console.log('数据写入失败');
          }
        ```
        + 读取数据：objectStore.get(key:any|IDBKeyRange);// 通过主键值读取数据，对应获取多条记录的方法是`getAll(key:any|IDBKeyRange,count?:number)`,key可以是一个具体的值，也可以是一个范围,IDBKeyRange可以通过其静态方法生成
        ```typescript
          const request = db.transaction(['person'], 'readonly').objectStore("person").get(1);
          request.onsuccess = function (event) {// get request 成功监听
              console.log('数据读取成功'+request.result);
          };
          request.onerror = function (event) {
              console.log('数据读取失败');
          }
        ```
        + 遍历数据：objectStore.openCursor();// 通过游标request来遍历数据
        ```typescript
          const objectStore = db.transaction('person').objectStore('person');
          objectStore.openCursor().onsuccess = function (event) {
              const cursor = event.target.result;
              if (cursor) {
                  console.log('Id: ' + cursor.key);
                  console.log('Name: ' + cursor.value.name);
                  console.log('Age: ' + cursor.value.age);
                  console.log('Email: ' + cursor.value.email);
                  cursor.continue();
              } else {
                  console.log('没有更多数据了！');
              }
          };
        ```
        + 更新数据：objectStore.put(data:object);//更新数据需要使用put，add在主键存在时会抛错
        ```typescript
          const request = db.transaction(['person'], 'readwrite').objectStore('person')
              .put({ id: 1, name: '李四', age: 35, email: 'lisi@example.com' });
          
          request.onsuccess = function (event) {
              console.log('数据更新成功');
          };
          request.onerror = function (event) {
              console.log('数据更新失败');
          }
        ```
        + 删除数据：objectStore.delete(keyValue:any);// 通过主键值删除数据
        ```typescript
          const request = db.transaction(['person'], 'readwrite')
              .objectStore('person')
              .delete(1);
          
          request.onsuccess = function (event) {
              console.log('数据删除成功');
          };
        ```
        + 使用索引进行查询：objectStore.index(indexName:string).get(keyValue:any);// 通过建立的索引去查询值为keyValue的数据
        ```typescript
          const transaction = db.transaction(['person'], 'readonly');
          const store = transaction.objectStore('person');
          const index = store.index('name');
          const request = index.get('李四');

          request.onsuccess = function (e) {
              var result = e.target.result;
              if (result) {
                  console.log("获取到指定的数据");
              } else {
                  console.log("未找到指定数据");
              }
          }
        ```
    - 第三方库：[Dexie.js](https://dexie.org/)
