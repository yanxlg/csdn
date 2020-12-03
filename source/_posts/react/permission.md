---
title: 权限设计
abbrlink: 5a8a6c8d
date: 2020-07-11 17:50:13
tags: react
---
## 前言
大型前端系统功能比较繁杂，需要针对不同类型的用户开放不同的功能，因此前端需要根据不同的权限生成`不同的页面`，`不同的路由`，`不同的菜单`，甚至`不同的交互`等等，这就是前端权限系统所需要包含的内容。

## 常规方案
在通常的前端项目中，不管是单页面还是jquery项目，一般都是通过给需要控制的组件添加id的方式进行组件控制，前端加载页面时会先获取用户的权限列表（可缓存），然后根据权限列表去针对每个受控组件进行交互控制，组件的id为扁平化方式管理，组件之间没有父子关系，这种方式比较简单，抽象出复用代码其实就等价与`if...else...`。

## 通用性需求
- 在常规方案中存在一些缺陷：
1. 页面中很多类似组件，由于扁平化设计，命名id时比较麻烦，需要规避已有的（`接口校验`），但也要保持一定的语义化，因此通常采用长驼峰或者uri格式命名，保持一定的层级结构，例如`账号管理/角色管理/新增角色`按钮，id通常命名为`account/role/add`形式。
2. 常规方案较简单，后台仅实现了权限的有和无，前端根据权限的处理方式完全由前端代码`hard code`处理，通用性不强，配置性不强

- 健壮性需求：
我们开发一个权限系统，需要考虑兼容性，扁平化模式属于基本模式，需要支持，同时需要支持树形权限数据，并且抽象出通用权限交互，支持后台权限配置时即可配置前端交互

## 方案设计
1. 需要根据实时权限数据更新页面视图，因此需要使用React中的Provider设计。
2. 树结构的支持需要在React中获取到父组件，通过获取权限组件链，构成真实权限路径（`path`），此实现需要对于React底层有一定的了解，了解FiberNode的相关属性，具体可参考{% post_link react/react `React` %}
3. 需要支持交互配置，因此权限数据结构设计如下：
```typescript
/**
 * 控制类型，权限组件支持多样化控制方式，可权限数据中配置，可组件属性中配置，权限数据优先级>组件属性
 * render 组件级控制，不实例化组件实例
 * display 元素级控制，实例化组件实例，但不渲染元素，与render的区别在于是否执行组件的初始化及部分生命周期；部分场景可能需要使用，例如埋点等，虽然不可见但是会触发某些操作
 * visibility 元素级控制，隐藏元素
 * disabled 元素级控制，不可点击
 **/
export type ControlType =
    | "render" // render 控制组件创建
    | "display" // 控制是否显示，UI级别比render低
    | "visibility" // 控制是否隐藏，UI级别比display 低
    | "disabled" // 控制是否可交互，经常用于按钮
    | "tooltip"; // 用自定义的toolTip组件包裹，通常用于做提示显示，toolTip交互由传入组件自己控制
/**
 * 权限数据元素
 * access：是否拥有该权限，默认值为true，当为false时等同于没有改权限，但是需要支持自定义属性及样式，因此需要该属性来实现控制
 * control：控制类型，`页面级别不生效`
 * styles：权限有无时额外样式属性，在一定程度上可以覆盖control的行为，需要受控组件支持style属性传入，`页面级别不生效`
 *   active：有权限时自定义样式
 *   inActive：无权限时自定义样式
 * classNames：权限有无时额外样式表，在一定程度上可以覆盖control的行为，需要受控组件支持className属性传入，`页面级别不生效`
 *   active：有权限时自定义样式表
 *   inActive：无权限时自定义样式表
 * deprecated：该权限是否被废弃
 * children：子权限列表
 * ==============================================
 * 无权限数据两种方式：
 * 一：没有该item=>从组件及Provider读取control属性，默认值为render
 * 二：item的access设置为false，从该item读取control属性及自定义属性
 * ==============================================
 */
export type PermissionItem = {
    access?: boolean; // 是否有权限，默认值为true，
    control?: ControlType; // 设置控制类型，不同控制类型对应不同的UI交互
    styles?: {
        active?: React.CSSProperties;
        inActive?: React.CSSProperties;
    }; // styles 可用于指定有权限和无权限自定义显示样式
    classNames?: {
        active?: string[];
        inActive?: string[];
    };
    deprecated?: boolean; // 该权限是否废弃，即可用于临时关闭某些权限限制，某些场景需要
    children?: {
        [key: string]: PermissionItem;
    };
    toolTip?: string; // control 为tooltip时使用属性
};

export type IPermissionTree = {
    [key: string]: PermissionItem;
};
```
    - 结构支持扁平化或`树`数据类型
    - 支持配置交互样式
    - 支持配置交互行为

4. Provider 设计
```typescript
export type IPermissionTree = {
    [key: string]: PermissionItem;
};

type IToolTipWrap = React.ReactElement<{ title: string }>;

declare interface IPermissionProviderProps {
    pTree?: IPermissionTree; // 拥有的权限列表
    format?: "tree" | "flat"; // 权限数据结构类型
    defaultControl?: ControlType;
    toolTipWrap?: IToolTipWrap;
    defaultToolTip?: string;
    checkLogin?: () => boolean; // 外部传入用于判断登录状态的方法
    history?: H.History;
    Page_403?: React.ReactNode; // 403 Page
}

export declare interface IPermissionContext extends IPermissionProviderProps {
    updateTree: (tree: IPermissionTree) => void;
}

const Context = React.createContext<IPermissionContext>(null);

const Provider: React.FC<IPermissionProviderProps> = ({ pTree, children, ...props }) => {
    const [tree, updateTree] = useState(pTree || {});
    return <Context.Provider value={{ pTree: tree, updateTree, ...props }}>{children}</Context.Provider>;
};

export default React.memo<PropsWithChildren<IPermissionProviderProps>>(Provider);

export { Context };
```
- Provider支持传入额外属性
1. `format`：控制权限是扁平化还是`树`数据结构处理，默认是`树`数据结构，因为本文推荐使用`树`结构
2. `defaultControl`：默认控制类型，如权限数据中不指定`control`属性将使用该属性来控制权限组件交互，默认值为`render`
3. `toolTipWrap`：当`control`为`tooltip`时需要的组件，用于展示tooltip用，本文是基于`ant-design`开发，因此，建议使用的是其`Tooltip`组件，由于组件配置属性较多，不同项目可能需要定制不同属性，因此设计为传入组件实例，使用方式`<Tooltip title="" trigger={'click'} />`，当然也可以使用其他组件或者自定义组件实例，只要满足`IToolTipWrap`定义
4. `defaultToolTip`：当`control`为`tooltip`时显示的默认提示，可统一配置
5. `checkLogin`：当页面组件需要登陆时，不完全依赖服务端接口去进行登陆校验，需要前端提前一步进行粗略拦截，如未登陆则自动重定向到`/login`页面
6. `history`：内部使用页面跳转需要的history实例
7. `Page_403`：页面组件如果无权限时使用的替代页面，内部已存在默认`403页面`，可外部使用其他组件实现自定义
8. 大部分全局配置可在服务端配置实现，修改时不需要修改前端代码重新发布

## 核心代码
1. 组件级权限组件
```jsx
// ReactInstanceMap.ts
import React from "react";

/**
 * copy react-dom/packages/share  code to use in outside
 */

declare interface FiberNode {
    elementType: any;
    pendingProps: any;
    return: any;
}

declare global {
    namespace JSX {
        interface Element {
            _owner?: FiberNode;
        }
    }
}

export function getOwner() {
    const ownChild = <></>;
    return ownChild._owner;
}
```
```jsx
import React, { ReactElement, useContext, useMemo } from "react";
import { IPermissionContext, ControlType, Context, IPermissionTree, PermissionItem } from "./Provider";
import * as assert from "assert";
import classNames from "classnames";
import { getOwner } from "./ReactInstanceMap";

export declare interface IPermissionChildProps {
    disabled?: boolean;
    style?: React.CSSProperties;
    className?: string;
}

export declare interface PermissionProps {
    pid: string;
    fallback?: () => React.ReactElement; // 无权限时替代组件
    // toolTip?:string;
    children?: ReactElement<IPermissionChildProps>;
    control?: ControlType; // 默认control
    styles?: {
        active?: React.CSSProperties;
        inActive?: React.CSSProperties;
    };
    classNames?: {
        active?: string[];
        inActive?: string[];
    };
    toolTipWrap?: IPermissionContext["toolTipWrap"];
    toolTip?: string;
}

// 需要缓存及局部缓存
const getChildren = (
    pItem: PermissionItem | boolean,
    children: ReactElement,
    defaultControl: ControlType,
    customStyles?: PermissionItem["styles"],
    customClassNames?: PermissionItem["classNames"],
    fallback?: PermissionProps["fallback"],
    toolTipWrap?: PermissionProps["toolTipWrap"],
    defaultToolTip?: string,
    toolTip?: string,
    extraProps?: any
) => {
    // 设置children属性
    if (pItem === true) {
        return children; // 废弃
    }

    const item: PermissionItem =
        typeof pItem === "boolean"
            ? {
                  control: defaultControl,
                  access: pItem
              }
            : pItem;

    const { access = true, styles, classNames: classes, control } = item;

    const style = access ? styles?.active : styles?.inActive;

    const customStyle = access ? customStyles?.active : customStyles?.inActive;
    const customClassName = access ? customClassNames?.active : customClassNames?.inActive;
    const mergeClassName = classNames(children.props?.className, access ? classes?.active : classes?.inActive, customClassName);

    const mergeStyle = {
        ...children.props?.style,
        ...(control === "display" && !access ? { display: "none" } : control === "visibility" && !access ? { visibility: "hidden" } : {}),
        ...style,
        ...customStyle
    };

    const mergeProps = {
        ...children.props,
        style: mergeStyle,
        className: mergeClassName,
        ...(control === "disabled" && !access ? { disabled: true } : {}),
        ...extraProps
    };
    // fallback优先级最高
    if (!access && fallback) {
        return fallback();
    }
    const _toolTip = item.toolTip || toolTip || defaultToolTip;
    if (control === "tooltip" || toolTip) {
        // 组件props上tooltip强制覆盖
        assert(_toolTip, "Permission ToolTip control must have tooltip property.");
        assert(toolTipWrap, "Permission ToolTip control must have toolTipWrap component.");

        // fix Popconfirm
        if ("onConfirm" in mergeProps) {
            mergeProps.disabled = true;
        } else if (mergeProps.popConfirmProps && "onConfirm" in mergeProps.popConfirmProps) {
            mergeProps.popConfirmProps.disabled = true;
        }

        if (_toolTip && toolTipWrap && !access) {
            // children 为disabled状态
            const child = React.cloneElement(children, { ...mergeProps, onClick: undefined, _privateClick: true }, children.props?.children); // onClick去掉
            if (children.props?.disabled) {
                return child;
            }
            return React.cloneElement(
                toolTipWrap,
                {
                    title: _toolTip
                },
                child
            );
        }
    }
    if (control === "render" && !access) {
        return null;
    } else {
        return React.cloneElement(children, mergeProps, children.props?.children);
    }
};

export const getPidList = (instance: any, format: IPermissionContext["format"], pid: string) => {
    if (format === "flat") {
        return [pid]; // 扁平化，根键值对
    } else {
        let permissions = [];
        let parent = instance;
        while (parent) {
            if (parent.elementType === PermissionComponent || parent.elementType?.type === PermissionComponent) {
                permissions.unshift(parent.pendingProps.pid);
            }
            parent = parent.return;
        }
        return permissions;
    }
};

/**
 * 判断是否有权限
 * @param pidList
 * @param pidMap
 * @return boolean|PermissionItem  true:废弃，不做处理，false：无权限，PermissionItem：权限控制Item
 */
export const checkPermission = (pidList: string[], pidMap: IPermissionTree) => {
    const length = pidList.length;
    let access: boolean | PermissionItem = false,
        i = 0,
        next = pidMap,
        cur = next[pidList[i]];
    while (!access && cur && i < length) {
        if (cur.deprecated) {
            // 废弃
            access = true; // 对于废弃的不做权限处理
        } else {
            i++;
            if (i < length) {
                next = cur.children;
                if (!next) {
                    cur = undefined;
                    i = length; //退出循环
                } else {
                    cur = next[pidList[i]];
                }
                if (i === length && cur) {
                    access = cur.deprecated ? true : cur;
                }
            } else {
                access = cur.deprecated ? true : cur;
            }
        }
    }
    return access;
};

const PermissionComponent = (props: PermissionProps) => {
    const { pid, fallback, children, control, styles, classNames, toolTipWrap: _toolTipWrap, toolTip, ...extra } = props;
    const context = useContext(Context);
    assert(context !== null, "Permission Component must be used by wrapped with PermissionProvider");
    assert(pid, "Permission Component must have pid");
    assert(children, "Permission Component must have children");
    const { format, pTree } = context;

    const owner = getOwner();
    const pidList = useMemo(() => getPidList(owner, format, pid), []);
    const check = useMemo(() => checkPermission(pidList, pTree), [pTree]);
    const toolTipWrap = _toolTipWrap || context.toolTipWrap;
    // 缓存创建节点
    return useMemo(() => {
        return getChildren(check, children, control || context.defaultControl || "render", styles, classNames, fallback, toolTipWrap, context.defaultToolTip, toolTip, extra);
    }, [children, pTree]);
};

const usePermissionFilter = (
    items: Array<{
        pid: string;
        [key: string]: any;
    }>
) => {
    const context = useContext(Context);
    const { format, pTree } = context;
    const owner = getOwner();

    return useMemo(() => {
        return items.filter(item => {
            const pidList = getPidList(owner, format, item.pid);
            const check = checkPermission(pidList, pTree);
            return check === true || (typeof check === "object" && (check.access === void 0 || check.access === true));
        });
    }, [items, pTree]);
};

export default React.memo<PermissionProps>(PermissionComponent);

export { usePermissionFilter };
```
2. 页面级权限组件
```jsx
import React, { PropsWithChildren, useCallback, useContext, useMemo } from "react";
import { Context } from "./Provider";
import assert from "assert";
import * as H from "history";
import { getOwner } from "./ReactInstanceMap";
import { checkPermission, getPidList } from "./ComponentPermission";

export declare interface IRouterPermission {
    login?: boolean; // 是否需要登录
    history?: H.History;
    pid: string;
}

const RouterPermission: React.FC<IRouterPermission> = ({ login, pid, history, children }) => {
    // 是否登录，是否拥有该页面权限，可能是子页面
    const context = useContext(Context);
    const { checkLogin, format, pTree, Page_403 } = context;
    if (login) {
        assert(checkLogin, "login check require checkLogin function.");
        const isLogin = checkLogin?.();
        if (!isLogin) {
            assert(history || context.history, "redirect must have history property.");
            // 未登录
            (history || context.history)?.replace("/login"); // 跳转登录，登录页面支持重定向
            return null;
        }
    }
    // 判断路由权限
    const owner = getOwner();
    const pidList = useMemo(() => getPidList(owner, format, pid), []);
    const check = useMemo(() => checkPermission(pidList, pTree), [pTree]);

    const goHome = useCallback(() => {
        (history || context.history)?.replace("/");
    }, []);

    if (check === true || (typeof check === "object" && check.access !== false)) {
        return <>{children}</>;
    } else {
        return Page_403 ? (
            <>{Page_403}</>
        ) : (
            <div>
                <div style={{ textAlign: "center", color: "rgba(0,0,0,.85)", fontSize: 24, lineHeight: 1.8 }}>403</div>
                <div
                    style={{
                        color: "rgba(0,0,0,.45)",
                        fontSize: 14,
                        lineHeight: 1.6,
                        textAlign: "center"
                    }}
                >
                    Sorry, you don't have access to this page.
                </div>
                <div style={{ textAlign: "center" }}>
                    <button
                        onClick={goHome}
                        style={{
                            color: "#fff",
                            background: "#1890ff",
                            borderColor: "#1890ff",
                            textShadow: "0 -1px 0 rgba(0,0,0,.12)",
                            boxShadow: "0 2px 0 rgba(0,0,0,.045)",
                            textAlign: "center",
                            cursor: "pointer",
                            height: 32,
                            padding: "4px 15px",
                            fontSize: 14,
                            borderRadius: 2,
                            border: "1px solid #d9d9d9",
                            outline: "none"
                        }}
                    >
                        Back to home
                    </button>
                </div>
            </div>
        );
    }
};

export default React.memo<PropsWithChildren<IRouterPermission>>(RouterPermission);
```
3. `decorators`：为了简化类组件的使用，提供装饰器模式支持
```jsx
/**
 * decorator + wrap support
 * 支持同一类组件中权限控制
 **/

import React from "react";

import { ReactElement } from "react";
import ComponentPermission, { IPermissionChildProps, PermissionProps } from "./ComponentPermission";
import RouterPermission, { IRouterPermission } from "./RouterPermission";

const ComponentPermissionWrap = (children: ReactElement<IPermissionChildProps>, props: Omit<PermissionProps, "children">) => {
    return <ComponentPermission {...props} children={children} />;
};

const RouterPermissionWrap = (children: React.ReactNode, props: IRouterPermission) => {
    return <RouterPermission {...props} children={children} />;
};

type RouterPermissionProps = {
    router: true;
} & IRouterPermission;

type ComponentPermissionProps = {
    router?: false;
} & Omit<PermissionProps, "children">;

type IPProps = RouterPermissionProps | ComponentPermissionProps;

// type RouterClass = React.ComponentClass<any, any>;

type ComponentClass<T extends IPermissionChildProps> = React.ComponentClass<T>;

type IIPermissionChildProps = IPermissionChildProps & {
    [key: string]: any;
};

/**
 * Class Decorator
 * @param config
 * @constructor
 */
function Permission<T>(config: IPProps) {
    const { router, ..._config } = config;
    if (router) {
        return function wrapWithConnect<T extends any>(WrappedComponent: ComponentClass<T>) {
            return (function(props: any) {
                return RouterPermissionWrap(<WrappedComponent {...props} />, _config);
            } as unknown) as any;
        };
    } else {
        return function wrapWithConnect<T extends IIPermissionChildProps>(WrappedComponent: ComponentClass<T>) {
            return (function(props: any) {
                return ComponentPermissionWrap(<WrappedComponent {...props} />, _config);
            } as unknown) as any;
        };
    }
}

export { Permission };
```
4. `wrapper`：函数式组件无法使用`decorator`，因此提供额外`wrapper`方式
```jsx
const PermissionRouterWrap = (Component: React.ComponentClass<any, any> | React.FC<any>, config: IRouterPermission) => {
    return (props: any) => {
        return (
            <PermissionRouter {...config}>
                <Component {...props} />
            </PermissionRouter>
        );
    };
};

const PermissionComponentWrap = <T,>(Component: React.ComponentClass<T, any> | React.FC<T>, config: Omit<PermissionProps, "children">) => {
    return (props: T) => {
        return (
            <PermissionComponent {...config}>
                <Component {...props} />
            </PermissionComponent>
        );
    };
};
```

## demo使用
1. tree 结构使用
```jsx
<PermissionProvider
    pTree={
        {
            "111": {
                children: {
                    "2222": {
                        access: false,
                        styles: {
                            active: {
                                backgroundColor: "red"
                            },
                            inActive: {
                                backgroundColor: "green"
                            }
                        }
                    }
                }
            }
        } as IPermissionTree
    }
>
    <div>
        dsads
        <PermissionComponent pid={"111"}>
            <div>
                <PermissionComponent pid={"2222"} toolTipWrap={toolTip} toolTip={"我是提示信息"}>
                    <div>122</div>
                </PermissionComponent>
                <B text="yyyyyyy" />
            </div>
        </PermissionComponent>
    </div>
</PermissionProvider>
```
2. `umijs`项目中使用
```jsx
// app.tsx
export function rootContainer(container: any) {
    return React.createElement(
        PermissionProvider,
        {
            format: 'flat',
            checkLogin: () => !!User.token,
            history: history,
            Page_403: <Page />,
            pTree: User.pData, // 缓存权限列表
            toolTipWrap: <Tooltip title="" trigger={'click'} />,
            defaultToolTip: '账号无此权限，请联系管理员！',
        },
        container,
    );
}
```
```typescript
// 页面组件
export default PermissionRouterWrap(GoodsAttr, {
    login: true,
    pid: 'setting/goods_attr',
});
```
```jsx
//交互组件
<PermissionComponent
    pid="setting/export/delete"
    control="tooltip"
>
    <CloseOutlined
        className={exportStyles.exportClose}
        onClick={() => deleteFile(id)}
    />
</PermissionComponent>
```

## 注意点
1. 由于受控组件外部嵌套一层权限组件，如之前父组件中使用了React.cloneElement或者React.createElement会将属性添加到权限组件中，权限组件已尽量将不想管属性向下传递，需要的属性命名尽量不常用，但也会出现偶然性冲突问题，需要在使用时注意。
2. 不支持扁平化与`树`结构混用，推荐使用一种方式，而不是混合使用
3. `树`结构如果出现多个页面重复功能时，如两个页面都有审核功能，主观上设计应该避免，同样的入口会对用户造成迷惑性，但一般来说还是不可避免会出现这种情况，在使用`树`结构时需要每个页面下都有该功能权限配置，唯一比较好的办法是配置页面中将该权限提升到与页面同级，如果选中则分别映射到两个页面下。
