---
title: react-router4 history模式
date: 2019-12-07 17:23:39
categories: 技术
tags: [前端]
---

react-router4的history模式写法:

<!--more-->

```javascript
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom'

//......省略，然后在根组件这样写：
<Router>
    <Switch>
      <Route path="/" exact component={Home} />
      <Route path="/user" exact component={User} />
      <Route path="" component={() => '404'} />
    </Switch>
</Router>
```

如果某个子组件不是`<Route />`的子组件，但仍想使用像`this.props.history.push()`这样的功能，可以导入`withRouter`,用它包裹你的组件
```javascript
import { withRouter } from 'react-router-dom'

function MyChild() {
    return <div>我是一个没有感情的组件</div>
}

export default withRouter(MyChild)
```

注意，如果你像在组件外的代码比如在store里进行一些跳转等，如果你使用hash模式，这样可以生效：
```javascript
import { createHashHistory } from 'history'

const history = createHashHistory()

//进行跳转
history.push('/')
```

但是在history模式里，如果你也这样写：

```javascript
import { createBrowserHistory } from 'history'

const history = createBrowserHistory()

//进行跳转
history.push('/')
```

`history.push`执行后，地址栏的url会改变，但页面并不会做出改变，我猜可能是组件检测不到路由变动。

这时候可以改成下面这样：
新建一个history.js文件：

```javascript
import { createBrowserHistory } from 'history'

const history = createBrowserHistory()

export default history
```

然后路由配置里修改如下：

```javascript
import { Router, Route, Switch } from 'react-router-dom'
import history from './history'

//......省略，然后在根组件这样写：
<Router history={history}>
    <Switch>
      <Route path="/" exact component={Home} />
      <Route path="/user" exact component={User} />
      <Route path="" component={() => '404'} />
    </Switch>
</Router>
```

虽然官网推荐使用BrowserRouter，但其实也可以像react-router3那样导出Router，给它注入history。
这样下面的代码就可以生效了：
```javascript
import { createBrowserHistory } from 'history'

const history = createBrowserHistory()

//进行跳转
history.push('/')
```

再说说部署到服务器时，nginx的配置：

```shell
location / {
  try_files $uri $uri/ /index.html;
}
```
这段的意思是如果 URL 匹配不到任何静态资源，则返回index.html页面，这个页面就是你app依赖的页面。

注意，在history模式里，如果你使用create-react-app，不能在package.json里设置 `homepage: '.'` 即设置静态资源读取路径为相对路径。
如果是hash模式，则无所谓，使用绝对路径或相对路径都可以。

使用react给我感觉就是，你一直在花很多时间搜索解决方案，react在坚持它先进理念的同时，也牺牲了很多便利。相比较而言vue就比较省心，使用起来不用有太多的心智负担。