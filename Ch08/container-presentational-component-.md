# Container 与 Presentational Components 入门

## 前言
在聊完了 React 和 Redux 整合后我们来谈谈分离 Presentational 和 Container Component 的概念，若你是第一次听过这个名词，我建议你可以先看看 Redux 作者 Dan AbramovFollow 所写的这篇文章 [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.vtcuxsurv)。

## Container 与 Presentational Components 超级比一比
以下先参考 [Redux 官网](http://redux.js.org/docs/basics/UsageWithReact.html) 列出两者相异之处：

1. Presentational Components  
  - 用途：怎么看事情（Markup、外观）
  - 是否让 Redux 意识到：否
  - 取得数据方式：从 props 取得
  - 改变数据方式：从 props 去呼叫 callback function
  - 写入方式：手动处理

2. Container Components
 - 用途：怎么做事情（采集数据，更新 State）
 - 是否让 Redux 意识到：是
 - 取得数据方式：订阅 Redux State（store）
 - 改变数据方式：Dispatch Redux Action
 - 写入方式：从 React Redux 产生

 从上面的分析读者可以发现，两者最大的差别在于 `Component` 主要负责单纯的 UI 的渲染，而 `Container` 则负责和 Redux 的 store 沟通，作为 `Redux` 和 `Component` 之间的桥梁。这样的分法可以让程序架构和职责更清楚，所以接下来我们就使用上一章节的 Redux TodoApp 进行改造，改造成 Container 与 Presentational Components 模式。

## Container Components

以下是 `src/containers/TodoHeaderContainer/TodoHeaderContainer.js` 的部份：

```javascript
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
```

以下是 `src/containers/TodoListContainer/TodoListContainer.js` 的部份：

```javascript
import { connect } from 'react-redux';
import TodoList from '../../components/TodoList';

import {
  deleteTodo,
} from '../../actions';

const mapStateToProps = (state) => ({
  todos: state.getIn(['todo', 'todos'])
});

const mapDispatchToProps = (dispatch) => ({
  onDeleteTodo: (index) => () => (
    dispatch(deleteTodo({ index }))
  )
});

export default connect(
  mapStateToProps,
  mapDispatchToProps,
)(TodoList);
```

## Presentational Components

以下是 `src/components/TodoHeader/TodoHeader.js` 的部份：

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

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

// Component 部分值的注意的是 todos state 是通过 map function 去迭代出元素，由于要让 React JSX 可以渲染并保持传入触发 event state 的 immutable，所以需使用 toJS() 转换 component of array。
// 由 Component 传进欲删除元素的 index

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

## 总结
That's it！通过区分 Container 与 Presentational Components 可以让程序架构和职责更清楚了！接下来我们将运用我们所学实际开发两个贴近生活的专案，让读者更加熟悉 React 生态系如何应用于实务上。

## 延伸阅读
1. [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.vtcuxsurv)
2. [Redux Usage with React](http://redux.js.org/docs/basics/UsageWithReact.html)
3. [React Higher Order Components in depth](https://medium.com/@franleplant/react-higher-order-components-in-depth-cf9032ee6c3e#.r8srulpaj)
4. [React higher order components](http://www.darul.io/post/2016-01-05_react-higher-order-components)

## :door: 任意门
| [回首页](https://github.com/aclk/reactjs101) | [上一章：Redux 实战入门](https://github.com/aclk/reactjs101/blob/master/Ch07/react-redux-real-world-example.md) | [下一章：用 React + Router + Redux + ImmutableJS 写一个 Github 查询应用](https://github.com/aclk/reactjs101/blob/master/Ch09/react-router-redux-github-finder.md) |

| [纠错、提问或想法](https://github.com/aclk/reactjs101/issues) |
