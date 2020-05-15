---
abbrlink: c72f21a0
title: React Hooks
tags: react
categories: react
date: 2019-9-20 13:20:41
---
## 前言
React随着16版函数式组件的推广，hooks普及率也越来越高，其实钩子函数在React中一直存在，只是以前不对外使用，React在底层每个声明周期都有前置钩子和后置钩子，因此hooks在React中尤为重要。

## Hooks
- useContext：用于和React中的Context特性配合使用，可以直接在函数式组件中绑定Context的值，并订阅更新
    ```tsx
        const GlobalContext = React.createContext<IGlobalContext>(null);

        const GlobalContextProvider: React.FC = props => {
            const contextRef = useRef(null);
            const [context, _setContext] = useState<IGlobalContext['context']>({
                age: 0,
            });
            useEffect(() => {
                contextRef.current = context;
            }, [context]);

            const setContext = useCallback((ctx: Partial<IGlobalContext['context']>) => {
                _setContext({
                    ...contextRef.current,
                    ...ctx,
                });
            }, []);

            const setAge = useCallback(age => {
                setContext({
                    age: age,
                });
            }, []);

            const wrapChild = useMemo(() => {
                return props.children;
            }, [props]);

            return useMemo(() => {
                return (
                    <GlobalContext.Provider value={{ context, setContext, setAge }}>
                        {wrapChild}
                    </GlobalContext.Provider>
                );
            }, [props, context]);
        };

        export { GlobalContext, GlobalContextProvider };

        // 父容器，Context需要在Provider中才生效，具体Context此处不做详细介绍
        const WrapIndex = () => {
            return (
                <GlobalContextProvider>
                    <Component />
                </GlobalContextProvider>
            );
        };

        // 组件中使用
        const Component=()=>{
            const {age,setAge} = useContext(GlobalContext);
        }
    ```
- useState：state管理，可以复合对象，可以扁平化，但是要`注意性能影响`，异步中多个setState同时调用会触发多次render，需要配合useMemo、useEffect等节流hooks优化或者使用useRef优化，需要了解React State更新合并机制
    ```typescript
        const [{name,age},setAllState] = useState({name:"Wang",age:1});// 复合数据管理可以在一定成都上有效控制重复render问题，扁平化管理在于代码简单，但是需要额外的优化控制
    ```
- useReducer：生成一个简易Redux
    ```typescript
        const initialState = {
            additionalPrice: 0,
            car: {
                price: 26395,
                name: "2019 Ford Mustang",
                image: "https://cdn.motor1.com/images/mgl/0AN2V/s1/2019-ford-mustang-bullitt.jpg",
                features: []
            },
            store: [
                { id: 1, name: "V-6 engine", price: 1500 },
                { id: 2, name: "Racing detail package", price: 1500 },
                { id: 3, name: "Premium sound system", price: 500 },
                { id: 4, name: "Rear spoiler", price: 250 }
            ]
        };
        const reducer = (state, action) => {
            switch (action.type) {
                case "REMOVE_ITEM":
                return {
                    ...state,
                    additionalPrice: state.additionalPrice - action.item.price,
                    car: { ...state.car, features: state.car.features.filter((x) => x.id !== action.item.id)},
                    store: [...state.store, action.item]
                };
                case "BUY_ITEM":
                return {
                    ...state,
                    additionalPrice: state.additionalPrice + action.item.price,
                    car: { ...state.car, features: [...state.car.features, action.item] },
                    store: state.store.filter((x) => x.id !== action.item.id)
                }
                default:
                return state;
            }
        }
        // 组件中使用
        const Component = () => {
            const [state, dispatch] = useReducer(reducer, initialState);

            // 读取值：state.additionalPrice === 0
            // 触发action
            dispatch({ type: 'BUY_ITEM', {} });
        }
    ```
- useRef：创建Ref引用，仅初始化一次，相当于在函数中定义了`一个不会重新初始化的变量`。该特性经常被用到，能有效解决变量存储问题，不被用于触发render的数据通常使用useRef来存储，避免触发render
    ```typescript
    const dataRef = useRef({});// 初始值为空对象
    // 取值
    dataRef.current;
    ```
- useLayoutEffect：与`useEffect`具有相似的功能，具体用法参考[useEffect](#useEffect)；但是又有区别，在类组件中`componentDidMount`和`componentDidUpdate`是同步触发的，在render结束后就执行，会阻塞浏览器painting，因此在类组件这两个生命周期中直接操作DOM不会出现DOM闪烁现象。而`useEffect`是异步触发的，并不会阻塞浏览器painting，为了分离js与渲染线程来提升性能，因此在`useEffect`中进行DOM操作会出现DOM闪烁，通常不会用到该Hook，React中不推荐去直接操作DOM，使用动画库除外。
- <span id="useEffect">useEffect</span>
    1. 触发场景：
        - 在组件render之后异步触发，不会阻塞浏览器painting，
        - 组件卸载前执行`清除副作用`，与依赖数组无关
    2. 参数：
        - 第一个参数是执行函数，可以返回副作用清除函数（例如事件绑定，定时器等等清除）
        - 第二个参数配置依赖数组，如果配置了依赖数组，仅当数组中值发生变化时才会触发当前Effect。
    3. 机制：
        - 副作用清除是在新的useEffect执行时调用上一次的useEffect副作用清除
        - 组件卸载前仅执行副作用清除函数

- useImperativeHandle：声明函数式组件`ref`实例，在类组件中，ref可以访问类组件的成员方法，而在函数式组件中需要通过该Hook声明`ref`实例
    ```typescript
    const Component = (props: ComponentPropsType, ref: Ref<ComponentRefType>) => {
        const [value,setValue] = useState(0);
        useImperativeHandle(
            ref,
            () => {
                return {
                    setValue: value => {
                        setValue(value);
                    },
                    getValue:()=>{
                        return value;
                    }
                };
            },
            [value],
        );
    }
    export default React.forwardRef(Component);
    ```
    1. 参数：
        - 第一个参数：ref引用
        - 第二个参数：ref实例工厂函数
        - 依赖，内部使用到的动态状态值需要添加到依赖中，不然只会访问初始值。
    2. 组件需要使用`React.forwardRef`包裹
- useCallback：函数生成器，常规的函数写法在每次组件更新时都重新生成一个新的函数，useCallback会记录原函数，如果依赖参数没有变化则不会重新创建函数，直接返回原有函数，减少创建消耗，但在js中其实创建消耗很小，再加上使用useCallback会存在检测依赖过程，优化结果不明显，可用可不用，但是推荐使用，在函数体较大时还是有优势的
- useMemo：[`常用Hook`]，常用于控制代码是否执行，或是否重新渲染
    1. 最常规用法，用于控制组件render，当deps不发生变化时不会重新render，直接返回原有虚拟节点
        ```typescript
        const Cmponent = ()=>{
            return useMemo(()=>{
                return <div></div>
            },[]);// 仅执行一次render
        }
        ```
    2. 延伸用法（业务部分）,可以实现在渲染前根据条件进行一些处理，相对于不用useMemo可以进行deps比较，类似于React@16-中类组件的`componentWillReceiveProps`声明周期及16+中`getDerivedStateFromProps`静态声明周期，用来判断前后状态修改并执行相关操作，不同于`useEffect`的是该Hooks触发于render之前
       ```typescript
        const Cmponent = ({}:CmponentProps)=>{
            useMemo(()=>{

            },[]);
            return useMemo(()=>{
                return <div></div>
            },[]);// 仅执行一次render
        }
        ```
- useDebugValue：用于将label显示在React 调试工具`DevTools`中，调试用，编译会自动忽略
- useResponder：[实验性Hook]功能目前不明
- useDeferredValue：[实验性Hook]跟`useTransition`类似，也是延迟更新作用，相当于在新数据准备好之前，可以继续沿用旧数据，如果配置时间内新数据来了，（从旧内容切换到）显示新内容，否则立即更新状态，该 loading 就 loading，即预加载。
- useTransition：[实验性Hook]延迟操作，通常用于延迟更新，例如loading效果，如果速度极快时loading会一闪而过，这种场景可以使用useTransition来延迟更新loading状态，如果极快时不会显示loading
- useMutableSource：[实验性Hook]不依赖Context，构建全局状态管理，目前全局状态仅可通过Context来管理，新版的redux本质上也是context
- useOpaqueIdentifier：[实验性Hook]功能目前不明

## 原理
Hooks用法比较简单，原理也需要做相应了解，一般面试可能会涉及到相关只是
- Hooks与组件的关系：React在渲染时会生成一个虚拟节点树（JSON，现在为Fibber链表），一个组件中所有的Hooks信息（称之为`memoizedState链表`）会存储对应组件Fibber节点上，跟组件生命周期关联，一同创建，一同销毁。
- Hooks存储方式：单向链表，Rect中目前链表结构使用比较频繁，某些方面类似于数组，有些人在手写React源码时会用数组方式实现，其实并不正确，单向链表没有索引，仅存在游标，通过游标及next可以遍历整个链表，与数组相似的地方在于都是可遍历结构，不同的地方是数组可以根据索引获取任意位置的值，单向链表从HEAD开始仅能向下
- Hooks 阶段：`mount`和`update`，分别对应组件创建和更新不同生命周期，在`mount`阶段创建链表节点并添加到Fiber节点属性上，在`update`阶段主要判断deps，更新链表值，返回原有值或新值。
- Hooks不能嵌套在`if语句`、`for语句`、`子函数`等不定语句中的原因：`memoizedState链表` 是按hook定义的顺序来生成数据链的，且仅在`mount`阶段生成一次，后续`update`阶段并不会重新初始化`memoizedState链表`，仅会从链表中通过游标cursor按个返回现有状态，所以如果Hooks顺序变化或者数量变化，`memoizedState`并不会感知到，也不能去更新。
- `"Capture Value"` 特性是如何产生的：`"Capture Value"`特性就是通常说的状态记忆特性，对于deps未发生变化的Hooks，在update阶段会返回原有的callback或者useMemo结果并不会返回新传入的callback，因此useCallback调用时还是调用的原方法，原方法的作用域下所有变量都是原状态（除`useRef`引用外，因此通常也用useRef来存储变量，提升性能），因此形成了状态记忆。主要原因是函数作用域，并不是闭包。
- 自定义的 Hook 是如何影响使用它的函数组件的：自定义Hooks其实本质上就是一个函数，在执行时内部Hooks跟组件共用一个`memoizedState链表`
- 链表节点上属性介绍
    - memoizedState：最新值
    - baseQueue：稳定值，如果最新值render时抛出异常，即会使用该稳定值进行数据回溯，防止页面崩溃
    - naxt：链表指针，只想下一个链表节点
- setState第二个解构值是更新队列的一个dispatch方法，更新队列本文不做介绍
- 简单图解
{% asset_img img-left mount.png mount阶段 %}
{% asset_img img-left update.png update阶段 %}

## 优化注意
1. State合并：很多时候一个组件中需要使用多个useState来分别存储不同的状态管理，但是需要注意React的setState触发render机制{% post_link react/react React底层原理分析 %}，在同步函数中如果多次调用setState则只会触发一次render，如果在异步中多次调用则触发n次render ,因此针对代码中出现的异步中多次setState的现象需要进行合并，合并总体方法就是较少useState的使用，或者render使用useMemo节流；
    1. useState通过复合对象统一管理组件的所有状态
        ```typescript
            const [state, setState] = useState({
                total: 0,
                list: []
            });
        ```
    2. 通过`useRef`存储数据，useState控制render的触发
        ```typescript
            const listRef = useRef([]);
            const totalRef = useRef(0);
            const [loading, setLoading] = useState(false);// 控制render的进行

            useEffect(()=>{
                setLoading(true);
                // 异步中处理
                Promise().then(()=>{
                    listRef.current = [...list];
                    totalRef.current = 10;
                    setLoading(false);
                });
            },[]);
            
        ```
    3. 通过`useMemo`节流：需要注意useMemo的deps选择，应该选异步中最后一个setState的变量，否则在render时取到的total值会是错误的，该方式会触发组件函数重新执行，仅不会触发render，因此该方法相对于上面两种较差
        ```typescript
            const [list, setList] = useState([]);
            const [total, setTotal] = useState([]);
            useEffect(()=>{
                // 异步中处理
                Promise().then(()=>{
                    setTotal(10);
                    setList([...list]);
                });
            },[]);
            return useMemo(()=>{},[list]);// 仅当list更改后才会render，与total无关
        ```
2. 请求优化：目前前端流行的请求库如axios,umi-request都带有cancel功能，可以直接取消请求，而fetch最初计划基于Promise提供的cancel去实现请求cancel，后来因某些原因Promise的cancel一直未提供，因此有了不支持cancel这一败笔；目前前端基本不直接使用fetch，而是使用axios或umi-request，对于请求，前端合理的操作是页面加载时调用请求，如果页面卸载则取消对应请求，节省带宽，提升性能，并较少React中setState造成的已卸载组件不合理render。对于卸载组件render问题大部分都是在组件卸载前将setState方法置成空方法，而在函数式组件中可以使用自定义Hook实现请求回收
    1. 基于请求库封装实现一个可以cancel的请求创建类
        ```typescript
        class Fetch{
            public cancel(){
                // cancel该实例发送的所有请求
            }
            public get(...args){}
            public post(...args){}
            public put(...args){}
            ...
        }
        ```
    2. 自定义Hooks->useFetch
        ```typescript
        function useFetch(clean=true){// 配置clean参数，支持不清空
            const fetchRef = useRef<Fetch>(new Fetch());
            useEffect(() => {
                return () => {
                    // 组件卸载时执行的clean副作用
                    clean && fetchRef.current.cancel();
                };
            }, []);
            return [fetchRef.current];
        }
        ```
    3. 组件中使用
        ```typescript
        const Component = ()=>{
            const [fetch] = useFetch();

            useEffect(()=>{
                getDataService(fetch).then(()=>{});// 将fetch实例传递给service，service需要从外部接收fetch实例，将service与组件关联
            },[]);
        }
        ```