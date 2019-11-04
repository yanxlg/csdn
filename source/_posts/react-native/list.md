---
title: RN Long List
categories: ReactNative
abbrlink: 9b9cdea5
date: 2019-09-26 23:28:24
---
 React Native 开发中经常遇到长列表场景，不仅包括无限加载瀑布流，还包括聊天列表等等，大量的列表元素堆积在页面中时对于Application的内存消耗非常严重，严重影响App的性能，因此一直以来对于长列表的优化在RN中经常涉及，本文主要介绍官方的一些长列表组件使用方式及优秀的第三方组件的使用

### FlatList
1. FlatList是官方基于ScrollView 优化的一个长列表组件，在一定程度上解决了长列表占用系统大量内存，造成系统卡顿的问题，但是性能比不上RecyclerListview。
2. 使用过程中会出现FlatList onEndReached事件频繁触发的问题，此处主要记录该问题的引发原因及解决方案
    - 父组件高度没有铺满整个屏幕，通常需要将父组件高度设置为100%，使用flex:1并不能完全起作用
    - items数量不足：onEndReached事件触发是根据当前滚动位置举例底部的距离计算比例与onEndReachedThreshold比较，如果在其范围内就会触发onEndReached事件，通常列表元素存在数据加载过程，如果加载的数据量不足以铺满整个整个屏幕，则更新时会立即触发onEndReached事件，因为此时的底部距离是0
    - 滚动中频繁触发：在滚动过程中一般不会频繁触发，仅当检测到items发生变化，一般值长度时才允许触发下一次onEndReached事件，但是为了保证不被误触，通常使用一个控制flag来表示已经在加载下一页数据了，此时不需要再次加载下一页，当下一页数据更新完成后重置此flag
    - 非用户行为触发：例如items数量不足，例如code实现scroll，这些情况一般不想其触发onEndReached事件，因此需要在一些用户交互事件中加一个flag进行相关控制，包括的事件：onScrollEndDrag、onScrollBeginDrag、onMomentumScrollEnd、onMomentumScrollBegin。end事件中flag置为false，start事件中flag置为true。
    - 具体代码如下：
    ```typescript jsx
       import React, {RefObject} from "react";
       import {FlatList, FlatListProps, NativeScrollEvent, NativeSyntheticEvent} from "react-native";
       
       interface IVoVaFlatListProps<ITemT> extends FlatListProps<ITemT> {
           canRefresh?: boolean;
           flatListRef?:RefObject<FlatList<ITemT>>
       }
       
       interface IVoVaFlatListState {
           refreshing: boolean
       }
       
       class VoVaFlatList<ITemT> extends React.PureComponent<IVoVaFlatListProps<ITemT>, IVoVaFlatListState> {
           private canAction: boolean = false;
           private loadingNext: boolean = false;
       
           constructor(props: IVoVaFlatListProps<ITemT>) {
               super(props);
               const {canRefresh = true} = props;
               this.state = {
                   refreshing: canRefresh
               }
           }
       
           private async onRefresh() {
               const {onRefresh} = this.props;
               if (onRefresh) {
                   this.setState({
                       refreshing: true
                   });
                   await onRefresh();
               }
               this.setState({
                   refreshing: false
               })
           }
       
           private async onEndReached(info: { distanceFromEnd: number }) {
               if (!this.canAction) return;
               if (this.loadingNext) return;
               this.loadingNext = true;
               const {onEndReached} = this.props;
               // loading More
               if (onEndReached) {
                   await onEndReached(info);
               }
               this.loadingNext = false;
               console.log("end");
           }
       
           private onScrollEndDrag(event: NativeSyntheticEvent<NativeScrollEvent>) {
               this.canAction = false;
               const {onScrollEndDrag} = this.props;
               onScrollEndDrag && onScrollEndDrag(event);
           }
       
           private onScrollBeginDrag(event: NativeSyntheticEvent<NativeScrollEvent>) {
               this.canAction = true;
               const {onScrollBeginDrag} = this.props;
               onScrollBeginDrag && onScrollBeginDrag(event);
           }
       
           private onMomentumScrollEnd(event: NativeSyntheticEvent<NativeScrollEvent>) {
               this.canAction = false;
               const {onMomentumScrollEnd} = this.props;
               onMomentumScrollEnd && onMomentumScrollEnd(event);
           }
       
           private onMomentumScrollBegin(event: NativeSyntheticEvent<NativeScrollEvent>) {
               this.canAction = true;
               const {onMomentumScrollBegin} = this.props;
               onMomentumScrollBegin && onMomentumScrollBegin(event);
           }
       
           componentDidMount(): void {
               this.onRefresh();
           }
       
           render() {
               const {flatListRef,refreshing,onRefresh,onEndReached,onScrollEndDrag,onMomentumScrollEnd,onScrollBeginDrag,onMomentumScrollBegin,...props} = this.props;
               return (
                   <FlatList ref={flatListRef} {...props} refreshing={this.state.refreshing}
                             onRefresh={this.onRefresh.bind(this)} onEndReached={this.onEndReached.bind(this)}
                             onScrollEndDrag={this.onScrollEndDrag.bind(this)}
                             onMomentumScrollEnd={this.onMomentumScrollEnd.bind(this)}
                             onScrollBeginDrag={this.onScrollBeginDrag.bind(this)}
                             onMomentumScrollBegin={this.onMomentumScrollBegin.bind(this)}/>
               )
           }
       }
       
       export {VoVaFlatList}

    ```
