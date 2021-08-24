# Flux 基础概念与实战入门

![React Flux](./images/react-flux.jpeg "React Flux")

## 前言
随着 React App 复杂度提升，我们会发现常常需要从 Parent Component 通过 props 传递方法到 Child Component 去改变 state tree，不但不方便也难以管理，因此我们需要更好的数据架构来建置更复杂的应用程序。[Flux](https://facebook.github.io/flux/) 是 Facebook 推出的 client-side 应用程序架构（Architecture），主要想解决 `MVC` 架构的一些问题。事实上，Flux 并非一个完整的前端 Framework，其特色在于实现了 Unidirectional Data Flow（单向流）的数据流设计模式，在开发复杂的大型应用程序时可以更容易地管理 state（状态）。由于 React 主要是负责 View 的部份，所以通过搭配 Flux-like 的数据处理架构，可以更好的去管理我们的 state（状态），处理复杂的使用者互动（例如：Facebook 同时要维护使用者是否点赞、点击相片，是否有新消息等状态）。

由于原始的 Flux 架构在实现上有些部分可以精简和改善，在实际操作上我们通常会使用开发者社群开发的 Flux-like 相关的架构实现（例如：[Redux](http://redux.js.org/index.html)、[Alt](http://alt.js.org/)、[Reflux](https://github.com/reflux/refluxjs) 等）。不过这边我们主要会使用 Facebook 本身提供 `Dispatcher API` 函数库（可以想成是一个 pub/sub 处理器，通过 broadcast 将 `payloads` 传给注册的 callback function）并搭配 `NodeJS` 的 `EventEmitter` 模块去完成 Flux 架构的实现。  

## Flux 概念介绍
![React Flux](./images/flux-simple-diagram.png "React Flux")

在 Flux Unidirectional Data Flow（单项流）世界里有四大主角，分别负责不同对应的工作：

1. actions / Action Creator 

  action 负责定义所有改变 state（状态）的行为，可以让开发者快速了解 App 的各种功能，若你想改变 state 你只能发 action。注意 action 可以是同步或是非同步。例如：新增代办事项，调用非同步 API 获取数据。

  实际操作上我们会分成 action 和 Action Creator。action 为描述行为的 object（对象），Action Creator 将 action 送给 dispatcher。一般来说符合 Flux Standard Action 的 action 会如以下范例代码，具备 `type` 来区别所触发的行为。而 `payload` 则是所夹带的数据：

  ```
  // action
  const addTodo = {
    type: 'ADD_TODO',
    payload: {
      text: 'Do something.'  
    }
  }

  AppDispatcher.dispatch(addTodo);
  ```

  当发生 rejected Promise 情况：

  ```
  {
    type: 'ADD_TODO',
    payload: new Error(),
    error: true
  }
  ```

2. Dispatcher

  `Dispatcher` 是 Flux 架构的核心，每个 App 只有一个 Dispatcher，提供 API 让 store 可以注册 `callback function`，并负责向所有 store 发送 action 事件。在本范例中我们使用 Facebook 提供的 Dispatcher API，其内建有 `dispatch` 和 `subscribe` 方法。

3. Stores

  一个 App 通常会有多个 store 负责存放业务逻辑，根据不同业务会有不同 store，例如：TodoStore、RecipeStore。 store 负责操作和储存数据并提供 `view` 使用 `listener`（监听器），若有数据更新即会触发更新。值得注意的是 store 只提供 `getter API` 读取数据，若想改变 state 一律发送 action。

4. Views（Controller Views）

  这部份是 `React` 负责的范畴，负责提供监听事件的 `callback function`，当事件发生时重新取得数据并重绘 `View`。

## Flux 流程回顾

![React Flux](./images/flux-react.png "React Flux")

Flux 架构前置作业：

1. Stores 向 Dispatcher 注册 callback，当数据改变时告知 Stores
2. Controller Views 向 Stores 取得初始数据
3. Controller Views 将数据给 Views 去渲染 UI
4. Controller Views 向 store 注册 listener，当数据改变时告知 Controller Views 

Flux 与使用者互动运作流程：

1. 使用者和 App 互动，触发事件，Action Creator 发送 actions 给 Dispatcher
2. Dispatcher 依序将 action 传给 store 并由 action type 判断合适的处理方式
3. 若有数据更新则会触发 Controller Views 向 store 注册的 listener 并向 store 取得更新数据
4. View 根据 Controller Views 的新数据重新绘制 UI

## Flux 实战初体验
介绍完了整个 Flux 基本架构后，接下来我们就来动手实践一个简单 Flux 架构的 Todo，让使用者可以在 `input` 输入代办事项并新增。

首先，我们先完成一些开发的前置作业，先通过以下指令在根目录产生 npm 配置文件 `package.json`：

```
$ npm init
```

安装相关包（包含开发环境使用的包）：

```
$ npm install --save react react-dom flux events
```

```
$ npm install --save-dev babel-core babel-eslint babel-loader babel-preset-es2015 babel-preset-react eslint eslint-config-airbnb eslint-loader eslint-plugin-import eslint-plugin-jsx-a11y eslint-plugin-react html-webpack-plugin webpack webpack-dev-server
```

安装好后我们可以设计一下我们的文件夹结构，首先我们在根目录建立 `src`，放置 `script` 的 `source` 。在 `components` 文件夹中我们会放置所有 `components`（个别元件文件夹中会用 `index.js` 输出元件，让引入元件更简洁），另外还有 `actions`、`constants`、`dispatcher`、`stores`，其余配置文件则放置于根目录下。

![React Flux 文件夹结构](./images/folder.png "React Flux 文件夹结构")

接下来我们参考上一章设定一下开发文档（`.babelrc`、`.eslintrc`、`webpack.config.js`）。这样我们就完成了开发环境的设定可以开始动手实践 `React Flux` 应用程序了！

HTML Markup：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>TodoFlux</title>
</head>
<body>
    <div id="app"></div>
</body>
</html>
```

以下为 `src/index.js` 完整代码，安排了父 `component` 和在 HTML Markup 插入位置：

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import TodoHeader from './components/TodoHeader';
import TodoList from './components/TodoList';

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {};
  }
  render() {
    return (
      <div>
        <TodoHeader />
        <TodoList />
      </div>
    );
  }
}

ReactDOM.render(<App />, document.getElementById('app'));
```

通常实际操作上我们会开一个 `constants` 文件夹存放 `config` 或是 `actionTypes` 常数。以下是 `src/constants/actionTypes.js`：

```javascript
export const ADD_TODO = 'ADD_TODO';
```

在这个范例中我们继承了 Facebook 提供的 Dispatcher API（主要是继承了 `dispatch`、`register` 和 `subscribe` 的方法），打造自己的 DispatcherClass，当使用者触发 `handleAction()` 会 `dispatch` 出事件。以下是 `src/dispatch/AppDispatcher.js`：

```javascript
// Todo app dispatcher with actions responding to both
// view and server actions
import { Dispatcher } from 'flux';

class DispatcherClass extends Dispatcher {
  handleAction(action) {
    this.dispatch({
      type: action.type,
      payload: action.payload,
    });
  }
}

const AppDispatcher = new DispatcherClass();

export default AppDispatcher;
```

以下是我们利用 `AppDispatcher` 打造的 `Action Creator` 由 `handleAction` 负责发出传入的 `action` ，完整代码如 `src/actions/todoActions.js`：

```javascript
import AppDispatcher from '../dispatcher/AppDispatcher';
import { ADD_TODO } from '../constants/actionTypes';

export const TodoActions = {
  addTodo(text) {
    AppDispatcher.handleAction({
      type: ADD_TODO,
      payload: {
        text,
      },
    });
  },
};
```

`Store` 主要是负责数据以及业务逻辑处理，我们继承了 `events` 模块的 `EventEmitter`，当 `action` 传入 `AppDispatcher.register` 的处理范围后，根据 `action type` 选择适合处理的 `store` 进行处理，处理完后通过 `emit` 方法发出事件让监听的 `Views Controller` 知道。以下是 `src/stores/TodoStore.js`：

```javascript
import AppDispatcher from '../dispatcher/AppDispatcher';
import { ADD_TODO } from '../constants/actionTypes';
import { EventEmitter } from 'events';

const store = {
  todos: [],
  editing: false,
};

class TodoStoreClass extends EventEmitter {
  addChangeListener(callback) {
    this.on(ADD_TODO, callback);
  }
  removeChangeListener(callback) {
    this.removeListener(ADD_TODO, callback);
  }
  getTodos() {
    return store.todos;
  }
}

const TodoStore = new TodoStoreClass();

AppDispatcher.register((action) => {
  switch (action.type) {
    case ADD_TODO:
      store.todos.push(action.payload.text);
      TodoStore.emit(ADD_TODO);
      break;
    default:
      return true;
  }
  return true;
});

export default TodoStore;
```

在这个 React Flux 范例中我们把 `View` 和 `Views Controller` 整合在一起。在 `TodoHeader` 中，我们主要任务是让使用者可以通过 `input` 新增代办事项。使用者输入文字在 `input` 时会触发 `onChange` 事件，进而更新内部的 `state`，当使用者按了送出钮就会触发 `onAdd` 事件，`dispatch` 出 `addTodo event`。以下是 `src/components/TodoHeader.js` 完整范例：

```javascript
import React, { Component } from 'react';
import { TodoActions } from '../../actions/todoActions';

class TodoHeader extends Component {
  constructor(props) {
    super(props);
    this.onChange = this.onChange.bind(this);
    this.onAdd = this.onAdd.bind(this);
    this.state = {
      text: '',
      editing: false,
    };
  }
  onChange(event) {
    this.setState({
      text: event.target.value,
    });
  }
  onAdd() {
    TodoActions.addTodo(this.state.text);
    this.setState({
      text: '',
    });
  }
  render() {
    return (
      <div>
        <h1>TodoFlux</h1>
        <div>
          <input
            value={this.state.text}
            type="text"
            placeholder="请输入代办事项"
            onChange={this.onChange}
          />
          <button
            onClick={this.onAdd}
          >
            送出
          </button>
        </div>
      </div>
    );
  }
}

export default TodoHeader;
```

在上面的 Component 中我们让使用者可以新增代办事项，接下来我们要让新增的代办事项可以显示。我们在 `componentDidMount` 设了一个监听器 `TodoStore` 数据改变时会去把数据重新再更新，这样当使用者新增代办事项时 `TodoList` 就会保持同步。以下是 `src/components/TodoList.js` 完整代码：

```javascript
import React, { Component } from 'react';
import TodoStore from '../../stores/TodoStore';

function getAppState() {
  return {
    todos: TodoStore.getTodos(),
  };
}
class TodoList extends Component {
  constructor(props) {
    super(props);
    this.onChange = this.onChange.bind(this);
    this.state = {
      todos: [],
    };
  }
  componentDidMount() {
    TodoStore.addChangeListener(this.onChange);
  }
  onChange() {
    this.setState(getAppState());
  }
  render() {
    return (
      <div>
        <ul>
          {
            this.state.todos.map((todo, key) => (
              <li key={key}>{todo}</li>
            ))
          }
        </ul>
      </div>
    );
  }
}

export default TodoList;
```

若读者都有跟著上面的步骤走完的话，最后我们在终端机的根目录位置执行 `npm start` 就可以看到整个成果，YA！
![React Flux ](./images/flux-demo.png "React Flux ")

## 总结
Flux 优势：

1. 让开发者可以快速了解整个 App 中的行为
2. 数据和业务逻辑统一存放好管理
3. 让 View 单纯化只负责 UI 的排版不需负责 state 管理
4. 清楚的架构和分工对于复杂中大型应用程序易于维护和管理代码

Flux 劣势：

1. 代码上不够简洁
2. 对于简单小应用来说稍微复杂

以上就是 Flux 的实战入门，我知道一开始接触 Flux 的读者一定会觉得很抽象，有些读者甚至会觉得这个架构到底有什么好处（明明感觉没比 MVC 高明到哪去或是一点都不简洁），但如同上述优点所说 Flux 设计模式的优势在于清楚的架构和分工对于复杂中大型应用程序易于维护和管理代码。若还是不熟悉的读者可以跟著范例多动手，相信慢慢就可以体会 Flux 的特色。事实上，在开发社区中为了让 Flux 架构更加简洁，产生了许多 Flux-like 的架构和函式库，接下来将带读者们进入目前最热门的架构：`Redux`。

## 延伸阅读
1. [Getting To Know Flux, the React.js Architecture](https://scotch.io/tutorials/getting-to-know-flux-the-react-js-architecture)
2. [Flux 官方网站](https://facebook.github.io/flux/)
3. [从 Flux 与 MVC 的差异来简介 Flux](http://blog.techbridge.cc/2016/04/29/introduce-flux-from-flux-and-mvc/)
4. [Flux Stores and ES6](https://medium.com/@softwarecf/flux-stores-and-es6-9b453dbf9db#.uuf1ddj8u)
5. [React and Flux: Migrating to ES6 with Babel and ESLint](https://medium.com/front-end-developers/react-and-flux-migrating-to-es6-with-babel-and-eslint-6390cf4fd878#.vafamphwy)
6. [Building an ES6/JSX/React Flux App – Part 2 – The Flux](https://shellmonger.com/2015/08/17/building-an-es6jsxreact-flux-app-part-2-the-flux/)
7. [Question: How to choose between Redux's store and React's state? #1287](https://github.com/reactjs/redux/issues/1287)
8. [acdlite/flux-standard-action](https://github.com/acdlite/flux-standard-action)

（image via [devjournal](http://devjournal.ru/wp-content/uploads/2016/03/React.js-Flux-Redux.png)、[facebook](https://facebook.github.io/flux/)、[scotch.io](https://cask.scotch.io/2014/10/V70cSEC.png)）

## :door: 任意门
| [回首页](https://github.com/aclk/reactjs101) | [上一章：ImmutableJS 入门教学](https://github.com/aclk/reactjs101/blob/master/Ch06/react-immutable-introduction.md) | [下一章：Redux 基础概念](https://github.com/aclk/reactjs101/blob/master/Ch07/react-redux-introduction.md) |

| [纠错、提问或想法](https://github.com/aclk/reactjs101/issues) |
