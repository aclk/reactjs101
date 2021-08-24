# React 生态系（Ecosystem）入门简介

![React 生态系（Ecosystem）入门简介](./images/react-eco-wp.gif "React 生态系（Ecosystem）入门简介")

根据 [React 官方网站](https://facebook.github.io/react/) 的说明：React 是一个专注于 UI（View）的 JavaScript 函数库（Library）。自从 Facebook 于 2013 年开源 React 这个函数库后，相关的生态系开始蓬勃发展。事实上，通过学习 React 生态系（ecosystem）的过程中，可以让我们顺便学习现代化 Web 开发的重要观念（例如：模块化、ES6+、Webpack、Babel、ESLint、函数式程序设计等），成为更好的开发者。

## ReactJS

ReactJS 是 Facebook 推出的 JavaScript 函数库，若以 MVC 框架来看，React 定位是在 View 的范畴。在 ReactJS 0.14 版之后，ReactJS 更把原先处理 DOM 的部分独立出去（react-dom），让 ReactJS 核心更单纯，也更符合 React 所倡导的 `Learn once, write everywhere` 的理念。事实上，ReactJS 本身的 API 相对单纯，但由于整个生态系非常庞大，因此学习 React 却是一条漫长的道路。此外，当你想把 React 应用在你的应用程序时，你通常必须学习整个 React Stack 才能充分发挥 React 的最大优势。

## JSX

事实上，JSX 并非一种全新的语言，而是一种语法糖（[Syntatic Sugar](https://en.wikipedia.org/wiki/Syntactic_sugar)），一种语法类似 [XML](https://zh.wikipedia.org/wiki/XML) 的 ECMAScript 语法扩充。在 JSX 中 HTML 和组建这些元素标签的程序码有紧密的关系，这和过去我们强调 HTML、JavaScript 分离的观念有很大不同。当然，你可以选择不要在 React 使用 JSX，不过相信我，当你真正开始编写 React 元件（Component）时，你会很庆幸有 JSX 真好。

## NPM

NPM（Node Package Manager）是 Node.js 下的主流包管理工具。在 NPM 上有非常多的包，可以让你不用再重造轮子，更可以让你可以轻松用指令管理不同的包。由于 NPM 主要是基于 [CommonJS](https://en.wikipedia.org/wiki/CommonJS) 的规范，通常必须搭配 Browserify 这样的工具才能在前端使用 NPM 的模块。然而因 NPM 是基于 Nested Dependency Tree，不同的包有可能会在引入依赖时会引入相同但不同版本的包，造成档案大小过大的情形。这和另一个包管理工具 [Bower](https://bower.io/) 专注在前端包且使用 Flat Dependency Tree（让使用者决定相依的包版本）是比较不同的地方。

## ES6+

[ES6+](https://babeljs.io/blog/2015/06/07/react-on-es6-plus) 系指 ES6（ES2015）和 ES7 的联集，在 ES6+ 新的标准当中引入许多新的特性和功能，弥补了过去 JavaScript 被诟病的一些特性。由于未来 React 将以支持 ES6+ 为主，因此直接学习 ES6+ 用法是相对好的选择，本书的所有范例也将会以 ES6+ 编写。

## Babel

由于并非所有浏览器都支持 ES6+ 语法，所以通过 [Babel](https://babeljs.io/) 这个 JavaScript 编译器（可以想成是翻译机）可以让你的 ES6+ 、JSX 等程序码转换成浏览器可以看的懂得语法。通常会在数据夹的 root 位置加入 `.babelrc` 进行转译规则 `preset` 和引用外挂（plugin）的设定。

## JavaScript 模块化开发

随著 Web 应用程序的复杂性提高，JavaScript 模块化开发已经成为必然的趋势，以下简单介绍 JavaScript 模块化的相关规范。事实上，在一开始没有官方定义的标准时出现了各种社群自行定义的规范和实践。

1. CDN-Based

    也就是最传统的 `<script>` 引入方式，然而使用这种方式虽然简单方便，但在开发实际中大型应用程序时会产生许多弊端：

    - 全局作用域容易造成变数污染和冲突
    - 文件只能依照 `<script>` 顺序载入，不具弹性
    - 在大型专案中各种资源和版本难以维护
    - 必须由开发者自行判断模块和函数库之间的依赖关系

2. AMD

    [Asynchronous Module Definition](https://en.wikipedia.org/wiki/Asynchronous_module_definition) 简称 AMD，为非同步载入模块的规范，其在定义时模块时即需定义依赖的模块。AMD 常用于浏览器端，其最著名的实践为 [RequireJS](http://requirejs.org/)

    基本格式：

    ```js
    define(id?, dependencies?, factory);
    ```

3. CommonJS

    [CommonJS](http://wiki.commonjs.org/wiki/CommonJS) 规范是一种同步模块载入的规范。以 Node.js 其遵守 CommonJS 规范，使用 `require` 进行模块同步载入，并通过 `exports`、`module.exports` 来输出模块。主要实现为 [Node.js](https://nodejs.org/en/) 服务器端的同步载入和浏览器端的 [Browserify](http://browserify.org/)。

4. CMD

    CMD 全称为 [Common Module Definition](https://github.com/cmdjs/specification/blob/master/draft/module.md)，其规范和 AMD 类似，但相对简洁，却又保持和 CommonJS 的兼容性。其最大特色为：依赖就近，延迟执行。主要实现为：[Sea.js](http://seajs.org/docs/#intro)。

5. UMD

    [Universal Module Definition](https://github.com/umdjs/umd) 是为了要兼容 CommonJS 和 AMD 所设计的规范，希望让模块能跨平台执行。

6. ES6 Module

    ECMAScript6 的标准中定义了 JavaScript 的模块化方式，让 JavaScript 在开发大型复杂应用程序时上更为方便且易于管理，亦可以取代过去 AMD、CommonJS 等规范，成为通用于浏览器端和服务器端的模块化解决方案。但目前浏览器和 Node 在 ES6 模块支持度还不完整，大部分情况需要通过 [Babel](https://babeljs.io/) 转译器进行转译。

## Webpack/Browserify + Gulp

随著网页应用程序开发的复杂性提升，现在的网页往往不单只是单纯的网页，而是一个网页应用程序（WebApp）。为了管理复杂的应用程序开发，此时模块化开发方法便显得日益重要，而理想上的模块化开发工具一直是前端工程的很大的议题。Webpack 和 Browserify + Gulp 则是进行 React 应用程序开发常用的开发工具，可以协助进行自动化程序码打包、转译等重复性工作，提升开发效率。本书范例主要会搭配 Webpack 进行开发。

1. Webpack

    [Webpack](https://webpack.github.io/) 是一个模块打包工具（module bundler），以下列出 Webpack 的几项主要功能：
    - 将 CSS、图片与其他资源打包
    - 打包之前预处理（Less、CoffeeScript、JSX、ES6 等）的档案
    - 依 entry 文件不同，把 .js 分拆为多个 .js 档案
    - 整合丰富的 Loader 可以使用（Webpack 本身仅能处理 JavaScript 模块，其余档案如：CSS、Image 需要载入不同 Loader 进行处理）

2. Browserify

    如同官网上说明的：`Browserify lets you require('modules') in the browser by bundling up all of your dependencies.`，Browserify 是一个可以让你在浏览器端也能使用像 Node 用的 [CommonJS](https://en.wikipedia.org/wiki/CommonJS) 规范一样，用输出（export）和引用（require）来管理模块。此外，也能让前端使用许多在 NPM 中的模块。

3. Gulp

    `Gulp` 是一个前端任务工具自动化管理工具（Task Runner）。随著前端工程的发展，我们在开发前端应用程序时有许多工作是必须重复进行，例如：打包文件、uglify、将 LESS 转译成一般的 CSS 的档案，转译 ES6 语法等工作。若是使用一般手动的方式，往往会造成效率的低下，所以通过像是 [Grunt](http://gruntjs.com/)、Gulp 这类的 Task Runner 不但可以提升效率，也可以更方便管理这些任务。由于 Gulp 是通过 pipeline 方式来处理档案，在使用上比起 Grunt 的方式直观许多，所以这边我们主要讨论的是 Gulp。

## ESLint

[ESLint](http://eslint.org/) 是一个提供 JavaScript 和 JSX 的程序码检查工具，可以确保团队的程序码品质。其支持可插拔的特性，可以根据需求在 `.eslintrc` 设定检查规则。目前主流的检查规则会使用 Airbnb 所发布的 [Airbnb React/JSX Style Guide](https://github.com/airbnb/javascript/tree/master/react)，在使用上需先安装 [eslint-config-airbnb](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb) 等包。

## React Router

[React Router](https://github.com/reactjs/react-router) 是 React 中主流使用的 Routing 函数库，通过 URL 的变化来管理对应的状态和元件。若开发不刷页的单页式（single page application）的 React 应用程序通常都会需要用到。

## Flux/Redux

[Flux](https://facebook.github.io/flux/) 是一个实现单项流的应用程序数据架构（architecture），同样是由 Facebook 推出，并和 React 专注于 View 的部份形成互补。而由 Dan Abramov 所开发的 [Redux](https://github.com/reactjs/redux) 被 React 开发社群认为是 Flux-like 更优雅的作法，也是目前主流搭配 React 的状态（State）管理工具。让你在开发复杂的应用程序时可以更方便管理你的状态（state）。

## ImmutableJS

[ImmutableJS](https://facebook.github.io/immutable-js/)，是一个能让开发者建立不可变数据结构的函数库。建立不可变（immutable）数据结构不仅可以让状态可预测性更高，也可以提升程序的性能。

## Isomorphic JavaScript

Isomorphic JavaScript 是指前后端（Client/Server）共用相同部分的程序码，让 JavaScript 应用可以同时执行在浏览器端和服务器端，在 React 中可以通过服务器端渲染（server side rendering）静态 HTML 的方式达到 Isomorphic JavaScript 效果，让 SEO 和执行性能更加提升并让前后端共用程序码。而另一个常一起出现的 Universal JavaScript 一般定义更为广泛，系指可以运行在不同环境下的 JavaScript Code，并不局限于浏览器和服务器端。但要留意的是在 Github 和许多技术文章的分享上会把两者定义为同一件事情。

## React 测试

Facebook 本身有提供 [Test Utilities](https://facebook.github.io/react/docs/test-utils.html)，但由于不够好用，所以目前主流开发社群比较倾向使用 Airbnb 团队开发的 [enzyme](https://github.com/airbnb/enzyme)，其可以与市面上常见的测试工具（[Mocha](https://mochajs.org/)、[Karma](https://karma-runner.github.io/)、Jest 等）搭配使用。其中 [Jest](https://facebook.github.io/jest/) 是 Facebook 所开发的单元测试工具，其主要基于 [Jasmine](http://jasmine.github.io/) 所建立的测试框架。Jest 除了支持 JSDOM 外，也可以自动模拟 (mock) 通过 `require()` 进来的模块，让开发者可以更专注在目前被测试的模块中。

## React Native

[React Native](https://facebook.github.io/react-native/)和过去的 [Apache Cordova](https://cordova.apache.org/) 等基于 WebView 的解决方案比较不同，它让开发者可以使用 React 和 JavaScript 开发原生应用程序（Native App），让 `Learn once, write anywhere` 理想变得可能。

## GraphQL/Relay

[GraphQL](http://graphql.org/docs/getting-started/) 是 Facebook 所开发的数据查询语言（Data Query Language），主要是想解决传统 RESTful API 所遇到的一些问题，并提供前端更有弹性的 API 设计方式。[Relay](https://facebook.github.io/relay/) 则是 Facebook 提出搭配 GraphQL 用于 React 的一个定义式数据框架，可以降低 Ajax 的请求数量（类似的框架还有 Netflix 推出的 [Falcor](https://netflix.github.io/falcor/)）。但由于目前主流的后端 API 仍以传统 RESTful API 设计为主，所以在使用 GraphQL 上通常会需要比较大架构设计的变动。因此本书则是把 GraphQL/Relay 介绍放到附录的部份，让有兴趣的读者可以自行参考体验一下。

## 总结

以上就是读者在 React 生态系游走时会遇到的各种关卡，也许有些初学者会对于这样庞大的体系所吓到，放弃学习 React 这项革新性技术的机会。不过别担心，接下来笔者将带领读者按图索骥，依序介绍整个 React 生态系的各种技术，一步步带领大家用 React 实作出生活中会用到的应用程序。

## 延伸阅读

1. [Navigating the React.JS Ecosystem](https://www.toptal.com/react/navigating-the-react-ecosystem)
2. [petehunt/react-howto](https://github.com/petehunt/react-howto#learning-relay-falcor-etc)
3. [React Ecosystem - A summary](https://staminaloops.github.io/undefinedisnotafunction/react-ecosystem/)
4. [React Official Site](https://facebook.github.io/react/)
5. [A collection of awesome things regarding React ecosystem](https://github.com/enaqx/awesome-react)
6. [Webpack 中文指南](http://zhaoda.net/webpack-handbook/index.html)
7. [AMD 和 CMD 的区别有哪些？](https://www.zhihu.com/question/20351507)
8. [jslint to eslint](https://www.qianduan.net/jslint-to-eslint/)
9. [Facebook的Web开发三板斧：React.js、Relay和GraphQL](http://1ke.co/course/595)
10. [airbnb/javascript](https://github.com/airbnb/javascript)

（image via [jpsierens](http://jpsierens.com/wp-content/uploads/2016/06/react-eco-wp.gif)）

## :door: 任意门

| [回首页](https://github.com/aclk/reactjs101) | [上一章：Web 前端工程入门简介](https://github.com/aclk/reactjs101/blob/master/Ch01/front-end-introduction.md) | [下一章：React 开发环境设置与 Webpack 入门教学](https://github.com/aclk/reactjs101/blob/master/Ch02/webpack-dev-enviroment.md) |

| [纠错、提问或许愿](https://github.com/aclk/reactjs101/issues) |
