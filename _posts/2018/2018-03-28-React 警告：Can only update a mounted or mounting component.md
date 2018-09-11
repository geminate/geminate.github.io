---
layout: post
title: React 警告：Can only update a mounted or mounting component
categories: React
---

## 问题原因

这个警告的意思是 我们对一个没有装载的组件调用了 `setState()` 方法

出现的原因 是由于异步的 `callback` 中调用了 `setState()` 方法，但我们的异步请求返回之前，组件可能就已经被卸载了，例如：

* 在 `componentDidMount()` 中 fetch 数据并在回调中 setState()
* 在 `componentDidMount()` 中 `setTimeout` 或 `setInterval` 并在回调中 setState()


## 解决办法

三种方法任选其一

**设置组件的装载标志位**

{% highlight js %}
    componentWillMount() {
        this.mounted = true;
        fetch("").then((status, response) => {
            if (this.mounted) {
                // Code
            }
        });
    }

    componentWillUnmount() {
        this.mounted = false;
    }
{% endhighlight %}

**在 componentWillUnmount() 中重写 setState 方法**

{% highlight js %}
    componentWillUnmount() {
        this.setState = (state, callback) => {
            return;
        };
    }
{% endhighlight %}

**使用 Redux 管理全部的异步调用。**

