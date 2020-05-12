---
title: ReactNative Optimize
abbrlink: 42d065b1
date: 2019-10-19 15:00:12
password: mikemessi
message: 本文暂不开放，需要密码才可阅读.
wrong_pass_message: 密码错误，请输入正确的密码.
---
受到react机制影响
React 中渲染优化机制通常是基于PureComponent或者手动shouldComponentUpdate实现，但是在RN中存在一个问题，父组件render时，子组件声明周期中this.props.children和nextProps.children永远不相等，但是也没有创建新实例，很奇怪的现象，如果children用一个变量或属性引用保存，又不会出现该现象
