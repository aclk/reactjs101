# Redux 实战入门

## 前言
上一节我们了解了 Redux 基本的概念和特性后，本章我们要实际动手用 Redux、React Redux 结合 ImmutableJS 开发一个简单的 Todo 应用。话不多说，那就让让我们开始吧！

以下这张图表示了整个 React Redux App 的数据流程图（使用者与 View 互动 => dispatch 出 Action => Reducers 依据 action type 分配到对应处理方式，回传新的 state => 通过 React Redux 传送给 React，React 重新绘制 View）：

![React Redux](./images/redux-flow.png "React Redux")

## 动手创作 React Redux ImmutableJS TodoApp
在开始创作之前我们先完成一些开发的前置作业，先通过以下指令在根目录产生 npm 配置文件 `package.json`：

```
$ npm init
```

安装相关包（包含开发环境使用的包）：

```
$ npm install --save react react-dom redux react-redux immutable redux-actions redux-immutable
```

```
$ npm install --save-dev babel-core babel-eslint babel-loader babel-preset-es2015 babel-preset-react eslint eslint-config-airbnb eslint-loader eslint-plugin-import eslint-plugin-jsx-a11y eslint-plugin-react html-webpack-plugin webpack webpack-dev-server
```

安装好后我们可以设计一下我们的文件夹目录结构，首先我们在根目录建立 `src`，放置 `script` 的 `source` 。在 `components` 文件夹目录中我们会放置所有 `components`（个别元件文件夹目录中会用 `index.js` 输出元件，让引入元件更简洁）、`containers`（负责和 store 互动取得 state），另外还有 `actions`、`constants`、`reducers`、`store`，其余配置文件则放置于根目录下。

大致上的文件夹目录结构会长这样：

![React Redux](./images/redux-folder.png "React Redux")

接下来我们参考上一章设定一下开发文档（`.babelrc`、`.eslintrc`、`webpack.config.js`）。这样我们就完成了开发环境的设定可以开始动手实践 `React Redux` 应用程式了！

首先我们先用 Component 之眼感受一下我们应用程式，将它切成一个个 `Component`。在这边我们设计一个主要的 `Main` 包含两个子 Component：`TodoHeader`、`TodoList`。

![React Redux](./images/react-redux-demo.png "React Redux")

首先设计 HTML Markup：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Redux Todo</title>
</head>
<body>
  <div id="app"></div>
</body>
</html>
```

在编写 `src/index.js` 之前，我们先说明整合 `react-redux` 的用法。从以下这张图可以看到 `react-redux` 是 React 和 Redux 间的桥梁，使用 `Provider`、`connect` 去连结 `store` 和 React View。

![React Redux](./images/using-redux.jpg "React Redux")

事实上，整合了 `react-redux` 后，我们的 React App 就可以解决传统跨 Component 之前传递 state 的问题和困难。只要通过 `Provider` 就可以让每个 React App 中的 `Component` 取用 store 中的 state，非常方便（接下来我们也会更详细说明 Container/Component、`connect` 的用法）。

![React Redux](./images/redux-store.png "React Redux")

以下是 `src/index.js` 完整代码： 

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import Main from './components/Main';
import store from './store';

ReactDOM.render(
  <Provider store={store}>
    <Main />
  </Provider>,
  document.getElementById('app')
);
```

其中 `src/components/Main/Main.js` 是 Stateless Component，负责所有 View 的进入点。

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import TodoHeaderContainer from '../../containers/TodoHeaderContainer';
import TodoListContainer from '../../containers/TodoListContainer';

const Main = () => (
  <div>
    <TodoHeaderContainer />
    <TodoListContainer />
  </div>
);

export default Main;
```

接下来我们定义一下 `Actions` 的部份，由于是范例 App 所以相对简单，这边只定义一个 todoActions。在这边我们使用了 [redux-actions](https://github.com/acdlite/redux-actions)，它可以方便我们使用 Flux Standard Action 格式的 action。以下是 `src/actions/todoActions.js` 完整代码：

```javascript
import { createAction } from 'redux-actions';
import {
  CREATE_TODO,
  DELETE_TODO,
  CHANGE_TEXT,
} from '../constants/actionTypes';

export const createTodo = createAction('CREATE_TODO');
export const deleteTodo = createAction('DELETE_TODO');
export const changeText = createAction('CHANGE_TEXT');
```

我们在 `src/actions/index.js` 将所有 actions 输出

```javascript
export * from './todoActions';
```

另外我们把 constants 放到 `components` 文件夹目录中方便管理，以下是 `src/constants/actionTypes.js` 代码：

```javascript
export const CREATE_TODO = 'CREATE_TODO';
export const DELETE_TODO = 'DELETE_TODO';
export const CHANGE_TEXT = 'CHANGE_TEXT';

/* 
或是可以考虑使用 keyMirror，方便产生与 key 相同的常数
import keyMirror from 'fbjs/lib/keyMirror';

export default keyMirror({
    ADD_ITEM: null,
    DELETE_ITEM: null,
    DELETE_ALL: null,
    FILTER_ITEM: null
});
*/
```

设定 Actions 后我们来讨论一下 Reducers 的部份。在讨论 Reducers 之前我们先来设定一下我们的前端的数据结构，在这边我们把所有数据结构（initialState）放到 `src/constants/models.js` 中。这边特别注意的是由于 Redux 中有一个重要特性是 `State is read-only`，也就是说更新当 reducers 进到 action 只会回传新的 state 不会更改到原有的 state。因此我们会在整个 Redux App 中使用 `ImmutableJS` 让整个数据流维持在 `Immutable` 的状态，也可以提升程式开发上的性能和避免不可预期的副作用。

以下是 `src/constants/models.js` 完整代码，其设定了 TodoState 的数据结构并使用 `fromJS()` 转成 `Immutable`：

```javascript
import Immutable from 'immutable';

export const TodoState = Immutable.fromJS({
  'todos': [],
  'todo': {
    id: '',
    text: '',
    updatedAt: '',
    completed: false,
  }
});
```

接下来我们要讨论的是 Reducers 的部份，在 `todoReducers` 中我们会根据接收到的 action 进行 mapping 到对应的处理函数并传入夹带的 `payload` 数据（这边我们使用 [redux-actions](https://github.com/acdlite/redux-actions) 来进行 mapping，使用上比传统的 switch 更为简洁）。Reducers 接收到 action 的处理方式为 `(initialState, action) => newState`，最终会回传一个新的 state，而非更改原来的 state，所以这边我们使用 `ImmutableJS`。

```javascript
import { handleActions } from 'redux-actions';
import { TodoState } from '../../constants/models';

import {
  CREATE_TODO,
  DELETE_TODO,
  CHANGE_TEXT,
} from '../../constants/actionTypes';

 const todoReducers = handleActions({
  CREATE_TODO: (state) => {
    let todos = state.get('todos').push(state.get('todo'));
    return state.set('todos', todos)
  },
  DELETE_TODO: (state, { payload }) => (
    state.set('todos', state.get('todos').splice(payload.index, 1))
  ),
  CHANGE_TEXT: (state, { payload }) => (
    state.merge({ 'todo': payload })
  )
}, TodoState);

export default todoReducers;
```

```javascript
import { handleActions } from 'redux-actions';
import UiState from '../../constants/models';

export default handleActions({
  SHOW: (state, { payload }) => (
    state.set('todos', payload.todo)
  ),
}, UiState); 
```

虽然 Redux 本身仅会有一个 store，但 redux 本身有提供了 `combineReducers` 可以让我们切割我们 state 方便维护和管理。实上，state 的规划也是一们学问，通常需要不断地实践和工作团队讨论才能找到比较好的方式。不过这边要注意的是我们改使用了 `redux-immutable` 的 `combineReducers` 这样可以确保我们的 state 维持在 `Immutable` 的状态。    

由于 Redux 官方也没有特别明确或严谨的规范。在一般情况我会将 reducers 分为 `data` 和单纯和 UI 有关的 `ui` state。但由于这边是比较简单的例子，我们最终只使用到 `src/reducers/data/todoReducers.js`。 

```javascript
import { combineReducers } from 'redux-immutable';
import ui from './ui/uiReducers';// import routes from './routes';
import todo from './data/todoReducers';// import routes from './routes';

const rootReducer = combineReducers({
  todo,
});

export default rootReducer;
```

还记得我们上面说明 React Redux 之前的桥梁时有提到的 store 吗？现在我们要更仔细地去设计 `store`，我们这边使用到了 redux 其中两个 API：applyMiddleware、createStore。分别可以产生 store 和挂载我们要使用的 middleware（这边我们只使用到 redux-logger 方便我们除错）。注意我们 initialState 也是维持在 `Immutable` 的状态。  

```javascript
import { createStore, applyMiddleware } from 'redux';
import createLogger from 'redux-logger';
import Immutable from 'immutable';
import rootReducer from '../reducers';

const initialState = Immutable.Map();

export default createStore(
  rootReducer,
  initialState,
  applyMiddleware(createLogger({ stateTransformer: state => state.toJS() }))
);
```

通过 `src/store/index.js` 输出 configureStore：

```javascript
export { default } from './configureStore';
```

讲解完架构层面的议题，终于我们来到了 View 的部份。加油，距离我们终点也不远了！
在开始讨论 `Component` 的部份之前我们先来研究一下 

[react-redux](https://github.com/reactjs/react-redux) 所提供的 API `connect` 将 props 传给 Component，其用法如下：

`connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])` 

在我们的范例 App 中我们只会先用到前两个参数，第三个参数会在之后的例子里用到。第一个参数 mapStateToProps 是一个让开发者可以从 store 取出想要 state 并当做 props 往下传的功能，第二个参数则是将 dispatch 行为封装成函数顺著 props 可以方便往下传和调用。

以下是 `src/components/TodoHeader/TodoHeader.js` 的部份：

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import { connect } from 'react-redux';
import TodoHeader from '../../components/TodoHeader';

// 将欲使用的 actions 引入
import {
  changeText,
  createTodo,
} from '../../actions';

const mapStateToProps = (state) => ({
  // 从 store 取得 todo state
  todo: state.getIn(['todo', 'todo'])
});

const mapDispatchToProps = (dispatch) => ({
  // 当使用者在 input 输入数据值即会触发这个函数，发出 changeText action 并附上使用者输入内容 event.target.value
  onChangeText: (event) => (
    dispatch(changeText({ text: event.target.value }))
  ),
  // 当使用者按下送出时，发出 createTodo action 并清空 input 
  onCreateTodo: () => {
    dispatch(createTodo());
    dispatch(changeText({ text: '' }));
  }
});

export default connect(
  mapStateToProps,
  mapDispatchToProps,
)(TodoHeader);

// 开始建设 Component 并使用 connect 进来的 props 并绑定事件（onChange、onClick）。注意我们的 state 因为是使用 `ImmutableJS` 所以要用 `get()` 取值
const TodoHeader = ({
  onChangeText,
  onCreateTodo,
  todo,
}) => (
  <div>
    <h1>TodoHeader</h1>
    <input type="text" value={todo.get('text')} onChange={onChangeText} />
    <button onClick={onCreateTodo}>送出</button>
  </div>
);

export default TodoHeader;
```

以下是 `src/components/TodoList/TodoList.js` 的部份：

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import { connect } from 'react-redux';
import TodoList from '../../components/TodoList';

import {
  deleteTodo,
} from '../../actions';

const mapStateToProps = (state) => ({
  todos: state.getIn(['todo', 'todos'])
});

// 由 Component 传进欲删除元素的 index
const mapDispatchToProps = (dispatch) => ({
  onDeleteTodo: (index) => () => (
    dispatch(deleteTodo({ index }))
  )
});

export default connect(
  mapStateToProps,
  mapDispatchToProps,
)(TodoList);

// Component 部分值的注意的是 todos state 是通过 map function 去迭代出元素，由于要让 React JSX 可以渲染并保持传入触发 event state 的 immutable，所以需使用 toJS() 转换 component of array。
const TodoList = ({
  todos,
  onDeleteTodo,
}) => (
  <div>
    <ul>
    {
      todos.map((todo, index) => (
        <li key={index}>
          {todo.get('text')}
          <button onClick={onDeleteTodo(index)}>X</button>
        </li>
      )).toJS()
    }
    </ul>
  </div>
);

export default TodoList;
```

若是一切顺利的话就可以在浏览器上看到自己努力的成果！（因为我们有使用 `redux-logger` 所以打开 console 会看到 action 和 state 的变化情形，但记得在 `production` 环境要拿掉）

![React Redux](./images/react-redux-dev-demo.png "React Redux")

## 总结
以上就是 Redux 实战入门，对于第一次自己动手写 Redux 的朋友可能会需要多练习几次，多体会整个架构。在接下来的章节我们将优化我们的 React Redux TodoApp，让它可以有更清晰好维护的架构。

## 延伸阅读
1. [Redux 官方网站](http://redux.js.org/index.html)

（image via [JonasOhlsson](http://www.slideshare.net/JonasOhlsson/using-redux)、[licdn](https://media.licdn.com/mpr/mpr/shrinknp_800_800/AAEAAQAAAAAAAAUQAAAAJDAyMWU1MmZhLTYzMTQtNDJkNy1hYzM4LTE5MWQzNWM1ODcyNA.png)）

## :door: 任意门
| [回首页](https://github.com/aclk/reactjs101) | [上一章：Redux 基础概念](https://github.com/aclk/reactjs101/blob/master/Ch07/react-redux-real-world-example.md) | [下一章：Container 与 Presentational Components 入门](https://github.com/aclk/reactjs101/blob/master/Ch08/container-presentational-component-.md) |

| [纠错、提问或想法](https://github.com/aclk/reactjs101/issues) |
