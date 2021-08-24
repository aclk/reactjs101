# React Component 规范与生命周期（Life Cycle）

## 前言

经过前面的努力相信目前读者对于用 React 开发一些简单的组件（Component）已经有一定程度的掌握了，现在我们将更细部探讨 React Component 的规范和其生命周期。

## React Component 规范

若读者还有印象的话，我们前面介绍 React 特性时有描述 React 的主要编写方式有两种：一种是使用 ES6 Class，另外一种是 Stateless Components，使用 Functional Component 的写法，单纯渲染 UI。这边再帮大家复习一下上一个章节的简单范例：

1. 使用 ES6 的 Class（可以进行比较复杂的操作和组件生命周期的控制，相对于 stateless components 耗费资源）

  ```javascript
  //  注意组件开头第一个字母都要大写
  class MyComponent extends React.Component {
    // render 是 Class based 组件唯一必须的方法（method）
    render() {
      return (
        <div>Hello, {this.props.name}</div>
      );
    }
  }

  // PropTypes 验证，若传入的 props type 不符合将会显示错误
  MyComponent.propTypes = {
    name: React.PropTypes.string,
  }

  // Prop 预设值，若对应 props 没传入值将会使用 default 值，为每个实例化 Component 共用的值
  MyComponent.defaultProps = {
    name: '',
  }

  // 将 <MyComponent /> 组件插入 id 为 app 的 DOM 元素中
  ReactDOM.render(<MyComponent name="Mark"/>, document.getElementById('app'));
  ```

1. 使用 Functional Component 写法（单纯地 render UI 的 stateless components，没有内部状态、没有实际对象和 ref，没有生命周期函数。若非需要控制生命周期的话建议多使用 stateless components 获得比较好的性能）

  ```javascript
  // 使用 arrow function 来设计 Functional Component 让 UI 设计更单纯（f(D) => UI），减少副作用（side effect）
  const MyComponent = (props) => (
    <div>Hello, {props.name}</div>
  );

  // PropTypes 验证，若传入的 props type 不符合将会显示错误
  MyComponent.propTypes = {
    name: React.PropTypes.string,
  }

  // Prop 预设值，若对应 props 没传入值将会使用 default 值
  MyComponent.defaultProps = {
    name: '',
  }

  // 将 <MyComponent /> 组件插入 id 为 app 的 DOM 元素中
  ReactDOM.render(<MyComponent name="Mark"/>, document.getElementById('app'));
  ```

值得留意的是在 ES6 Class 中 `render()` 是唯一必要的方法（但要注意的是请保持 `redner()` 的纯粹，不要在里面进行 `state` 修改或是使用非同步方法和浏览器互动，若需非同步互动请于 `componentDidMount()` 操作），而 Functional Component 目前允许 `return null` 值。 喔对了，在 ES6 中也不支持 `mixins` 复用其他组件的方法了。

## React Component 生命周期

React Component，就像人会有生老病死一样有生命周期。一般而言 Component 有以下三种生命周期的状态：

1. Mounting：已插入真实的 DOM
2. Updating：正在被重新渲染
3. Unmounting：已移出真实的 DOM

针对 Component 的生命周期状态 React 也有提供对应的处理方法：

1. Mounting

- componentWillMount()
- componentDidMount()

1. Updating

- componentWillReceiveProps(object nextProps)：已载入组件收到新的参数时呼叫
- shouldComponentUpdate(object nextProps, object nextState)：组件判断是否重新渲染时呼叫，起始不会呼叫除非呼叫 forceUpdate()
- componentWillUpdate(object nextProps, object nextState)
- componentDidUpdate(object prevProps, object prevState)

1. Unmounting

- componentWillUnmount()

很多读者一开始学习 Component 生命周期时会觉得很抽象，所以接下来用一个简单范例让大家感受一下 Component 的生命周期。读者可以发现当一开始载入组件时第一个会触发 `console.log('constructor');`，依序执行 `componentWillMount`、`componentDidMount` ，而当点击文字触发 `handleClick()` 更新 `state` 时则会依序执行 `componentWillUpdate`、`componentDidUpdate`：

HTML Markup：

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <script src="https://fb.me/react-15.1.0.js"></script>
  <script src="https://fb.me/react-dom-15.1.0.js"></script>
  <title>Component LifeCycle</title>
</head>
<body>
  <div id="app"></div>
</body>
</html>
```

Component 生命周期展示：

```javascript
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    console.log('constructor');
    this.handleClick = this.handleClick.bind(this);
    this.state = {
      name: 'Mark',
    }
  }
  handleClick() {
    this.setState({'name': 'Zuck'});
  }
  componentWillMount() {
    console.log('componentWillMount');
  }
  componentDidMount() {
    console.log('componentDidMount');
  }
  componentWillReceiveProps() {
    console.log('componentWillReceiveProps');
  }
  componentWillUpdate() {
    console.log('componentWillUpdate');
  }
  componentDidUpdate() {
    console.log('componentDidUpdate');
  }
  componentWillUnmount() {
    console.log('componentWillUnmount');
  }
  render() {
    return (
      <div onClick={this.handleClick}>Hi, {this.state.name}</div>
    );
  }
}

ReactDOM.render(<MyComponent />, document.getElementById('app'));
```

<a class="jsbin-embed" href="http://jsbin.com/yokebo/embed?html,js,console,output">点击看详细范例</a><script src="http://static.jsbin.com/js/embed.min.js?3.39.12"></script>

![React Component 规格与生命周期](./images/react-lifecycle.png)

其中特殊处理的函数 `shouldComponentUpdate`，目前预设 `return true`。若你想要优化效能可以自己编写判断方式，若采用 `immutable` 可以使用 `nextProps === this.props` 比对是否有变动：

```javascript
shouldComponentUpdate(nextProps, nextState) {
  return nextProps.id !== this.props.id;
}
```

## Ajax 非同步处理

若有需要进行 Ajax 非同步处理，请在 `componentDidMount` 进行处理。以下通过 `jQuery` 执行 `Ajax` 取得 `Github API`　资料当做范例：

HTML Markup：

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <script src="https://fb.me/react-15.1.0.js"></script>
  <script src="https://fb.me/react-dom-15.1.0.js"></script>
  <script src="https://code.jquery.com/jquery-3.1.0.js"></script>
  <title>GitHub User</title>
</head>
<body>
  <div id="app"></div>
</body>
</html>
```

app.js

```javascript
class UserGithub extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
          username: '',
          githubtUrl: '',
          avatarUrl: '',
        }
    }
    componentDidMount() {
        $.get(this.props.source, (result) => {
            console.log(result);
            const data = result;
            if (data) {
              this.setState({
                    username: data.name,
                    githubtUrl: data.html_url,
                    avatarUrl: data.avatar_url
              });
            }
        });
    }
    render() {
        return (
          <div>
            <h3>{this.state.username}</h3>
            <img src={this.state.avatarUrl} />
            <a href={this.state.githubtUrl}>Github Link</a>.
          </div>
        );
    }
}

ReactDOM.render(
  <UserGithub source="https://api.github.com/users/torvalds" />,
  document.getElementById('app')
);
```

<a class="jsbin-embed" href="http://jsbin.com/kupusa/embed?html,js,output">点击看详细范例</a><script src="http://static.jsbin.com/js/embed.min.js?3.39.12"></script>

## 总结

以上介绍了 React Component 规范与生命周期（Life Cycle）的概念，其中生命周期的概念对于初学者来说可能会比较抽象，建议读者跟著范例动手实践。接下来我们将更进一步介绍 `React Router` 让读者感受一下单页式应用程序（single page application）的设计方式。

## 延伸阅读

1. [Component Specs and Lifecycle](https://facebook.github.io/react/docs/component-specs.html#lifecycle-methods)

（image via [react-lifecycle](http://imgh.us/react-lifecycle.svg)）

## :door: 任意门

| [回首页](https://github.com/aclk/reactjs101) | [上一章：Props、State、Refs 与表单处理](https://github.com/aclk/reactjs101/blob/master/Ch04/props-state-introduction.md) | [下一章：React Router 入门实战教学](https://github.com/aclk/reactjs101/blob/master/Ch05/react-router-introduction.md) |

| [纠错、提问或想法](https://github.com/aclk/reactjs101/issues) |
