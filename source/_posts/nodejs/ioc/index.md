---
title: IOC
categories: nodejs
abbrlink: 183e2cbd
date: 2020-05-16 17:55:42
top: true
---
## 前言
nodejs开发一直以来都是比较麻烦的事，相对于其他服务端语言，如常用的`Java`，开发过程就显得非常复杂，在其他语言中兴起了大量的注解、注入等思想，而由于nodejs是基于js语法，js中需要手动管理对象实例的创建回收，因而早期nodejs无法实现注解、注入的思想，但是随着`Es6装饰器`、`typescript`的兴起，js开发也逐渐变得类服务端化，跟服务端语言一样有具体的类型，接口等，因此在js上实现注解也成为了可能。本文主要介绍IOC的背景、原理、生命周期及简单实现。

## 背景
随着前端的发展，前端开发也已经变成了面向对象编程，而不是早期的面向函数编程。在面向对象设计的软件系统中，它的底层都是由N个对象构成的，各个对象之间通过相互合作，最终实现系统地业务逻辑。而面向对象开发就不可避免涉及到耦合性问题，对象之间耦合度过高的系统，必然会出现牵一发而动全身的情形。耦合关系不仅会出现在对象与对象之间，也会出现在软件系统的各模块之间，以及软件系统和硬件系统之间。如何降低系统之间、模块之间和对象之间的耦合度，是软件工程永远追求的目标之一。为了解决对象之间的耦合度过高的问题，软件专家Michael Mattson 1996年提出了IOC理论，用来实现对象之间的“解耦”，目前这个理论已经被成功地应用到实践当中。

## 概念
IOC是`Inversion of Control`的缩写，有些翻译过来叫`控制反转`，也有些叫`依赖注入（DI）`。
### 控制反转
&emsp;&emsp;IOC理论提出的主要观点是：借助`第三方`实现具有依赖关系的对象之间解耦，控制反转的意思是，在你需要生成对象的时候不用来调用我的构造函数new一个对象，而是我自己会在你需要的时候主动new一个对象传给你，简单说就是创建对象时主从关系反转，而控制反转主要由第三方`IOC容器`实现，系统中所需要的对象都交给第三方去控制，由`IOC容器`来控制对象的创建和复用从而做到对系统进行解耦。我们再来看看，控制反转(IOC)到底为什么要起这么个名字？我们来对比一下：
&emsp;&emsp;软件系统在没有引入IOC容器之前，如果对象A依赖于对象B，那么对象A在初始化或者运行到某一点的时候，自己必须主动去创建对象B或者使用已经创建的对象B。无论是创建还是使用对象B，控制权都在自己手上。
&emsp;&emsp;软件系统在引入IOC容器之后，这种情形就完全改变了，由于IOC容器的加入，对象A与对象B之间失去了直接联系，所以，当对象A运行到需要对象B的时候，IOC容器会主动创建一个对象B注入到对象A需要的地方。
&emsp;&emsp;通过前后的对比，我们不难看出来：对象A获得依赖对象B的过程,由主动行为变为了被动行为，控制权颠倒过来了，这就是“控制反转”这个名称的由来。
### 依赖注入
&emsp;&emsp;从上面可以知道IOC其实就是获得依赖对象的过程被反转了。控制被反转之后，获得依赖对象的过程由自身管理变为了由IOC容器主动注入。于是，`依赖注入（Dependency Injection）`这个名词也应运而生。实际上这个名词说出了实现IOC的方法：`注入`。所谓依赖注入，就是由IOC容器在运行期间，动态地将某种依赖关系注入到对象之中。
所以，依赖注入(DI)和控制反转(IOC)是从不同的角度的描述的同一件事情，就是指通过引入IOC容器，利用依赖关系注入的方式，实现对象之间的解耦。

## 优缺点
### 优点
1. 统一管理对象生成，在相同生命周期中进行复用，较少创建多实例开销，较少内存消耗。
2. 代码结构简单，各模块之间解耦，便于理解和维护。
3. 开发方便，开发效率提升，使用IOC后不需要太多关注不同对象之间的依赖，不需要去写创建对象的方法。
4. 为广泛使用的接口更改实现类比较简单（例如，用生产实例替换模拟web服务）
5. 实现拦截器、守卫等功能更加简单

### 缺点
1. 软件系统中由于引入了第三方IOC容器，生成对象的步骤变得有些复杂，本来是两者之间的事情，又凭空多出一道手续，所以，我们在刚开始使用IOC框架的时候，会感觉系统变得不太直观。所以，引入了一个全新的框架，就会增加团队成员学习和认识的培训成本，并且在以后的运行维护中，还得让新加入者具备同样的知识体系。
2. 由于IOC容器生成对象`基本`都是通过反射方式，在运行效率上有一定的损耗。如果你要追求运行效率的话，就必须对此进行权衡。
3. IOC框架产品本身的成熟度需要进行评估，如果引入一个不成熟的IOC框架产品，那么会影响到整个项目，所以这也是一个隐性的风险。
4. 额外的配置，使用`IOC`框架产品，必然需要进行大量的配制工作，比较繁琐，对于一些小的项目而言，客观上也可能加大一些工作成本。

## 原理
&emsp;&emsp;`IOC`中主要用到的技术是反射，在`.Net`、`Java`、`PHP5`中反射比较常见，用的也比较多，而在js和nodejs中反射并不常见，对于js而言，反射是新增的api，很多环境下不支持，而对于nodejs而言，反射用处并不大，目前因`IOC`的兴起而逐渐被广泛使用。Nodejs中最初实现`IOC`并发展的比较好的，首先就数`Nest`框架。`Nest`基于`Express`做了上层封装，整个框架采用了大量注入，目前使用量比较广泛，用户数也已经超过alibaba的`egg`，但目前alibaba也在基于`egg`做`IOC`上层封装，新的框架叫`midway`，只不过目前还不成熟。`Nest`是基于`Express`进行封装，而`midway`是基于`koa2`进行封装，在并发性能上`koa2`要超过`express`不少。当然`Nest`也能无缝从`Express`切换到`Fastify`(一个号称并发行最好的Nodejs服务中间件)，但其并没有大量流行使用，与已经发布的`koa3`相比结果也不明确，因此目前很多人还是比较看好`egg`及上层框架`midway`。
&emsp;&emsp;生成对象使用的是反射，而注入的实现，我们就需要知道目标对象的`元数据`，通过`元数据`我们才能从`IOC容器`中获取或者创建目标对象。`元数据`相对于前端来说比较抽象，因为js是弱语言，没有类型之分，而且也不支持`元数据`的修改和获取。随着`typescript`的广泛运用，前端工程是在Nodejs中实现`IOC`有了希望，通过`ES6 Decorator`技术以及`typescript`中的类型定义及编译后生成类型元数据再加上第三方库`reflect-metadata`终于凑齐了实现`IOC`的必要条件，因此，Nodejs中`IOC`也逐渐流行开来。
> `元数据`（Metadata）：为描述数据的数据，对数据及信息资源的描述性信息。元数据（Metadata）是描述其它数据的数据（data about other data），或者说是用于提供某种资源的有关信息的结构数据（structured data）。元数据是描述信息资源或数据等对象的数据，其使用目的在于：识别资源；评价资源；追踪资源在使用过程中的变化；实现简单高效地管理大量网络化数据；实现信息资源的有效发现、查找、一体化组织和对使用资源的有效管理。 元数据的基本特点主要有。例如`红苹果是一种红色的，圆的，吃起来可口的水果`，其中涉及到这样几个信息：颜色（红色）、形状（圆的）、口感（可口），有了这些信息，我们就可以大致想像出红苹果是个什么样的水果。推而广之，只要提供这几类的信息，我们也可以推测出其水果的样子。

主要技术：
- Decorator
- reflect-metadata

实现步骤：
1. 需要支持注入的类通过`Decorator`添加特定的元数据，以便能被scanner扫描到
2. 在使用的类中通过属性装饰器，将需要注入的对象类的元数据添加到该类的元数据上，以便于在创建该类的实体时scanner可以扫描到内部需要注入的对象信息从而进行对象注入
3. 定义scanner，服务启动阶段扫描文件，处理元数据，执行阶段分析元数据创建依赖对象，管理`IOC`容器等

## 生命周期
`IOC`的对象生命周期或者`Scope`通常分为三种，分别为`Request`、`Transient`、`SignleTon`，不同的框架中可能叫法不一样，但最终的生命周期都是一样的。
- `Request`：
顾名思义就是，生命周期就是与`请求链路`关联，请求建立开始，该实例创建，请求链路结束，该实例销毁，自动回收。当然并不意味着一定在请求建立时就立即创建对象，有可能是在请求链路过程中才开始创建，该Scope仅表示该类实例在整个请求链路中仅会实例化一次，后续都是复用之前创建的实例，在请求链路结束后立即销毁，释放内存。
- `Transient`：
顾名思义，生命周期比较短，仅能作用与当前类或提供者，意味着每个类或提供者中都要创建新的实例，不能跨类复用，等待自动回收，与请求链路没有关系，程序启动时实例化。
- `SignleTon`：
顾名思义，全局唯一，在整个进程中公用一个实例，该实例在服务启动阶段就会被`scanner`创建出来，即与`Application`生命周期关联。
- 生命周期覆盖：一个`Scope`的类中注入了另一个`Scope`的类实例就会发生生命周期覆盖，按照`Scope`的优先级的一定规则（`Request`>`Transient`>`SignleTon`）进行覆盖，例如：
    1. 一个提供器类中注入了一个`Scope`为`Request`实例，那么这个提供器会强制提升为`Request``Scope`，不管之前设置的是什么，这很好理解，属性需要在request链路中才能创建那么提前创建也没有意义
    2. 父`Request`、子`Transient`或`SignleTon`则互不影响，子仍然提前创建，父在请求链路中创建并注入已经创建好的子实例
    3. 父`Transient`、子`SignleTon`或者父`SignleTon`、子`Transient`，这两个互补影响，都是Application运行时创建，主要区别就是`Transient`有个局部作用与就是单个类的实例中复用

## 使用方式
```typescript
// utils.ts
@Injectable()
export class Utils {
    public test(){

    }
}
// userService.ts
@Injectable({scope:Scope.Request})
export class UserService {
    @Inject()
    private utils: Utils;

    public getUserIdBySession(){
        this.utils.test();
    }
}

// userController
export default class UserController {
    @Inject()
    private utils: Utils;

    @Inject()
    private userService: UserService;

    @Get('/info')
    private async getUserInfo() {
        const userId = this.userService.getUserIdBySession();
        this.utils.test();
        return 'success';
    }
}
```
在上述代码中，`Utils`类在被`scanner`扫描到后会立即创建一个该类的唯一实例，跟当前进程关联，`UserService`类由于`Scope`是`Request`，因此在请求被建立并用到该类对象时才会被创建，也就是在`UserController`被实例化时一起被创建且被注入到`UserController`实例中，而`UserController`和`UserService`中的`Utils`都是直接被注入启动时已经创建好的实例，当然，这仅仅是`IOC`中常规的使用方式，还有其他的例如构造函数注入，方法参数注入等等，基本原理都是一样的。

## 简单实现
上面介绍了`IOC`原理，和简单使用，下面为了更加了解`IOC`，我们通过简单的代码实现一套`IOC`框架。
1. `reflect-metadata` 第三方库引用
`reflect-metadata`第三方库是用于扩展js中`元数据`的设置和获取，js和nodejs中目前没有`元数据`相关的api，因此需要使用其他方式进行扩展，`reflect-metadata`本质上是使用`WeakMap`作为`元数据`容器，将类的`元数据`存储在容器中，并不是直接修改了类的描述`元数据`。

2. 注入装饰器实现
```typescript
// 提供者，可注入类
export interface InjectableOptions {
  /**
   * Specifies the lifetime of an injected Provider or Controller.
   */
  scope?: Scope;// Scope 类型见上文生命周期
}

/**
* 用于装饰支持注入的类，表示允许通过注入方式创建对象，添加该元数据方便scanner扫描并分析哪些类需要进行注入管理
**/
export function Injectable(options?: InjectableOptions): ClassDecorator {
  return (target: object) => {
    Reflect.defineMetadata(SCOPE_OPTIONS_METADATA, options, target);
  };
}

/**
* 用于装饰被注入属性，主要是将一个类的依赖注入数据都添加到该类的构造函数上，当Scanner创建该类时就可以直接读取依赖元数据进而创建并注入需要的依赖对象
**/
export function Inject<T = any>(token?: T) {
    return (target: object, key: string | symbol, index?: number) => {
        token = token || Reflect.getMetadata('design:type', target, key);// 获取typescript定义的类型装饰器，‘design:type’是typescript为属性类型专门创建的一个metadata
        const type =
            token && isFunction(token) ? ((token as any) as () => void).name : token;

        if (!isUndefined(index)) {
            let dependencies =
                Reflect.getMetadata(SELF_DECLARED_DEPS_METADATA, target) || [];

            dependencies = [...dependencies, { index, param: type }];
            Reflect.defineMetadata(SELF_DECLARED_DEPS_METADATA, dependencies, target);
            return;
        }
        let properties =
            Reflect.getMetadata(PROPERTY_DEPS_METADATA, target.constructor) || [];

        properties = [...properties, { key, type }];
        Reflect.defineMetadata(
            PROPERTY_DEPS_METADATA,
            properties,
            target.constructor,
        );
    };
}
```
3. 实例容器
根据上述`Scope`分类，可以直接将容器分为三个，部分框架直接使用一个容器来管理，对于注入的类创建一个单独的子实例池来控制。下面主要介绍单个容器的方式，其实容器的多少实现方式大同小异，仅注入实例时从哪个容器中取和添加需要不同的逻辑判断而已。简单的设计图如下：
{% asset_img container.png IOC容器 %}
核心代码实现：
```typescript
    // IOC 容器
    class IOCContainer{
        private _injectables = new Map<any, InstanceWrapper<unknown>>();// 实例池容器
        public addItem(metadataType:Type<any>){// 添加IOC容器管理类，通常scanner调用
            this._injectables.add(metadataType.name,new InstanceWrapper({
                name: metadataType.name,
                metatype: metadataType,
                isResolved: false,
                instance: null,
                host: this,
            }));// 创建该类的实例池
        }
        public getInjectables(metadataType:Type<any>){
            return this._injectables.get(metadataType.name);// 获取实例池
        }
    }

    export interface InstancePerContext<T> {
        instance: T;
        isResolved?: boolean;
        isPending?: boolean;
        donePromise?: Promise<void>;
    }

    class InstanceWrapper{
        private values = new WeakMap<any, InstancePerContext<unknown>>();// 实例池容器
        public loadInstanceByContextId(contextId:any){
            return values.get(contextId);
        }
        public addInstance(contextId:any,instance:any){
            // 简单设置
            values.set(contextId,{
                instance:instance,
                isResolved:true,
                isPending:false
            });
        }
    }
```
简化核心代码就如上，在基础上扩展异步构造方法的支持`isPending`属性控制等逻辑就可以形成完整的IOC框架，具体的可以参考[Nest.js源码](https://github.com/nestjs/nest)，或者参考我个人造的轮子[`flybirds`](https://github.com/yanxlg/sky)

## 补充
本文主要讲解`IOC`实现原理以及简单的代码，不贴大量源码，真实`IOC`框架需要考虑更多东西，例如构造函数入参注入，方法参数注入，异步注入等等

## 扩展
1. 回收机制：前端`IOC`中回收机制使用的是`WeakMap`进行管理，通过释放key的因哟过达到释放实例的目的。前端中`WeakMap`兼容行不好，因此前端目前没有类似`IOC`框架，一是因为影响前端性能，而是引入库较多，最终影响都是性能问题。而在Nodejs中已经支持了`WeakMap`，与客户端环境无关。
2. 前端某些框架中`@Inject`装饰器，例如`Mobx`，是属于`IOC`废弃设计`DL`开发的，属于依赖查找，并不是真实意义上依赖注入，仅仅是通过装饰器修改类，在原有组件类上Wrap了一层函数式组件，组件中查找需要的Mobx实例，作为props传递给真实类组件，而且Mobx中实例都是业务代码中主动创建并添加进去的，并不是第三方通过scanner扫描后创建的，`IOC`分为`DL（依赖查找）`和`DI（依赖注入）`两种，不过`DL`也已经废弃。
