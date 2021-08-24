# 用 React + Router + Redux + ImmutableJS 写一个 Github 查找应用

## 前言

学了一身本领后，本章将带大家完成一个单页式应用程序（Single Page Application），集成 React + Redux + ImmutableJS + React Router 搭配 Github API 制作一个简单的 Github 用户查找应用，实际体验一下开发 React App 的感受。

## 功能规划

让访客可以使用 Github ID 搜索 Github 用户，展示 Github 用户名、follower、following、avatar_url 并可以返回首页。

## 使用技术

1. React
2. Redux
3. Redux Thunk
4. React Router
5. ImmutableJS
6. Fetch
7. [Material UI](http://www.material-ui.com/#/)
8. Roboto Font from Google Font
9. Github API（<https://api.github.com/users/torvalds>）

不过要注意的是 Github API 若没有使用 App key 的话可以调用 API 的次数会受限

## 项目成果截屏

![React Redux](./images/demo-1.png "React Redux")

![React Redux](./images/demo-2.png "React Redux")

## 环境安装与设置

1. 安装 Node 和 NPM

2. 安装所需包

`
$ npm install --save react react-dom redux react-redux react-router immutable redux-immutable redux-actions whatwg-fetch redux-thunk material-ui react-tap-event-plugin
`

`
$ npm install --save-dev babel-core babel-eslint babel-loader babel-preset-es2015 babel-preset-react babel-preset-stage-1 eslint eslint-config-airbnb eslint-loader eslint-plugin-import eslint-plugin-jsx-a11y eslint-plugin-react html-webpack-plugin webpack webpack-dev-server redux-logger
`

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

太好了！这样我们就完成了开发环境的设置可以开始动手实践 `Github Finder` 应用程序了！

## 动手实操

1. Setup Mockup

    HTML Markup（`src/index.html`）：

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
        <title>GithubFinder</title>
        <link href="https://fonts.googleapis.com/css?family=Roboto:300,400,500" rel="stylesheet">
    </head>
    <body>
        <div id="app"></div>
    </body>
    </html>
    ```

    设置 `webpack.config.js` 的进入点 `src/index.js`：

    ```javascript
    import React from 'react';
    import ReactDOM from 'react-dom';
    import { Provider } from 'react-redux';
    import { browserHistory, Router, Route, IndexRoute } from 'react-router';
    import injectTapEventPlugin from 'react-tap-event-plugin';
    import MuiThemeProvider from 'material-ui/styles/MuiThemeProvider';
    import Main from './components/Main';
    import HomePageContainer from './containers/HomePageContainer';
    import ResultPageContainer from './containers/ResultPageContainer';
    import store from './store';

    // 引入 react-tap-event-plugin 避免 material-ui onTouchTap event 会遇到的问题
    // Needed for onTouchTap
    // http://stackoverflow.com/a/34015469/988941
    injectTapEventPlugin();

    // 用 react-redux 的 Provider 包起来将 store 传递下去，让每个 components 都可以访问到 state
    // 这边使用 browserHistory 当做 history，并使用 material-ui 的 MuiThemeProvider 包裹整个 components
    // 由于这边是简易的 App 我们设计了 Main 为母模版，其有两个子组件 HomePageContainer 和 ResultPageContainer，其中 HomePageContainer 为根位置的子组件
    ReactDOM.render(
      <Provider store={store}>
        <MuiThemeProvider>
          <Router history={browserHistory}>
            <Route path="/" component={Main}>
              <IndexRoute component={HomePageContainer} />
              <Route path="/result" component={ResultPageContainer} />
            </Route>
          </Router>
        </MuiThemeProvider>
      </Provider>,
      document.getElementById('app')
    );
    ```

2. Actions

    首先先定义 actions 常数：

    ```javascript
    export const SHOW_SPINNER = 'SHOW_SPINNER';
    export const HIDE_SPINNER = 'HIDE_SPINNER';
    export const GET_GITHUB_INITIATE = 'GET_GITHUB_INITIATE';
    export const GET_GITHUB_SUCCESS = 'GET_GITHUB_SUCCESS';
    export const GET_GITHUB_FAIL = 'GET_GITHUB_FAIL';
    export const CHAGE_USER_ID = 'CHAGE_USER_ID';
    ```

    现在我们来规划我们的 actions 的部份，这个范例我们使用到了 `redux-thunk` 来处理异步的 action（若读者对于新的 Ajax 处理方式 fetch() 不熟悉可以先[参考这个文档](https://developer.mozilla.org/zh-TW/docs/Web/API/GlobalFetch/fetch)）。以下是 `src/actions/githubActions.js` 完整代码：

    ```javascript
    // 这边引入了 fetch 的 polyfill，考以让旧的浏览器也可以使用 fetch
    import 'whatwg-fetch';
    // 引入 actionTypes 常数
    import {
      GET_GITHUB_INITIATE,
      GET_GITHUB_SUCCESS,
      GET_GITHUB_FAIL,
      CHAGE_USER_ID,
    } from '../constants/actionTypes';

    // 引入 uiActions 的 action
    import {
      showSpinner,
      hideSpinner,
    } from './uiActions';

    // 这边是这个范例的重点，要学习我们之前尚未讲解的异步 action 处理方式：不同于一般同步 action 直接发送 action，异步 action 会回传一个带有 dispatch 参数的 function，里面使用了 Ajax（这里使用 fetch()）进行处理
    // 一般和 API 交互的流程：INIT（开始请求/秀出 spinner）-> COMPLETE（完成请求/隐藏 spinner）-> ERROR（请求失败）
    // 这次我们虽然没有使用 redux-actions 但我们还是维持标准 Flux Standard Action 格式：{ type: '', payload: {} }

    export const getGithub = (userId = 'torvalds') => {
      return (dispatch) => {
        dispatch({ type: GET_GITHUB_INITIATE });
        dispatch(showSpinner());
        fetch('https://api.github.com/users/' + userId)
          .then(function(response) { return response.json() })
          .then(function(json) {
            dispatch({ type: GET_GITHUB_SUCCESS, payload: { data: json } });
            dispatch(hideSpinner());
          })
          .catch(function(response) { dispatch({ type: GET_GITHUB_FAIL }) });
      }
    }

    // 同步 actions 处理，回传 action 对象
    export const changeUserId = (text) => ({ type: CHAGE_USER_ID, payload: { userId: text } });
    ```

    以下是 `src/actions/uiActions.js` 负责处理 UI 的行为：

    ```javascript
    import { createAction } from 'redux-actions';
    import {
      SHOW_SPINNER,
      HIDE_SPINNER,
    } from '../constants/actionTypes';

    // 同步 actions 处理，回传 action 对象
    export const showSpinner = () => ({ type: SHOW_SPINNER});
    export const hideSpinner = () => ({ type: HIDE_SPINNER});
    ```

    透过于 `src/actions/index.js` 将我们 actions 输出

    ```javascript
    export * from './uiActions';
    export * from './githubActions';
    ```

3. Reducers

    接下来我们要来设置一下 Reducers 和 models（initialState 格式）的设计，注意我们这个范例都是使用 `ImmutableJS`。以下是 `src/constants/models.js`：

    ```javascript
    import Immutable from 'immutable';

    export const UiState = Immutable.fromJS({
      spinnerVisible: false,
    });

    // 我们使用 userId 来暂存用户 ID，data 存放 Ajax 取回的数据
    export const GithubState = Immutable.fromJS({
      userId: '',
      data: {},
    });
    ```

    以下是 `src/reducers/data/githubReducers.js`：

    ```javascript
    import { handleActions } from 'redux-actions';
    import { GithubState } from '../../constants/models';

    import {
      GET_GITHUB_INITIATE,
      GET_GITHUB_SUCCESS,
      GET_GITHUB_FAIL,
      CHAGE_USER_ID,
    } from '../../constants/actionTypes';

    const githubReducers = handleActions({
      // 当用户按送出按钮，发出 GET_GITHUB_SUCCESS action 时将接收到的数据 merge
      GET_GITHUB_SUCCESS: (state, { payload }) => (
        state.merge({
          data: payload.data,
        })
      ),
      // 当用户输入用户 ID 会发出 CHAGE_USER_ID action 时将接收到的数据 merge
      CHAGE_USER_ID: (state, { payload }) => (
        state.merge({
          'userId':
          payload.userId
        })
      ),
    }, GithubState);

    export default githubReducers;

    ```

    以下是 `src/reducers/ui/uiReducers.js`：

    ```javascript
    import { handleActions } from 'redux-actions';
    import { UiState } from '../../constants/models';

    import {
      SHOW_SPINNER,
      HIDE_SPINNER,
    } from '../../constants/actionTypes';

    // 随着 fetch 结果显示 spinner
    const uiReducers = handleActions({
      SHOW_SPINNER: (state) => (
        state.set(
          'spinnerVisible',
          true
        )
      ),
      HIDE_SPINNER: (state) => (
        state.set(
          'spinnerVisible',
          false
        )
      ),
    }, UiState);

    export default uiReducers;
    ```

    将 reduces 使用 `redux-immutable` 的 `combineReducers` 在一起。以下是 `src/reducers/index.js`：

    ```javascript
    import { combineReducers } from 'redux-immutable';
    import ui from './ui/uiReducers';// import routes from './routes';
    import github from './data/githubReducers';// import routes from './routes';

    const rootReducer = combineReducers({
      ui,
      github,
    });

    export default rootReducer;
    ```

    运用 redux 提供的 createStore API 把 `rootReducer`、`initialState`、`middlewares` 集成后创建出 store。以下是 `src/store/configureSotore.js`

    ```javascript
    import { createStore, applyMiddleware } from 'redux';
    import reduxThunk from 'redux-thunk';
    import createLogger from 'redux-logger';
    import Immutable from 'immutable';
    import rootReducer from '../reducers';

    const initialState = Immutable.Map();

    export default createStore(
      rootReducer,
      initialState,
      applyMiddleware(reduxThunk, createLogger({ stateTransformer: state => state.toJS() }))
    );
    ```

4. Build Component

    终于我们进入了 View 的细节设计，首先我们先针对母模版，也就是每个页面都会出现的 `AppBar` 做设计。以下是 `src/components/Main/Main.js`：

    ```javascript
    import React from 'react';
    // 引入 AppBar
    import AppBar from 'material-ui/AppBar';

    const Main = (props) => (
      <div>
        <AppBar
          title="Github Finder"
          showMenuIconButton={false}
        />
        <div>
          {props.children}
        </div>
      </div>
    );

    // 进行 propTypes 验证
    Main.propTypes = {
      children: React.PropTypes.object,
    };

    export default Main;
    ```

    以下是 `src/components/HomePage/HomePage.js`：

    ```javascript
    import React from 'react';
    // 使用 react-router 的 Link 当做超链接，发送 userId 当作 query
    import { Link } from 'react-router';
    import RaisedButton from 'material-ui/RaisedButton';
    import TextField from 'material-ui/TextField';
    import IconButton from 'material-ui/IconButton';
    import FontIcon from 'material-ui/FontIcon';

    const HomePage = ({
      userId,
      onSubmitUserId,
      onChangeUserId,
    }) => (
      <div>
        <TextField
          hintText="Please Key in your Github User Id."
          onChange={onChangeUserId}
        />
        <Link to={{
          pathname: '/result',
          query: { userId: userId }
        }}>
          <RaisedButton label="Submit" onClick={onSubmitUserId(userId)} primary />
        </Link>
      </div>
    );

    export default HomePage;
    ```

    以下是 `src/components/ResultPage/ResultPage.js`，将 `userId` 当作 `props` 传给 `<GithubBox />`：

    ```javascript
    import React from 'react';
    import GithubBox from '../../components/GithubBox';

    const ResultPage = (props) => (
      <div>
        <GithubBox data={props.data} userId={props.location.query.userId} />
      </div>
    );

    export default ResultPage;
    ```

    以下是 `src/components/GithubBox/GithubBox.js`，负责截取的 Github 数据呈现：

    ```javascript
    import React from 'react';
    import { Link } from 'react-router';
    // 引入 material-ui 的卡片式组件
    import { Card, CardActions, CardHeader, CardMedia, CardTitle, CardText } from 'material-ui/Card';
    // 引入 material-ui 的 RaisedButton
    import RaisedButton from 'material-ui/RaisedButton';
    // 引入 ActionHome icon
    import ActionHome from 'material-ui/svg-icons/action/home';

    const GithubBox = (props) => (
      <div>
        <Card>
          <CardHeader
            title={props.data.get('name')}
            subtitle={props.userId}
            avatar={props.data.get('avatar_url')}
          />
          <CardText>
            Followers : {props.data.get('followers')}
          </CardText>
          <CardText>
            Following : {props.data.get('following')}
          </CardText>
          <CardActions>
            <Link to="/">
              <RaisedButton
                label="Back"
                icon={<ActionHome />}
                secondary={true}
              />
            </Link>
          </CardActions>
        </Card>
      </div>
    );

    export default GithubBox;
    ```

5. Connect State to Component

    最后，我们要将 Container 和 Component 连接在一起（若忘记了，请先回去复习 Container 与 Presentational Components 入门！）。以下是 `src/containers/HomePage/HomePage.js`，负责将 userId 和使用到的事件处理方法用 props 传进 component ：

    ```javascript
    import { connect } from 'react-redux';
    import HomePage from '../../components/HomePage';

    import {
      getGithub,
      changeUserId,
    } from '../../actions';

    export default connect(
      (state) => ({
        userId: state.getIn(['github', 'userId']),
      }),
      (dispatch) => ({
        onChangeUserId: (event) => (
          dispatch(changeUserId(event.target.value))
        ),
        onSubmitUserId: (userId) => () => (
          dispatch(getGithub(userId))
        ),
      }),
      (stateProps, dispatchProps, ownProps) => {
        const { userId } = stateProps;
        const { onSubmitUserId } = dispatchProps;
        return Object.assign({}, stateProps, dispatchProps, ownProps, {
          onSubmitUserId: onSubmitUserId(userId),
        });
      }
    )(HomePage);
    ```

    以下是 `src/containers/ResultPage/ResultPage.js`：

    ```javascript
    import { connect } from 'react-redux';
    import ResultPage from '../../components/ResultPage';

    export default connect(
      (state) => ({
        data: state.getIn(['github', 'data'])
      }),
      (dispatch) => ({})
    )(ResultPage);
    ```

6. That's it

    若一切顺利的话，这时候你可以在终端机下 `$ npm start` 指令，然后在 `http://localhost:8008` 就可以看到你的努力成果啰！

    ![React Redux](./images/demo-1.png "React Redux")

## 总结

本章带领读者们从零开始集成 React + Redux + ImmutableJS + React Router 搭配 Github API 制作一个简单的 Github 用户查找应用。下一章我们将挑战高端应用，学习 Server Side Rendering 方面的知识，并用 React + Redux + Node（Isomorphic）开发一个食谱分享网站。

## 延伸阅读

1. [Tutorial: build a weather app with React](http://joanmira.com/tutorial-build-a-weather-app-with-react/)
2. [OpenWeatherMap](http://openweathermap.org/)
3. [Weather Icons](https://erikflowers.github.io/weather-icons/)
4. [Weather API Icons](https://erikflowers.github.io/weather-icons/api-list.html)
5. [Material UI](http://www.material-ui.com/#/)
6. [【翻译】这个API很“迷人”——(新的Fetch API)](http://www.w3ctech.com/topic/854)
7. [Redux: trigger async data fetch on React view event](http://stackoverflow.com/questions/33304225/redux-trigger-async-data-fetch-on-react-view-event)
8. [Github API](https://api.github.com/)
9. [传统 Ajax 已死，Fetch 永生](https://github.com/camsong/blog/issues/2)

## :door: 任意门

| [回首页](https://github.com/aclk/reactjs101) | [上一章：Container 与 Presentational Components 入门](https://github.com/aclk/reactjs101/blob/master/Ch08/container-presentational-component-.md) | [下一章：React Redux Sever Rendering（Isomorphic JavaScript）入门](https://github.com/aclk/reactjs101/blob/master/Ch10/react-redux-server-rendering-isomorphic-javascript.md) |

| [纠错、提问或想法](https://github.com/aclk/reactjs101/issues) |
