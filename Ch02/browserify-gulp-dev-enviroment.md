# Browserify + Gulp + Babelify

![一看就懂的 React 开发环境建置与 Webpack 入门教学](./images/react-browserify-gulp.png "一看就懂的 React 开发环境建置与 Webpack 入门教学")

在进入第二种方法前，首先先介绍一下会用到 [Browserify](http://browserify.org/)、[Gulp](http://gulpjs.com/)、[Babelify](https://github.com/babel/babelify) 三种前端开发常会用到的工具：

[Browserify](http://browserify.org/)

- 如同官网上说明的：`Browserify lets you require('modules') in the browser by bundling up all of your dependencies.
`，Browserify 是一个可以让你在浏览器端也能使用像 Node 用的 [CommonJS](https://en.wikipedia.org/wiki/CommonJS) 规范一样，用输出（export）和引用（require）来管理模组。此外，也能使用许多在 NPM 中的模组

[Gulp](http://gulpjs.com/)

- `Gulp` 是一个前端任务工具自动化管理工具。随著前端工程的发展（Task Runner），我们在开发前端应用程式时有许多工作是必须重复进行，例如：打包文件、uglify、将 LESS 编译成一般的 CSS 的文件，编译 ES6 语法等工作。若是使用一般手动的方式，往往会造成效率的低下，所以通过像是 [Grunt](http://gruntjs.com/)、Gulp 这类的 Task Runner 不但可以提升效率，也可以更方便管理这些任务。由于 Gulp 是通过 pipeline 方式来处理文件，在使用上比起 Grunt 的方式直观许多，所以这边我们主要讨论的是 Gulp

[Babelify](https://github.com/babel/babelify)

- `Babelify` 是一个使用 Browserify 进行 Babel 转换的外挂，你可以想成是一个翻译机，可以将 React 中的 `JSX` 或 `ES6` 语法转成浏览器相容的 `ES5` 语法

初步了解了三种工具的概念后，接下来我们就开始我们的环境设置：

1. 若是电脑中尚未安装 [Node](https://zh.wikipedia.org/zh-tw/Node.js)（Node.js 是一个开放源码、跨平台的、可用于服务器端和网路应用的 Google V8 引擎执行运行环境）和 NPM（Node 包管理器 Node Package Manager。是一个以 JavaScript 编写的软体包管理系统，预设环境为 Node.js，从 Node.js 0.6.3 版本开始，npm 被自动附带在安装包中）的话，请先 [上官网安装](https://nodejs.org/en/)

2. 用 `npm` 安装 `browserify`

3. 用 `npm` 安装 `gulp`、`gulp-concat`、`gulp-html-replace`、`gulp-streamify`、`gulp-uglify`、`watchify`、`vinyl-source-stream` 开发环境用的包（development dependencies）

    ```bash
    // 使用 npm install --save-dev 会将安装的包名称和版本存放到 package.json 的 devDependencies 栏位中
    $ npm install --save-dev gulp gulp-concat gulp-html-replace gulp-streamify gulp-uglify watchify vinyl-source-stream
    ```

4. 安装 `babelify`、`babel-preset-es2015`、`babel-preset-react`，编译 `ES6` 和 `JSX` 开发环境用的包，并于根目录底下设定 `.babelrc`，设定编译规则（presets：es2015、react）和使用的外挂

    ```bash
    // 使用 npm install --save-dev 会将安装的包名称和版本存放到 package.json 的 devDependencies 栏位中
    $ npm install --save-dev babelify babel-preset-es2015 babel-preset-react
    ```

    ```js
    // filename: .babelrc
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

6. 编写 Component

    ```js
    // filename: ./app/index.js
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

    ```html
    <!-- filename: ./index.html -->
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Hello React!</title>
    </head>
    <body>
        <div id="app"></div>
        <!-- build:js -->
        <script src="./dist/src/bundle.js"></script>
        <!-- endbuild -->
    </body>
    </html>
    ```

7. 设定 `gulpfile.js`

    ```js
    // filename: gulpfile.js
    // 引入所有需要的文件
    const gulp = require('gulp');
    const uglify = require('gulp-uglify');
    const htmlreplace = require('gulp-html-replace');
    const source = require('vinyl-source-stream');
    const browserify = require('browserify');
    const watchify = require('watchify');
    const babel = require('babelify');
    const streamify = require('gulp-streamify');
    // 文件位置参数
    const path = {
      HTML: 'index.html',
      MINIFIED_OUT: 'bundle.min.js',
      OUT: 'bundle.js',
      DEST: 'dist',
      DEST_BUILD: 'dist/build',
      DEST_SRC: 'dist/src',
      ENTRY_POINT: './app/index.js'
    };
    // 复制 html 到 dist 文件夹中
    gulp.task('copy', function(){
      gulp.src(path.HTML)
        .pipe(gulp.dest(path.DEST));
    });
    // 监听文件是否有变化，若有变化则重新编译一次
    gulp.task('watch', function() {
      gulp.watch(path.HTML, ['copy']);
    var watcher  = watchify(browserify({
        entries: [path.ENTRY_POINT],
        transform: [babel],
        debug: true,
      }));
    return watcher.on('update', function () {
        watcher.bundle()
          .pipe(source(path.OUT))
          .pipe(gulp.dest(path.DEST_SRC))
          console.log('Updated');
      })
        .bundle()
        .pipe(source(path.OUT))
        .pipe(gulp.dest(path.DEST_SRC));
    });
    // 执行 build production 的流程（包括 uglify、编译等）
    gulp.task('copy', function(){
      browserify({
        entries: [path.ENTRY_POINT],
        transform: [babel],
      })
        .bundle()
        .pipe(source(path.MINIFIED_OUT))
        .pipe(streamify(uglify(path.MINIFIED_OUT)))
        .pipe(gulp.dest(path.DEST_BUILD));
    });
    // 将 script 引用换成 production 的文件
    gulp.task('replaceHTML', function(){
      gulp.src(path.HTML)
        .pipe(htmlreplace({
          'js': 'build/' + path.MINIFIED_OUT
        }))
        .pipe(gulp.dest(path.DEST));
    });
    // 设定 NODE_ENV 为 production
    gulp.task('apply-prod-environment', function() {
        process.env.NODE_ENV = 'production';
    });

    // 若直接执行 gulp 会执行 gulp default 的任务：watch、copy。若跑 gulp production，则会执行 build、replaceHTML、apply-prod-environment
    gulp.task('production', ['build', 'replaceHTML', 'apply-prod-environment']);
    gulp.task('default', ['watch', 'copy']);
    ```

8. 成果展示
    到目前为止我们的文件夹的结构应该会是这样：

    ![一看就懂的 React 开发环境建置与 Webpack 入门教学](./images/browserify-folder-pregulp.png "一看就懂的 React 开发环境建置与 Webpack 入门教学")

    接下来我们通过在终端机（terminal）下 `gulp` 指令来处理我们设定好的任务：

    ```bash
    // 当只有输入 gulp 没有输入任务名称时，gulp 会自动执行 default 的任务，我们这边会执行 `watch` 和 `copy` 的任务，前者会监听 `./app/index.js` 是否有改变，有的话则更新。后者则是会把 `index.html` 复制到 `./dist/index.html`
    $ gulp
    ```

    当执行完 `gulp` 后，我们可以发现多了一个 `dist` 文件夹

    ![一看就懂的 React 开发环境建置与 Webpack 入门教学](./images/browserify-folder-possgulp.png "一看就懂的 React 开发环境建置与 Webpack 入门教学")

    如果我们是要进行 `production` 的应用程序开发的话，我们可以执行：

    ```bash
    // 当输入 gulp production 时，gulp 会执行 production 的任务，我们这边会执行 `replaceHTML`、`build` 和 `apply-prod-environment` 的任务，`build` 任务会进行编译和 `uglify`。`replaceHTML` 会取代 `index.html` 注解中的 `<script>` 引入文件，变成引入压缩和 `uglify` 后的 `./dist/build/bundle.min.js`。`apply-prod-environment` 则是会更改 `NODE_ENV` 变数，让环境设定改为 `production`，有兴趣的读者可以参考[React 官网说明](https://facebook.github.io/react/downloads.html)
    $ gulp production
    ```

    此时我们可以在浏览器上打开我们的 `./dist/hello.html`，就可以看到 `Hello, world!` 了！

(image via [srinisoundar](https://cdn-images-1.medium.com/max/477/1*qhI4E_g3TDOK0uu1VAJlCQ.png)、[sitepoint](https://d2sis3lil8ndrq.cloudfront.net/screencasts/46e215cd-2eb3-4cf0-b699-713977a2b644.png)、[keyholesoftware](https://keyholesoftware.com/wp-content/uploads/Browserify-5.png)、[survivejs](http://survivejs.com/webpack/images/webpack.png))

## :door: 任意门

| [回首页](https://github.com/aclk/reactjs101) |

| [纠错、提问或想法](https://github.com/aclk/reactjs101/issues) |
