---
layout: post
title: react-router 和 react-redux 搭配使用时，Link点击页面不更新问题
categories: React
no-post-nav: true
---

> [参考文档](https://reacttraining.com/react-router/core/guides/redux-integration/blocked-updates)

今天在react-router 和 react-redux 搭配使用时，遇到了页面不刷新的问题。

问题描述：点击组件中的 Link 后，地址改变但页面内容未更新，手动刷新页面页面内容才更新。

出现条件：

* 组件通过`connet()(Comp)`被连接到 Redux
* 组件不是一个 `route 组件` ，也就是说 组件不是被这样渲染的：`<Route component={SomeConnectedThing}/>`
* 解决办法：使用 withRouter 包裹组件：

```javascript
// before
export default connect(mapStateToProps)(Something)

// after
import { withRouter } from 'react-router-dom'
export default withRouter(connect(mapStateToProps)(Something))
```

出现原因：Redux 的 connect 修改了 组件的shouldComponentUpdate方法，但当组件不包含route传来的属性(props)时，地址的改变也就不会引起组件的更新。

withRouter 可以为组件注入router相关的一些参数，例如history等。