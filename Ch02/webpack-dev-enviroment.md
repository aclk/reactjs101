# React 开发环境设置与 Webpack 入门教学

![React 开发环境设置与 Webpack 入门教学](./images/react-webpack-browserify.png "React 开发环境设置与 Webpack 入门教学")

## 前言

俗话说工欲善其事，必先利其器。写程序也是一样，搭建好开发环境后可以让自己在后续开发上更加顺利。因此本章接下来将讨论 React 开发环境的两种主要方式：CDN-based、 [webpack](https://webpack.github.io/)（这边我们就先不讨论 [TypeScript](https://www.typescriptlang.org/) 的开发方式）。至于 [browserify](https://webpack.github.io/) 搭配 [Gulp](http://gulpjs.com/) 的方法则会放在补充资料中，让读者阅读完本章后可以开始 React 开发之旅！

## JavaScript 模块化

随著网站开发的复杂度提升，许多现代化的网站已不是单纯的网站而已，更像是个富有互动性的网页应用程序（Web App）。为了应付现代化网页应用程序开发的需求，解决一些像是全局变量污染、低维护性等问题，JavaScript 在模块化上也有长足的发展。过去一段时间读者们或许听过像是 `Webpack`、`Browserify`、`module bundlers`、`AMD`、`CommonJS`、`UMD`、`ES6 Module` 等有关 JavaScript 模块化开发的专有名词或工具，在前面一个章节我们也简单介绍了关于 JavaScript 模块化的简单观念和规范介绍。若是读者对于 JavaScript 模块化开发尚不熟悉的话推荐可以参考 [这篇文章](http://huangxuan.me/2015/07/09/js-module-7day/) 和 [这篇文章](https://medium.freecodecamp.com/javascript-modules-a-beginner-s-guide-783f7d7a5fcc#.oa2n5s5zt) 当作入门。

总的来说，使用模块化开发 JavaScript 应用程序主要有以下三种好处：

1. 提升维护性（Maintainability）
2. 命名空间（Namespacing）
3. 提供可重用性（Reusability）

而在 React 应用程序开发上更推荐使用像是 `Webpack` 这样的 `module bundlers` 来组织我们的应用程序，但对于一般读者来说 `Webpack` 强大而完整的功能相对复杂。为了让读者先熟悉 `React` 核心观念（我们假设读者已经有使用 `JavaScript` 或 `jQuery` 的基本经验），我们将从使用 `CDN` 引入 `<script>` 的方式开始介绍：

![React 开发环境设置与 Webpack 入门教学](./images/react.png "React 开发环境设置与 Webpack 入门教学")
使用 CDN-based 的开发方式缺点是较难维护我们的程序代码（当引入函数库一多就会有很多 `<script/>`）且会容易遇到版本相容性问题，不太适合开发大型应用程序，但因为简单易懂，适合教学上使用。

以下是 React [官方首页的范例](https://facebook.github.io/react/index.html)，以下使用 `React v15.2.1`：

1. 理解 `React` 是 `Component` 导向的应用程序设计
2. 引入 `react.js`、`react-dom.js`（react 0.14 后将 react-dom 从 react 核心分离，更符合 react 跨平台抽象化的定位）以及 `babel-standalone` 版 script（可以想成 `babel` 是翻译机，翻译浏览器看不懂的 `JSX` 或 `ES6+` 语法成为浏览器看的懂得的 `JavaScript`。为了提升效率，通常我们都会在服务器端做转译，这点在 production 环境尤为重要）
3. 在 `<body>` 编写 React Component 要插入（mount）指定节点的地方：`<div id="example"></div>`
4. 通过 `babel` 进行语言翻译 `React JSX` 语法，`babel` 会将其转为浏览器看的懂得 `JavaScript`。其代表意义是：`ReactDOM.render(想要 render 的 Component 或 HTML 元素, 想要插入的位置)`。所以我们可以在浏览器上打开我们的 `hello.html`，就可以看到 `Hello, world!` 。That's it，我们第一个 `React` 应用程序就算完成了！

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Hello React!</title>
    <!-- 以下引入 react.js, react-dom.js（react 0.14 后将 react-dom 从 react 核心分离，更符合 react 跨平台抽象化的定位）以及 babel-core browser 版 -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/15.2.1/react.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/15.2.1/react-dom.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/6.18.1/babel.min.js"></script>
  </head>
  <body>
    <!-- 这边的 id="example" 的 <div> 为 React Component 要插入的地方 -->
    <div id="example"></div>
    <!-- 以下就是包在 babel（通过进行语言翻译）中的 React JSX 语法，babel 会将其转为浏览器看的懂得 JavaScript -->
    <script type="text/babel">
      ReactDOM.render(
        <h1>Hello, world!</h1>,
        document.getElementById('example')
      );
    </script>
  </body>
</html>
```

在浏览器浏览最后成果：

![React 开发环境设置与 Webpack 入门教学](./images/hello-world.png "React 开发环境设置与 Webpack 入门教学")

## Webpack

![React 开发环境设置与 Webpack 入门教学](./images/webpack-module-bundler.png "React 开发环境设置与 Webpack 入门教学")

上面我们先简单介绍了 CDN-based 的开发方式让大家先对于 React 有个基本印象，但由于 CDN-based 的开发方式有不少缺点。所以接下来的 Webpack 将会是我们接下来范例的主要使用的开发工具。

[Webpack](https://webpack.github.io/) 是一个模块打包工具（module bundler），以下列出 Webpack 的几项主要功能：

- 将 CSS、图片与其他资源打包
- 打包之前预处理（Less、CoffeeScript、JSX、ES6 等）文件
- 依 entry 文件不同，把 .js 分拆为多个 .js 文件
- 整合丰富的 Loader 可以使用（Webpack 本身仅能处理 JavaScript 模块，其余文件如：CSS、Image 需要载入不同 Loader 进行处理）

接下来我们一样通过 Hello World 实例来介绍如何用 Webpack 设置 React 开发环境：

1. 依据你的操作系统安装 [Node](https://nodejs.org/en/) 和 [NPM](https://www.npmjs.com/)（目前版本的 Node 都会内建 NPM）

2. 通过 NPM 安装 Webpack（可以 global 或 local project 安装，这边我们使用 local）、webpack loader、webpack-dev-server

    Webpack 中的 loader 类似于 browserify 内的 transforms，但 Webpack 在使用上比较多元，除了 JavaScript loader 外也有 CSS Style 和图片的 loader。此外，`webpack-dev-server` 则可以启动开发用 server，方便我们测试

    ```bash
    // 按指示初始化 NPM 设定档 package.json
    $ npm init
    // --save-dev 是可以让你将安装包的名称和版本信息存放到 package.json，方便日后使用
    $ npm install --save-dev babel-core babel-eslint babel-loader babel-preset-es2015 babel-preset-react html-webpack-plugin webpack webpack-dev-server
    ```

3. 在根目录设定 `webpack.config.js`

    事实上，`webpack.config.js` 有点类似于 `gulp` 中的 `gulpfile.js` 功用，主要是设定 `webpack` 的相关设定

    ```javascript
    // 这边使用 HtmlWebpackPlugin，将 bundle 好的 <script> 插入到 body。${__dirname} 为 ES6 语法对应到 __dirname
    const HtmlWebpackPlugin = require('html-webpack-plugin');

    const HTMLWebpackPluginConfig = new HtmlWebpackPlugin({
      template: `${__dirname}/app/index.html`,
      filename: 'index.html',
      inject: 'body',
    });

    module.exports = {
      // 文件起始点从 entry 进入，因为是阵列所以也可以是多个文件
      entry: [
        './app/index.js',
      ],
      // output 是放入产生出来的结果的相关参数
      output: {
        path: `${__dirname}/dist`,
        filename: 'index_bundle.js',
      },
      module: {
          // loaders 則是放想要使用的 loaders，在这边是使用 babel-loader 将所有 .js（这边用到正则）相关文件（排除了 npm 安裝的套件位置 node_modules）编译成浏览器可以阅读的 JavaScript。preset 则是使用的 babel 编译规则，这边使用 react、es2015。若是已经单独使用 .babelrc 作为 presets 設定的话，则可以省略 query
        loaders: [
          {
            test: /\.js$/,
            exclude: /node_modules/,
            loader: 'babel-loader',
            query: {
              presets: ['es2015', 'react'],
            },
          },
        ],
      },
      // devServer 则是 webpack-dev-server 设定
      devServer: {
        inline: true,
        port: 8008,
      },
      // plugins 放置所使用的外挂
      plugins: [HTMLWebpackPluginConfig],
    };
    ```

4. 在根目录设定 `.babelrc`

    ```javascript
    {
      "presets": [
        "es2015",
        "react",
      ],
      "plugins": []
    }
    ```

5. 安装 react 和 react-dom

    ```bash
    // 安装依赖库
    $ npm install --save react react-dom
    ```

6. 编写 Component（记得把 `index.html` 以及 `index.js` 放到 `app` 文件夹底下喔！）

    `index.html`

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>React Setup</title>
        <link rel="stylesheet" type="text/css" href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css">
    </head>
    <body>
        <!-- 想要插入 React Component 的位置 -->
        <div id="app"></div>
    </body>
    </html>
    ```

    `index.js`

    ```js
    import React from 'react';
    import ReactDOM from 'react-dom';

    class App extends React.Component {
      constructor(props) {
        super(props);
        this.state = {
        };
      }
      render() {
        return (
          <div>
            <h1>Hello, World!</h1>
          </div>
        );
      }
    }

    ReactDOM.render(<App />, document.getElementById('app'));
    ```

7. 在终端机使用 `webpack` 进行成果展示，webpack 相关指令：

    - webpack：会在开发模式下开始一次性的配置
    - webpack -p：会配置 production 的程序代码
    - webpack --watch：会监听程序代码的修改，当储存时有异动时会更新文件
    - webpack -d：加入 source maps 文件
    - webpack --progress --colors：加上处理进度与颜色

    如果不想每次都打一长串的指令码的话可以使用 `package.json` 中的 `scripts` 设定

    ```javascript
    "scripts": {
      "dev": "webpack-dev-server --devtool eval --progress --colors --content-base build"
    }
    ```

    然后在终端机执行：

    ```bash
    // 启动
    $ npm run dev
    ```

当我们此时我们可以打开浏览器输入 `http://localhost:8008` ，就可以看到 `Hello, world!` 了！

## 总结

以上就是 React 开发环境设置与 Webpack 入门教学，看到这边的读者不妨自己动手设定开发环境，体验一下 React 开发环境的感觉，毕竟若是只有阅读文字的话很容易就会忘记喔！若你不想在环境设定上花太多时间的话，不妨参考 Facebook 开发社区推出的 [create-react-app](https://github.com/facebookincubator/create-react-app)，可以快速上手，使用 Webpack、[Babel](https://babeljs.io/)、[ESLint](http://eslint.org/) 开发 React 应用程式。接下来的章节我们将持续延伸 React/JSX/Component 的介绍。

## 延伸阅读

1. [JavaScript 模块化七日谈](http://huangxuan.me/2015/07/09/js-module-7day/)
2. [前端模块化开发那点历史](https://github.com/seajs/seajs/issues/588)
3. [Webpack 中文指南](http://zhaoda.net/webpack-handbook/index.html)
4. [WEBPACK DEV SERVER](http://webpack.github.io/docs/webpack-dev-server.html)

(image via [srinisoundar](https://cdn-images-1.medium.com/max/477/1*qhI4E_g3TDOK0uu1VAJlCQ.png)、[sitepoint](https://d2sis3lil8ndrq.cloudfront.net/screencasts/46e215cd-2eb3-4cf0-b699-713977a2b644.png)、[keyholesoftware](https://keyholesoftware.com/wp-content/uploads/Browserify-5.png)、[survivejs](http://survivejs.com/webpack/images/webpack.png))

## :door: 任意门

| [回首页](https://github.com/aclk/reactjs101) | [上一章：React 生态系（Ecosystem）入门简介](https://github.com/aclk/reactjs101/blob/master/Ch01/react-ecosystem-introduction.md) | [下一章：ReactJS 与 Component 设计入门介绍](https://github.com/aclk/reactjs101/blob/master/Ch03/reactjs-introduction.md) |

| [纠错、提问或想法](https://github.com/aclk/reactjs101/issues) |
