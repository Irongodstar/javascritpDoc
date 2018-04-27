# dva 文档

## 什么 dva 
是一个基于 `redux`, `Redux-saga` 和 `react-router` 的轻量级前端框架, 它本身没有创造出新的概念出来。 只是将那几个类库中提到的概念包装起来，制定了一套简便的开发模式而已。



## 使用工具创建`dva`项目

### 通过 dva-cli 创建项目

先安装 dva-cli 。

```bash
$ npm install dva-cli -g
```

然后创建项目。

```bash
$ dva new myapp
```

最后，进入目录并启动。

```bash
$ cd myapp
$ npm start
```

## Dva 原理架构图

![](./images/dav.png)

## Reducer

reducer 是一个函数，接受 state 和 action，返回老的或新的 state 。即：`(state, action) => state`

### 增删改

以 todos 为例。

```javascript
app.model({
  namespace: 'todos',
  state: [],
  reducers: {
    add(state, { payload: todo }) {
      return state.concat(todo);
    },
    remove(state, { payload: id }) {
      return state.filter(todo => todo.id !== id);
    },
    update(state, { payload: updatedTodo }) {
      return state.map(todo => {
        if (todo.id === updatedTodo.id) {
          return { ...todo, ...updatedTodo };
        } else {
          return todo;
        }
      });
    },
  },
};
```

### 嵌套数据的增删改

建议最多一层嵌套，以保持 state 的扁平化，深层嵌套会让 reducer 很难写和难以维护。

```javascript
app.model({
  namespace: 'app',
  state: {
    todos: [],
    loading: false,
  },
  reducers: {
    add(state, { payload: todo }) {
      const todos = state.todos.concat(todo);
      return { ...state, todos };
    },
  },
});
```

下面是深层嵌套的例子，应尽量避免。

```javascript
app.model({
  namespace: 'app',
  state: {
    a: {
      b: {
        todos: [],
        loading: false,
      },
    },
  },
  reducers: {
    add(state, { payload: todo }) {
      const todos = state.a.b.todos.concat(todo);
      const b = { ...state.a.b, todos };
      const a = { ...state.a, b };
      return { ...state, a };
    },
  },
});
```

## Effect

示例：

```javascript
app.model({
  namespace: 'todos',
  effects: {
    *addRemote({ payload: todo }, { put, call }) {
      yield call(addTodo, todo);
      yield put({ type: 'add', payload: todo });
    },
  },
});
```

### Effects

#### put

用于触发 action 。

```javascript
yield put({ type: 'todos/add', payload: 'Learn Dva' });
```

#### call

用于调用异步逻辑，支持 promise 。

```javascript
const result = yield call(fetch, '/todos');
```

#### select

用于从 state 里获取数据。

```javascript
const todos = yield select(state => state.todos);
```

### 错误处理

#### 全局错误处理

dva 里，effects 和 subscriptions 的抛错全部会走 `onError` hook，所以可以在 `onError` 里统一处理错误。

```javascript
const app = dva({
  onError(e, dispatch) {
    console.log(e.message);
  },
});
```

然后 effects 里的抛错和 reject 的 promise 就都会被捕获到了。

#### 本地错误处理

如果需要对某些 effects 的错误进行特殊处理，需要在 effect 内部加 `try catch` 。

```javascript
app.model({
  effects: {
    *addRemote() {
      try {
        // Your Code Here
      } catch(e) {
        console.log(e.message);
      }
    },
  },
});
```

### 异步请求

异步请求基于 whatwg-fetch，API 详见：https://github.com/github/fetch

#### GET 和 POST

```javascript
import request from '../util/request';

// GET
request('/api/todos');

// POST
request('/api/todos', {
  method: 'POST',
  body: JSON.stringify({ a: 1 }),
});
```

#### 统一错误处理

假如约定后台返回以下格式时，做统一的错误处理。

```javascript
{
  status: 'error',
  message: '',
}
```

编辑 `utils/request.js`，加入以下中间件：

```javascript
function parseErrorMessage({ data }) {
  const { status, message } = data;
  if (status === 'error') {
    throw new Error(message);
  }
  return { data };
}
```

然后，这类错误就会走到 `onError` hook 里。

## Subscription

`subscriptions` 是订阅，用于订阅一个数据源，然后根据需要 dispatch 相应的 action。数据源可以是当前的时间、服务器的 websocket 连接、keyboard 输入、geolocation 变化、history 路由变化等等。格式为 `({ dispatch, history }) => unsubscribe` 。

### 异步数据初始化

比如：当用户进入 `/users` 页面时，触发 action `users/fetch` 加载用户数据。

```javascript
app.model({
  subscriptions: {
    setup({ dispatch, history }) {
      history.listen(({ pathname }) => {
        if (pathname === '/users') {
          dispatch({
            type: 'users/fetch',
          });
        }
      });
    },
  },
});
```

以上中的 `setup` 只是key, 暂时没有用途(官方: 可能会增加取消的功能, key就会用上了), 可以是任意的字符串(在这里我们尽量使用语义化的字符串表示). 只能传入 `dispatch` (是 `Redux` 的 `store` 中的 `dispatch`) 和 `history` (`react-router` 中的 `history`), 而具体的监听代码是需要具体的方法或是第三方库实现现监听的工作.

注: 如果想在 `subscriptions` 里的 `setup` 中如果要访问 `state` 或者 `props` 的内容可以吗？
官方: 不可以，需要触发 `action` 让 `effect` 去处理。


下面是监听 `keyEvent` 事件参考代码:

```javascript
import key from 'keymaster';
...
app.model({
  namespace: 'count',
  subscriptions: {
    keyEvent(dispatch) {
      key('⌘+up, ctrl+up', () => { dispatch({type:'add'}) });
    },
  }
});
```

下面是监听 websocket 事件有参考代码:

```javascript
// model

import * as service from '../services/socket';

subscriptions: {
    socket({dispatch}){ // socket相关
        return service.listen(data => {
            switch (data.type) {
                case 'connect':
                    if (data.state === 'success') {
                        dispatch({
                            type: 'connectSuccess'
                        })
                    } else {
                        dispatch({
                            type: 'connectFail'
                        })
                    }
                    break;
                case 'welcome':
                    dispatch({
                        type: 'welcome'
                    });
                    break;
            }
        })
    }
},

// services

import io from 'socket.io-client';

let socket = '';

export function listen(action) {
    if (socket === '') {
        try {
            socket = io("localhost:3000");
            action({
                type: 'connect',
                state: 'success'
            });
        } catch (err) {
            action({
                type: 'connect',
                state: 'fail'
            });
        }
    }
    socket.on('welcome', () => {
        action({
            type: 'welcome'
        })
    })
}
```


#### `path-to-regexp` Package

如果 url 规则比较复杂，比如 `/users/:userId/search`，那么匹配和 userId 的获取都会比较麻烦。这是推荐用 [path-to-regexp](https://github.com/pillarjs/path-to-regexp) 简化这部分逻辑。

```javascript
import pathToRegexp from 'path-to-regexp';

// in subscription
const match = pathToRegexp('/users/:userId/search').exec(pathname);
if (match) {
  const userId = match[1];
  // dispatch action with userId
}
```

## Router

### Config with JSX Element (router.js)

```javascript
<Route path="/" component={App}>
  <Route path="accounts" component={Accounts}/>
  <Route path="statements" component={Statements}/>
</Route>
```

详见：[react-router](https://github.com/reactjs/react-router/blob/master/docs/guides/RouteConfiguration.md)

### Route Components

Route Components 是指 `./src/routes/` 目录下的文件，他们是 `./src/router.js` 里匹配的 Component。

#### 通过 connect 绑定数据

比如：

```javascript
import { connect } from 'dva';
function App() {}

function mapStateToProps(state, ownProps) {
  return {
    users: state.users,
  };
}
export default connect(mapStateToProps)(App);
```

然后在 App 里就有了 `dispatch` 和 `users` 两个属性。

#### Injected Props (e.g. location)

Route Component 会有额外的 props 用以获取路由信息。

- location
- params
- children

更多详见：[react-router](https://github.com/reactjs/react-router/blob/master/docs/API.md#injected-props)

### 基于 action 进行页面跳转

```javascript
import { routerRedux } from 'dva/router';

// Inside Effects
yield put(routerRedux.push('/logout'));

// Outside Effects
dispatch(routerRedux.push('/logout'));

// With query
routerRedux.push({
  pathname: '/logout',
  query: {
    page: 2,
  },
});
```

除 `push(location)` 外还有更多方法，详见 [react-router-redux](https://github.com/reactjs/react-router-redux#pushlocation-replacelocation-gonumber-goback-goforward)

## dva 配置

### Redux Middleware

比如要添加 redux-logger 中间件：

```javascript
import createLogger from 'redux-logger';
const app = dva({
  onAction: createLogger(),
});
```

注：onAction 支持数组，可同时传入多个中间件。

### history

#### 切换 history 为 browserHistory

```javascript
import { browserHistory } from 'dva/router';
const app = dva({
  history: browserHistory,
});
```

#### 去除 hashHistory 下的 _k 查询参数

```javascript
import { useRouterHistory } from 'dva/router';
import { createHashHistory } from 'history';
const app = dva({
  history: useRouterHistory(createHashHistory)({ queryKey: false }),
});
```
