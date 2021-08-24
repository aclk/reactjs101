# 附录一、React ES5、ES6+ 常见用法对照表

![一看就懂的 React ES5、ES6+ 常见用法对照表](./images/react-es6.jpg)

## 前言

[React](https://facebook.github.io/react/) 是 Facebook 推出的开源 [JavaScript](https://en.wikipedia.org/wiki/JavaScript) Library。自从 React 正式开源后，React 生态系开始蓬勃发展。事实上，透过学习 React 生态系（ecosystem）的过程中，可以上我们顺便学习现代化 Web 开发的重要观念（例如：ES6、[Webpack](https://github.com/webpack/webpack)、[Babel](https://babeljs.io/)、组件化等），成为更好的开发者。虽然 ES6（ECMAScript2015）、ES7 是未来趋势（本文將 ES6、ES7 称为 ES6+），然而目前在网络上有许多的学习资源仍是以 ES5 为主，导致读者在学习上遇到一些坑和迷惑（本文假设读者对于 React 已经有些基本认识，若你对于 React 尚不熟悉，建议先行[阅读官方文件](https://facebook.github.io/react/)和[本篇入门教学](https://scotch.io/tutorials/learning-react-getting-started-and-concepts)）。因此本文希望透過整理在 React 中 ES5、ES6+ 常见用法对照表，让读者们可以在实现功能时（尤其在 [React Native](https://facebook.github.io/react-native/)）可以更清楚两者的差异，无痛转移到 ES6+。

## 大纲
1. Modules
2. Classes
3. Method definition
4. Property initializers
5. State
6. Arrow functions
7. Dynamic property names & template strings
8. Destructuring & spread attributes
9. Mixins
10. Default Parameters

## 1. Modules
随着 Web 技术的进展，组件化开发已经成为一个重要课题。关于 JavaScript 组件化我们这边不详述，建议读者参考 [PPT](http://huangxuan.me/js-module-7day/#/) 和 [这篇文章](http://justineo.github.io/singles/writing-modular-js/)。

ES5 若使用 CommonJS 标准，一般使用 `require()` 用法引入模块：

```js
var React = require('react');
var MyComponent = require('./MyComponent');
```

输出则是使用 `module.exports`：

```js
module.exports = MyComponent;
```


ES6+ `import` 用法：

```js
import React from 'react';
import MyComponent from './MyComponent';
```

输出则是使用 `export default`：

```js
export default class MyComponent extends React.Component {

}
```


## 2. Classes
在 React 中组件（Component）是组成视觉页面的基础。在 ES5 中我们使用 `React.createClass()` 来建立 Component，而在 ES6+ 则是用 [Classes](https://babeljs.io/docs/learn-es2015/#classes) 继承 `React.Component` 来建立 Component。若是有写过 Java 等面向对象语言（OOP）的读者应该对于这种写法比较不陌生，不过要注意的是 JavaScript 仍是原型继承类型的面向对象程序语言，只是使用 `Classes` 让面向对象使用上更加直观。对于选择 `class` 使用上还有疑惑的读者建议可以阅读 [React.createClass versus extends React.Component](https://toddmotto.com/react-create-class-versus-component/) 这篇文章。

ES5 `React.createClass()` 用法：

```js
var Photo = React.createClass({
  render: function() {
    return (
      <div>
        <images alt={this.props.description} src={this.props.src} />
      </div>
      );
  }
});
ReactDOM.render(<Photo />, document.getElementById('main'));
```

ES6+ `class` 用法：

```js
class Photo extends React.Component {
  render() {
    return <images alt={this.props.description} src={this.props.src} />;
  }
}
ReactDOM.render(<Photo />, document.getElementById('main'));
```

在 ES5 我們会在 `componentWillMount ` 生命周期定义希望在 `render` 前执行，且只会执行一次的任务：

```js
var Photo = React.createClass({
  componentWillMount: function() {}
});
```

在 ES6+ 则是定义在 `constructor ` 建构子中：

```js
class Photo extends React.Component {
  constructor(props) {
    super(props);
    // 原本在 componentWillMount 操作的动作可以放在这
  }
}
```

## 3. Method definition
在 ES6 中我们使用 `Method` 可以忽略 `function` 和 `,`，使用上更为简洁！ES5 `React.createClass()` 用法：

```js
var Photo = React.createClass({
  handleClick: function(e) {},
  render: function() {}
});
```

ES6+ class 用法：

```js
class Photo extends React.Component {
  handleClick(e) {}
  render() {}
}
```


## 4. Property initializers
Component 属性值是数据传递重要的元素，在 ES5 中我们使用 `propTypes ` 和  `getDefaultProps ` 来定义属性（props）的预设值和型别：

```js
var Todo = React.createClass({
  getDefaultProps: function() {
    return {
      checked: false,
      maxLength: 10,
    };
  },
  propTypes: {
    checked: React.PropTypes.bool.isRequired,
    maxLength: React.PropTypes.number.isRequired
  },
  render: function() {
    return();
  }
});
```

在 ES6+ 中我们则是参考 [ES7 property initializers](https://github.com/jeffmo/es-class-fields-and-static-properties) 使用 `class` 中的静态属性（static properties）来定义：

```js
class Todo extends React.Component {
  static defaultProps = {
    checked: false,
    maxLength: 10,
  }; // 注意有分号
  static propTypes = {
    checked: React.PropTypes.bool.isRequired,
    maxLength: React.PropTypes.number.isRequired
  };
  render() {
    return();
  }
}
```

ES6+ 另外一种写法，可以留意一下，主要是看各团队喜好和规范，选择合适的方式：

```js
class Todo extends React.Component {
    render() {
        return (
            <View />
        );
    }
}
Todo.defaultProps = {
    checked: false,
    maxLength: 10,
};
Todo.propTypes = {
    checked: React.PropTypes.bool.isRequired,
    maxLength: React.PropTypes.number.isRequired,
};
```


## 5. State
在 React 中 `Props` 和 `State` 是数据流传递的重要元素，不同的是 `state` 可更改，可以去执行一些运算。在 ES5 中我们使用 `getInitialState ` 去初始化 `state`：

```js
var Todo = React.createClass({
    getInitialState: function() {
        return {
            maxLength: this.props.maxLength,
        };
    },
});
```

在 ES6+ 中我们初始化 `state` 有两种写法：

```js
class Todo extends React.Component {
    state = {
        maxLength: this.props.maxLength,
    }
}
```

另外一种写法，使用在构造函数初始化。比较推荐使用这种方式，方便做一些运算：

```js
class Todo extends React.Component {
    constructor(props){
        super(props);
        this.state = {
            maxLength: this.props.maxLength,
        };
    }
}
```


## 6. Arrow functions

在讲 `Arrow functions` 之前，我们先聊聊在 React 中 `this` 和它所代表的 `context`。在 ES5 中，我们使用 `React.createClass()` 来建立 Component，而在 `React.createClass()` 下，预设帮你绑定好 `method` 的 `this`，你毋须自行绑定。所以你可以看到像是下面的例子，`callback function` handleButtonClick 中的  `this` 是指到 component 的实例（instance），而非触发事件的对象：

```js
var TodoBtn = React.createClass({
    handleButtonClick: function(e) {
        // 此 this 指到 component 的实例（instance），而非 button
        this.setState({showOptionsModal: true});
    },
    render: function(){
        return (
            <div>
                <Button onClick={this.handleButtonClick}>{this.props.label}</Button>
            </div>
        )
    },
});
```

然而自动绑定这种方式反而会让人容易误解，所以在 ES6+ 推荐使用 `bind ` 绑定 `this` 或使用 `Arrow functions`（它会绑定当前 `scope` 的 `this context`）两种方式，你可以参考下面例子：

```js
class TodoBtn extends React.Component
{
    handleButtonClick(e){
        // 确认绑定 this 指到 component instance
        this.setState({toggle: true});
    }
    render(){
        // 这边可以用 this.handleButtonClick.bind(this) 手动绑定或是 Arrow functions () => {} 用法
        return (
            <div>
                <Button onClick={this.handleButtonClick.bind(this)} onClick={(e)=> {this.handleButtonClick(e)} }>{this.props.label}</Button>
            </div>
        )
    },
}
```

`Arrow functions` 虽然一开始看起来有点怪异，但其实观念很简单：一个简化的函数。函数基本上就是参数（不一定要有参数）、表达式、回传值（也可能是回传 undefined）：

```
// Arrow functions 的一些例子
()=>7
e=>e+2
()=>{
    alert('XD');
}
(a,b)=>a+b
e=>{
    if (e == 2){
        return 2;
    }
    return 100/e;
}
```

不过要注意的是无论是 `bind` 或是 `Arrow functions`，每次执行回传都是指到一个新的函数，若需要再调用到这个函数，请记得先把它存起来：

错误用法：

```js
class TodoBtn extends React.Component{
    componentWillMount(){
        Btn.addEventListener('click', this.handleButtonClick.bind(this));
    }
    componentDidmount(){
        Btn.removeEventListener('click', this.handleButtonClick.bind(this));
    }
    onAppPaused(event){
    }
}
```

正确用法：

```js
class TodoBtn extends React.Component{
    constructor(props){
        super(props);
        this.handleButtonClick = this.handleButtonClick.bind(this);
    }
    componentWillMount(){
        Btn.addEventListener('click', this.handleButtonClick);
    }
    componentDidMount(){
        Btn.removeEventListener('click', this.handleButtonClick);
    }
}
```

更多 Arrows and Lexical This 特性可以[参考这个文件](https://babeljs.io/docs/learn-es2015/#arrows)。


## 7. Dynamic property names & template strings
以前在 ES5 我们要动态设定属性名称时，往往需要多写几行代码才能达到目标：

```js
var Todo = React.createClass({
  onChange: function(inputName, e) {
    var stateToSet = {};
    stateToSet[inputName + 'Value'] = e.target.value;
    this.setState(stateToSet);
  },
});
```

但在 ES6+中，透过 [enhancements to object literals](https://babeljs.io/blog/2015/06/07/react-on-es6-plus) 和 [template strings](https://babeljs.io/docs/learn-es2015/#template-strings) 可以轻松完成动态设定属性名称的任务：

```
class Todo extends React.Component {
  onChange(inputName, e) {
    this.setState({
      [`${inputName}Value`]: e.target.value,
    });
  }
}
```

Template Strings 是一种语法糖（syntactic sugar），方便我们组织字串（这边也用上 `let`、`const` 变数和常数定义的方式，和 `var` 的 `function scope` 不同的是它们是属于 `block scope`，亦即作用域存在于 `{}` 间）：

```js
// Interpolate variable bindings
const name = "Bob", let = "today";
`Hello ${name}, how are you ${time}?` \\ Hello Bob, how are you today?
```


## 8. Destructuring & spread attributes
在 React 的 Component 中，父组件利用 `props` 来传递数据到子组件是常见作法，然而我们有时会希望只传递部分数据，此时 ES6+ 中的 [Destructuring](https://babeljs.io/docs/learn-es2015/#destructuring) 和 [JSX 的 Spread Attributes
](https://facebook.github.io/react/docs/jsx-spread.html) ，`...` Spread Attributes 主要是用来迭代对象：

```js
class Todo extends React.Component {
  render() {
    var {
      className,
      ...others,  // ...others 包含 this.props 除了 className 外所有值。this.props = {value: 'true', title: 'header', className: 'content'}
    } = this.props;
    return (
      <div className={className}>
        <TodoList {...others} />
        <button onClick={this.handleLoadMoreClick}>Load more</button>
      </div>
    );
  }
}
```

但使用上要注意的是若是有重复的属性值则以后来覆盖，下面的例子中若 `...this.props`，有 `className`，则被后来的 `main` 所覆盖：

```js
<div {...this.props} className="main">
  …
</div>
```

而 `Destructuring` 也可以用在简化 `Module` 的引入上，这边我们先用 ES5 中引入方式来看：

```js
var React = require('react-native');
var Component = React.component;

class HelloWorld extends Component {
  render() {
    return (
      <View>
        <Text>Hello, world!</Text>
      </View>
    );
  }
}

export default HelloWorld;
```

以下 ES5 写法：

```
var React = require('react-native');
var View = React.View;
```

在 ES6+ 则可以直接使用 `Destructuring` 这种简化方式来引入模块中的组件：

```
// 这边等于上面的写法
var { View } = require('react-native');
```

更进一步可以使用 `import` 语法：

```js
import React, {
  View,
  Component,
  Text,
} from 'react-native';

class HelloWorld extends Component {
  render() {
    return (
      <View>
        <Text>Hello, world!</Text>
      </View>
    );
  }
}

export default HelloWorld;
```


## 9. Mixins
在 ES5 中，我们可以使用 `Mixins` 的方式去让不同的 Component 共用相似的功能，重用我们的代码：

```js
var PureRenderMixin = require('react-addons-pure-render-mixin');
React.createClass({
  mixins: [PureRenderMixin],

  render: function() {
    return <div className={this.props.className}>foo</div>;
  }
});
```

但由于官方不打算在 ES6+ 中继续推行 `Mixins`，若还是希望使用，可以参考看看[第三方套件](https://www.npmjs.com/package/es6-class-mixin)或是[这个文件的用法](https://gist.github.com/sebmarkbage/ef0bf1f338a7182b6775)。

## 10. Default Parameters
以前 ES5 我们函数要使用预设值需要这样使用：

```js
var link = function (height, color) {
    var height = height || 50;
    var color = color || 'red';
}
```

现在 ES6+ 的函数可以支援预设值，让代码更为简洁：

```js
var link = function(height = 50, color = 'red') {
  ...
}
```

## 总结
以上就是 React ES5、ES6+常见用法对照表，能看到这边的你应该已经对于 React ES5、ES6 使用上有些认识，先给自己一些掌声吧！确实从 ES6 开始，JavaScript 和以前我们看到的 JavaScript 有些不同，增加了许多新的特性，有些读者甚至会很怀疑说这真的是 JavaScript 吗？ES6 的用法对于初学者来说可能会需要写一点时间吸收，下一章我们将进到同样也是有革新性设计和有趣的 React Native，用 JavaScript 和 React 写 Native App！

## 延伸阅读
1. [React/React Native 的ES5 ES6写法对照表](http://bbs.reactnative.cn/topic/15/react-react-native-%E7%9A%84es5-es6%E5%86%99%E6%B3%95%E5%AF%B9%E7%85%A7%E8%A1%A8)
2. [React on ES6+](https://babeljs.io/blog/2015/06/07/react-on-es6-plus)
3. [react native 中es6语法解析](http://www.ghugo.com/react-native-es6/)
4. [Learn ES2015](https://babeljs.io/docs/learn-es2015/)
5. [ECMAScript 6入门](http://es6.ruanyifeng.com/)
6. [React官方网站](https://facebook.github.io/react/index.html)
7. [React INTRO TO REACT.JS](http://fraserxu.me/intro-to-react/)
8. [React.createClass versus extends React.Component](https://toddmotto.com/react-create-class-versus-component/)
9. [react-native-coding-style](https://github.com/lzbSun/react-native-coding-style)
10. [Airbnb React/JSX Style Guide](https://github.com/airbnb/javascript/tree/master/react)
11. [ECMAScript 6入门](http://es6.ruanyifeng.com/)

## :door: 任意门
| [回首页](https://github.com/kdchang/reactjs101) | [上一章：用 React + Redux + Node（Isomorphic JavaScript）开发食谱分享网站](https://github.com/aclk/reactjs101/blob/master/Ch10/react-router-redux-node-isomorphic-javascript-open-cook.md) | [下一章：用 React Native + Firebase 开发跨平台行动应用程序](https://github.com/aclk/reactjs101/blob/master/Appendix02/README.md) |

| [勘误、提问或许愿](https://github.com/aclk/reactjs101/issues) |
