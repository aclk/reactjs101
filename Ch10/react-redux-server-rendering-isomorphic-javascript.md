# React Redux Sever Rendering（Isomorphic JavaScript）入门

![React Redux Sever Rendering（Isomorphic）入门](./images/isomorphic-javascript.png "React Redux Sever Rendering（Isomorphic）入门")

## 前言
由于可能有些读者没听过 [Isomorphic JavaScript](http://isomorphic.net/) 。因此在进到开发 React Redux Sever Rendering 应用程序的主题之前我们先来聊聊 Isomorphic JavaScript 这个议题。

根据 [Isomorphic JavaScript](http://isomorphic.net/) 这个网站的说明：

>Isomorphic JavaScript
Isomorphic JavaScript apps are JavaScript applications that can run both client-side and server-side.
The backend and frontend share the same code.

Isomorphic JavaScript 系指浏览器端和服务器端共用 JavaScript 的代码。

另外，除了 Isomorphic JavaScript 外，读者或许也有听过 Universal JavaScript 这个用词。那什么是 Universal JavaScript 呢？它和 Isomorphic JavaScript 是指一样的意思吗？针对这个议题网络上有些开发者提出了自己的观点： [Universal JavaScript](https://medium.com/@mjackson/universal-javascript-4761051b7ae9#.67xsay73m)、[Isomorphism vs Universal JavaScript](https://medium.com/@ghengeveld/isomorphism-vs-universal-javascript-4b47fb481beb#.qvggcp3v8)。其中 Isomorphism vs Universal JavaScript 这篇文章的作者 Gert Hengeveld 指出 `Isomorphic JavaScript` 主要是指前后端共用 JavaScript 的开发方式，而 `Universal JavaScript` 是指 JavaScript 代码可以在不同环境下运行，这当然包含浏览器端和服务器端，甚至其他环境。也就是说 `Universal JavaScript` 在意义上可以涵盖的比 `Isomorphic JavaScript` 更广泛一些，然而在 Github 或是许多技术讨论上通常会把两者视为同一件事情，这部份也请读者留意。

## Isomorphic JavaScript 的好处
在开始真正撰写 Isomorphic JavaScript 前我们在进一步探讨使用 Isomorphic JavaScript 有哪些好处？在谈好处之前，我们先看看最早 Web 开发是如何处理页面渲染和 state 管理，还有遇到哪些挑战。

最早的时候我们谈论 Web 很单纯，都是由 Server 端进行模版的处理，你可以想成 template 是一个函数，我们发送数据进去，template 最后产生一张 HTML 给浏览器显示。例如：Node 使用的（[EJS](http://ejs.co/)、[Jade](http://jade-lang.com/)）、Python/Django 的 [Template](https://docs.djangoproject.com/el/1.10/ref/templates/) 或替代方案 [Jinja](https://github.com/pallets/jinja)、PHP 的 [Smarty](http://www.smarty.net/)、[Laravel](https://laravel.com/) 使用的 [Blade](https://laravel.com/docs/5.0/templates)，甚至是 Ruby on Rails 用的 [ERB](http://guides.rubyonrails.org/layouts_and_rendering.html)。都是由后端去 render 所有数据和页面，前端处理相对单纯。

然而随着前端工程的软件工程化和用户体验的要求，开始出现各式前端框架的百花齐放，例如：[Backbone.js](http://backbonejs.org/)、[Ember.js](http://emberjs.com/) 和 [Angular.js](https://angularjs.org/) 等前端 MVC (Model-View-Controller) 或 MVVM (Model-View-ViewModel) 框架，将页面于前端渲染的不刷页单页式应用程序（Single Page App）也因此开始流行。

后端除了提供初始的 HTML 外，还提供 API Server 让前端框架可以取得数据用于前端 template。复杂的逻辑由 ViewModel/Presenter 来处理，前端 template 只处理简单的是否显示或是元素迭代的状况，如下图所示：

![React Redux Sever Rendering（Isomorphic）入门](./images/client-mvc.png "React Redux Sever Rendering（Isomorphic）入门")

然而前端渲染 template 虽然有它的好处但也遇到一些问题包括性能、SEO 等议题。此时我们就开始思考 Isomorphic JavaScript 的可能性：为什么我们不能前后端都使用 JavaScript 甚至是 React？

![React Redux Sever Rendering（Isomorphic）入门](./images/isomorphic-api.png "React Redux Sever Rendering（Isomorphic）入门")

事实上，React 的优势就在于它可以很优雅地实现 Server Side Rendering 达到 Isomorphic JavaScript 的效果。在 `react-dom/server` 中有两个方法 `renderToString` 和 `renderToStaticMarkup` 可以在 server 端渲染你的 components。其主要都是将 React Component 在 Server 端转成 DOM String，也可以将 props 往下传，然而事件处理会失效，要到 client-side 的 React 接收到后才会把它加上去（但要注意 server-side 和 client-side 的 checksum 要一致不然会出现错误），这样一来可以提高渲染速度和 SEO 效果。`renderToString` 和 `renderToStaticMarkup` 最大的差异在于 `renderToStaticMarkup` 会少加一些 React 内部使用的 DOM 属性，例如：`data-react-id`，因此可以节省一些资源。

使用 `renderToString` 进行 Server 端渲染：

```javascript
import ReactDOMServer from 'react-dom/server';

ReactDOMServer.renderToString(<HelloButton name="Mark" />);
```

渲染出来的效果：

```html
<button data-reactid=".7" data-react-checksum="762752829">
  Hello, Mark
</button>
```

总的来说使用 Isomorphic JavaScript 会有以下的好处：

1. 有助于 SEO
2. Rendering 速度较快，性能较佳
3. 放弃蹩脚的 Template 语法拥抱 Component 组件化思考，便于维护
4. 尽量前后端共用代码节省开发时间

不过要注意的是如果有使用 Redux 在 Server Side Rendering 中，其流程相对复杂，不过大致流程如下：
由后端预先加载需要的 initialState，由于 Server 渲染必须全部都转成 string，所以先将 state 先 dehydration（脱水），等到 client 端再 rehydration（覆水），重建 store 往下传到前端的 React Component。

而要把数据从服务器端传递到客户端，我们需要：

1. 把取得初始 state 当做参数并对每个请求创建一个全新的 Redux store 实体
2. 选择性地 dispatch 一些 action
3. 把 state 从 store 取出来
4. 把 state 一起传到客户端

接下来我们就开始动手实作一个简单的 React Server Side Rendering Counter 应用程序。

## 项目成果截屏

![React Redux Sever Rendering（Isomorphic）入门](./images/react-server-rendering-demo.png "React Redux Sever Rendering（Isomorphic）入门")

## 环境安装与设置
1. 安装 Node 和 NPM

2. 安装所需套件

  ```
  $ npm install --save react react-dom redux react-redux react-router immutable redux-immutable redux-actions redux-thunk babel-polyfill babel-register body-parser express morgan qs
  ```

  ```
  $ npm install --save-dev babel-core babel-eslint babel-loader babel-preset-es2015 babel-preset-react babel-preset-stage-1 eslint eslint-config-airbnb eslint-loader eslint-plugin-import eslint-plugin-jsx-a11y eslint-plugin-react html-webpack-plugin webpack webpack-dev-server redux-logger
  ```

接下来我们先设置一下开发文档。

1. 设置 Babel 的设置档： `.babelrc`

  ```javascript
  {
    "presets": [
      "es2015",
      "react",
    ],
    "plugins": []
  }
  ```

2. 设置 ESLint 的设置档和规则： `.eslintrc`

  ```javascript
  {
    "extends": "airbnb",
    "rules": {
      "react/jsx-filename-extension": [1, { "extensions": [".js", ".jsx"] }],
    },
    "env" :{
      "browser": true,
    }
  }
  ```

3. 设置 Webpack 设置档： `webpack.config.js`

  ```javascript
  // 让你可以动态插入 bundle 好的 .js 档到 .index.html
  const HtmlWebpackPlugin = require('html-webpack-plugin');

  const HTMLWebpackPluginConfig = new HtmlWebpackPlugin({
    template: `${__dirname}/src/index.html`,
    filename: 'index.html',
    inject: 'body',
  });

  // entry 为进入点，output 为进行完 eslint、babel loader 转译后的文件位置
  module.exports = {
    entry: [
      './src/index.js',
    ],
    output: {
      path: `${__dirname}/dist`,
      filename: 'index_bundle.js',
    },
    module: {
      preLoaders: [
        {
          test: /\.jsx$|\.js$/,
          loader: 'eslint-loader',
          include: `${__dirname}/src`,
          exclude: /bundle\.js$/
        }
      ],
      loaders: [{
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        query: {
          presets: ['es2015', 'react'],
        },
      }],
    },
    // 启动开发测试用 server 设置（不能用在 production）
    devServer: {
      inline: true,
      port: 8008,
    },
    plugins: [HTMLWebpackPluginConfig],
  };
  ```

太好了！这样我们就完成了开发环境的设置可以开始动手实践 `React Server Side Rendering Counter` 应用程序了！

先看一下我们整个项目的数据结构，我们把整个项目分成三个主要的文件夹（`client`、`server`，还有共用代码的 `common`）：

![React Redux Sever Rendering（Isomorphic）入门](./images/react-server-rendering-folder.png "React Redux Sever Rendering（Isomorphic）入门")

## 动手实践

首先，我们先定义了 `client` 的 `index.js`：

```javascript
// 引用 babel-polyfill 避免浏览器不支持部分 ES6 用法
import 'babel-polyfill';
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import CounterContainer from '../common/containers/CounterContainer';
import configureStore from '../common/store/configureStore'
import { fromJS } from 'immutable';

// 从 server 取得传进来的 initialState。由于从字串转回对象，又称为 rehydration（覆水）
const initialState = window.__PRELOADED_STATE__;

// 由于我们使用 ImmutableJS，所以需要把在 server-side dehydration（脱水）又在前端 rehydration（覆水）的 initialState 转成 ImmutableJS 数据型态，并传进 configureStore 创建 store
const store = configureStore(fromJS(initialState));

// 接下来就跟一般的 React App 一样，把 store 透过 Provider 往下传到 Component 中
ReactDOM.render(
  <Provider store={store}>
    <CounterContainer />
  </Provider>,
  document.getElementById('app')
);

```

由于 Node 端要到新版对于 ES6 支持较好，所以先用 `babel-register` 在 `src/server/index.js` 去即时转译 `server.js`，但目前不建议在 `production` 环境使用。

```javascript
// use babel-register to precompile ES6 syntax
require('babel-register');
require('./server');
```

接着是我们 `server` 端，也是这个范例最重要的一个部分。首先我们用 `express` 创建了一个 port 为 3000 的 server，并使用 webpack 去运行 `client` 的代码。这个范例中我们使用了 `handleRender` 当 request 进来时（直接拜访页面或刷新）就会运行 fetchCounter() 进行处理：

```javascript
import Express from 'express';
import qs from 'qs';

import webpack from 'webpack';
import webpackDevMiddleware from 'webpack-dev-middleware';
import webpackHotMiddleware from 'webpack-hot-middleware';
import webpackConfig from '../webpack.config';

import React from 'react';
import { renderToString } from 'react-dom/server';
import { Provider } from 'react-redux';
import { fromJS } from 'immutable';

import configureStore from '../common/store/configureStore';
import CounterContainer from '../common/containers/CounterContainer';

import { fetchCounter } from '../common/api/counter';

const app = new Express();
const port = 3000;

function handleRender(req, res) {
  // 模仿实际异步 api 处理情形
  fetchCounter(apiResult => {
  // 读取 api 提供的数据（这边我们 api 是用 setTimeout 进行模仿异步状况），若网址参数有值择取值，若无则使用 api 提供的随机值，若都没有则取 0
    const params = qs.parse(req.query);
    const counter = parseInt(params.counter, 10) || apiResult || 0;
    // 将 initialState 转成 immutable 和符合 state 设计的格式
    const initialState = fromJS({
      counterReducers: {
        count: counter,
      }
    });
    // 创建一个 redux store
    const store = configureStore(initialState);
    // 使用 renderToString 将 component 转为 string
    const html = renderToString(
      <Provider store={store}>
        <CounterContainer />
      </Provider>
    );
    // 从创建的 redux store 中取得 initialState
    const finalState = store.getState();
    // 将 HTML 和 initialState 传到 client-side
    res.send(renderFullPage(html, finalState));
  })
}

// HTML Markup，同时也把 preloadedState 转成字串（stringify）传到 client-side，又称为 dehydration（脱水）
function renderFullPage(html, preloadedState) {
  return `
    <!doctype html>
    <html>
      <head>
        <title>Redux Universal Example</title>
      </head>
      <body>
        <div id="app">${html}</div>
        <script>
          window.__PRELOADED_STATE__ = ${JSON.stringify(preloadedState).replace(/</g, '\\x3c')}
        </script>
        <script src="/static/bundle.js"></script>
      </body>
    </html>
    `
}

// 使用 middleware 于 webpack 去进行 hot module reloading
const compiler = webpack(webpackConfig);
app.use(webpackDevMiddleware(compiler, { noInfo: true, publicPath: webpackConfig.output.publicPath }));
app.use(webpackHotMiddleware(compiler));
// 每次 server 接到 request 都会调用 handleRender
app.use(handleRender);

// 监听 server 状况
app.listen(port, (error) => {
  if (error) {
    console.error(error)
  } else {
    console.info(`==> 🌎  Listening on port ${port}. Open up http://localhost:${port}/ in your browser.`)
  }
});
```

处理完 Server 的部份接下来我们来处理 actions 的部份，在这个范例中 actions 相对简单，主要就是添加和减少两个行为，以下为 `src/actions/counterActions.js`：

```javascript
import { createAction } from 'redux-actions';
import {
  INCREMENT_COUNT,
  DECREMENT_COUNT,
} from '../constants/actionTypes';

export const incrementCount = createAction(INCREMENT_COUNT);
export const decrementCount = createAction(DECREMENT_COUNT);
```

以下为输出常数 `src/constants/actionTypes.js`：

```javascript
export const INCREMENT_COUNT = 'INCREMENT_COUNT';
export const DECREMENT_COUNT = 'DECREMENT_COUNT';
```

在这个范例中我们使用 `setTimeout()` 来仿真异步的产生数据让 server 端在每次接收 request 时读取随机产生的值。事实上，我们会开 API 让 Server 读取初始要导入的 initialState。

```javascript
function getRandomInt(min, max) {
  return Math.floor(Math.random() * (max - min)) + min
}

export function fetchCounter(callback) {
  setTimeout(() => {
    callback(getRandomInt(1, 100))
  }, 500)
}
```

谈完 actions 我们来看我们的 reducers，在这个范例中 reducers 也是相对简单的，主要就是针对添加和减少两个行为去 set 值，以下是 `src/reducers/counterReducers.js`：

```javascript
import { fromJS } from 'immutable';
import { handleActions } from 'redux-actions';
import { CounterState } from '../constants/models';

import {
  INCREMENT_COUNT,
  DECREMENT_COUNT,
} from '../constants/actionTypes';

const counterReducers = handleActions({
  INCREMENT_COUNT: (state) => (
    state.set(
      'count',
      state.get('count') + 1
    )
  ),
  DECREMENT_COUNT: (state) => (
    state.set(
      'count',
      state.get('count') - 1
    )
  ),
}, CounterState);

export default counterReducers;
```

准备好了 `rootReducer` 就可以使用 `createStore` 来创建我们 store，值得注意的是由于 `configureStore` 需要被 client-side 和 server-side 使用，所以把它输出成 function 方便传入 initialState 使用。以下是 `src/store/configureStore.js`：

```javascript
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import createLogger from 'redux-logger';
import rootReducer from '../reducers';

export default function configureStore(preloadedState) {
  const store = createStore(
    rootReducer,
    preloadedState,
    applyMiddleware(createLogger({ stateTransformer: state => state.toJS() }), thunk)
  )
  return store
}
```

最后来到了 `components` 和 `containers` 的时间，这次我们的 Component 主要有两个按钮让用户可以添加和减少数字并显示目前数字。以下是 `src/components/Counter/Counter.js`：

```javascript
import React, { Component, PropTypes } from 'react'

const Counter = ({
  count,
  onIncrement,
  onDecrement,
}) => (
  <p>
    Clicked: {count} times
    {' '}
    <button onClick={onIncrement}>
      +
    </button>
    {' '}
    <button onClick={onDecrement}>
      -
    </button>
    {' '}
  </p>
);

// 注意要检查 propTypes 和给定默认值
Counter.propTypes = {
  count: PropTypes.number.isRequired,
  onIncrement: PropTypes.func.isRequired,
  onDecrement: PropTypes.func.isRequired
}

Counter.defaultProps = {
  count: 0,
  onIncrement: () => {},
  onDecrement: () => {}
}

export default Counter;
```

最后把取出的 `count ` 和事件处理方法用 connect 传到 `Counter` 就大功告成了！以下是 `src/containers/CounterContainer/CounterContainer.js`：

```javascript
import 'babel-polyfill';
import { connect } from 'react-redux';
import Counter from '../../components/Counter';

import {
  incrementCount,
  decrementCount,
} from '../../actions';

export default connect(
  (state) => ({
    count: state.get('counterReducers').get('count'),
  }),
  (dispatch) => ({
    onIncrement: () => (
      dispatch(incrementCount())
    ),
    onDecrement: () => (
      dispatch(decrementCount())
    ),
  })
)(Counter);
```

若一切顺利，在终端机打上 `$ npm start`，你将可以在浏览器的 `http://localhost:3000` 看到自己的成果！

![React Redux Sever Rendering（Isomorphic）入门](./images/react-server-rendering-demo.png "React Redux Sever Rendering（Isomorphic）入门")

## 总结
本章阐述了 Web 页面浏览的进程和 Isomorphic JavaScript 的优势，并介绍了如何使用 React Redux 进行 Server Side Rendering 的应用编程。下一个章节我们将集成后端数据库，运用 React + Redux + Node（Isomorphic）开发一个简单的食谱分享网站。

## 延伸阅读
1. [DavidWells/isomorphic-react-example](https://github.com/DavidWells/isomorphic-react-example)
2. [RickWong/react-isomorphic-starterkit](https://github.com/RickWong/react-isomorphic-starterkit)
3. [Server-rendered React components in Rails](https://www.bensmithett.com/server-rendered-react-components-in-rails/)
4. [Our First Node.js App: Backbone on the Client and Server](http://nerds.airbnb.com/weve-launched-our-first-nodejs-app-to-product/)
5. [Going Isomorphic with React](https://bensmithett.github.io/going-isomorphic-with-react/#/)
6. [A service for server-side rendering your JavaScript views](https://github.com/airbnb/hypernova)
7. [Isomorphic JavaScript: The Future of Web Apps](http://nerds.airbnb.com/isomorphic-javascript-future-web-apps/)
8. [React Router Server Rendering](https://github.com/reactjs/react-router-tutorial/tree/master/lessons/13-server-rendering)

（image via [airbnb](http://nerds.airbnb.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-06-at-5.21.00-PM.png)）

## :door: 任意门
| [回首页](https://github.com/aclk/reactjs101) | [上一章：用 React + Router + Redux + ImmutableJS 写一个 Github 查找应用](https://github.com/aclk/reactjs101/blob/master/Ch09/react-router-redux-github-finder.md) | [下一章：用 React + Redux + Node（Isomorphic JavaScript）开发食谱分享网站](https://github.com/aclk/reactjs101/blob/master/Ch10/react-router-redux-node-isomorphic-javascript-open-cook.md) |

| [纠错、提问或想法](https://github.com/aclk/reactjs101/issues) |
