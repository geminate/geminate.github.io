---
layout: post
title: redux，react-redux 使用笔记
categories: React
---

## 一. connect()

react-redux 提供了 `connect` 方法，用于将一个UI组件包装成一个容器组件。使用方法如下：

{% highlight js %}
import React, {Component} from 'react';
import {connect} from 'react-redux';

class List extends Component {

}

export default connect(mapStateToProps, mapDispatchToProps)(List);
{% endhighlight %}

其中 connect 方法接受两个参数 `mapStateToProps` 和 `mapDispatchToProps`，前者负责输入逻辑，将 外部(store)中的状态state映射为当前组件的属性props，后者负责输出逻辑，将用户的各种操作映射为Action输出出去。

## 二. mapStateToProps(state)

connect中的这个参数负责将外部的状态映射为当前组件的属性。方法返回一个对象，这个对象即向组件中添加的映射属性。

{% highlight js %}
const mapStateToProps = (state) => ({
    listLength: state.listLength
});
{% endhighlight %}

上例中向该组件中添加了一个listLength属性，该属性的值从外部的state中获取。

mapStateToProps方法作为connect 方法的第一个参数可以被省略，省略后表示组件属性不会订阅外部的状态。

## 三. mapDispatchToProps()

connect的这个参数是一个返回对象的函数，返回的对象中定义了props 和 dispatch(Action)的对应关系。

{% highlight js %}
const mapDispatchToProps = (dispatch) => {
    return {
        onClick: () => {
            dispatch({type: 'ON_CLICK', payload: {}});
        }
    }
};
{% endhighlight %}

上例中在该组件中添加了一个onClick属性，该属性被调用时会发送一个名为‘ON_CLICK’的 Action。

mapDispatchToProps 也可以直接是一个对象，上例中代码也可以写成这样：

{% highlight js %}
const mapDispatchToProps = {
    onClick: () => ({type: 'ON_CLICK', payload: {}})
};
{% endhighlight %}

写成对象时，对象的key就是该组件新增的属性，对应的value是一个返回Action对象的函数，即Action Creator。

## 四. Provider
react-redux 提供了 `<Provider />` 组件方便为所有的子组件拿到state。store是state的存储器，作为参数被传入到Provider组件。

{% highlight js %}
import {Provider} from 'react-redux';

render(){
    return (
        <Provider store={store}>
            <App/>
        </Provider>
    )
}
{% endhighlight %}

Provider 组件一般应在整个应用的最外层，嵌套之后，其下的全部组件均可获取store中的state数据。

## 五. createStroe

redux 提供 `createStroe` 方法，该方法有三个参数，第一个参数定于reducer，第二个参数定义store的初始状态，最后一个参数引入中间件。

{% highlight js %}
import {createStore, applyMiddleware} from 'redux';
import thunk from 'redux-thunk';

const store = createStore(reducer, initalState, applyMiddleware(thunk));
{% endhighlight %}

初始状态就是一个形如下例的对象：

{% highlight js %}
const initalState = {
    listLength: 0
};
{% endhighlight %}

## 六. reducer

Reducer 是一个函数，它接受 Action 和当前 State 作为参数，返回一个新的 State。

{% highlight js %}
const reducer = (state = {}, action) => {
    switch (action.type) {
        case 'ON_CLICK': {
            return {...state, ...action.payload}
        }
        default: {
            return {...state};
        }
    }
};
{% endhighlight %}

上例中的reducer判断当Action为‘ON_CLICK’时，会将action中的 payload 合并到state中去。

## 七. combineReducers

当我们的应用比较庞大的时候，所有的reducer聚集到一起会非常多，这时候我们可以通过 `combineReducers` 对reducer按模块进行划分。

{% highlight js %}
import {combineReducers} from 'redux';

const initalState = {
    layout: {
        siderOpen: true
    },
    data: {
        menuList: [],
        appLoading: false
    }
};

const reducer = combineReducers({
    layout: (state = {}, action) => { // 这里的state 实际上是 state.layout
        switch (action.type) {
            case 'ON_CLICK': {
                return {...state, ...action.payload}
            }
            default: {
                return {...state};
            }
        }
    },
    data: (state = {}, action) => { // 这里的state 实际上是 state.data
        switch (action.type) {
            case 'DATA_CLICK': {
                return {...state, ...action.payload}
            }
            default: {
                return {...state};
            }
        }
    }
});
{% endhighlight %}

如上例，combineReducers 由 redux 提供，参数为多个 Reducer 组成的一个对象，对象的 key 为分类名称，value 为对应的 reducer。需要注意的是，在使用了 combineReducers 后，store 中 state 的结构需要与 combineReducers 中的分类相对应，即在 layout 分类下的 reducer 操作的 state 实际上是 store 中的 state.layout。

## 八. 异步操作 和 applyMiddleware

action 发起后 Reducer 立即算出 State 这中间没有供我们进行异步操作的余地，因为我们没有办法在 Reducer 在异步操作结束后自动执行其他操作。此时我们需要引入 中间件，如 redux-trunk。redux-trunk在被引入后，dispatch()的参数 由原来的 只能是一个形如{type：‘XX’，payload：{}}这样的对象,变成了也可以为一个函数。

{% highlight js %}
const mapDispatchToProps = (dispatch) => {
    return {
        fetchMenu: () => {
            dispatch(function (dispatch) {
                dispatch({XXXX：XXX});
                axios.get("/api/v1/menu", {dataType: 'json'}).then(res => {
                    dispatch({XXXX：XXX});
                });
            })
        }

    }
};
{% endhighlight %}

上例中原来的 dispatch(Action) 变为了 dispatch(function)，在function里面先发出了一个action，然后进行异步操作后再次发出一个Action，从而实现了效果。

上例改写为 Action Creater的形式如下：

{% highlight js %}
const mapDispatchToProps = {
    fetchMenu: () => {
        return (dispatch) => {
            dispatch({type: 'FETCH_START'});
            axios.get("/api/v1/menu", {dataType: 'json'}).then(res => {
                dispatch({type: 'FETCH_SUCCESS', payload: res.data});
            });
        }
    }
}
{% endhighlight %}

Action creator 不在返回一个 Action对象 ，而是一个方法。