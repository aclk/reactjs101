# 附录三、React 测试入门教学

![React 测试入门教学](./images/mocha.png)

## 前言
测试是软体开发中非常重要的一个环节，本章我们将带领大家从编写最简单的测试代码到整合 `Mocha` + `Chai` 官方提供的[测试工具](https://facebook.github.io/react/docs/test-utils.html)和 Airbnb 所设计的 [Enzyme](https://github.com/airbnb/enzyme) 进行 React 测试。

## Mocha 测试初体验
[Mocha](https://mochajs.org/) 是目前颇为流行的 JavaScript 测试框架之一，其可以很方便使用于浏览器端和 Node 环境。

>Mocha is a feature-rich JavaScript test framework running on Node.js and in the browser, making asynchronous testing simple and fun. Mocha tests run serially, allowing for flexible and accurate reporting, while mapping uncaught exceptions to the correct test cases.

除了 Mocha 外，尚有许多 JavaScript 单元测试工具可以选择，例如：[Jasmine](http://jasmine.github.io/)、[Karma](http://karma-runner.github.io/1.0/index.html) 等。但本章我们主要使用 `Mocha` + `Chai` 结合 React 官方测试工具和 Enzyme 进行讲解。

在这边我们先介绍一些比较常用的 Mocha 使用方法，让大家熟悉测试的用法（若是已经熟悉编写测试代码的读者这部份可以跳过）：

1. 安装环境与包

    安装 `react` 和 `react-dom`

    ```
    $ npm install --save react react-dom
    ```

    可以在全局安装 mocha：  

    ```
    $ npm install --global mocha
    ```

    也可以在开发环境下本地端安装（同时安装了 babel、eslint、webpack 等相关包，其中以 mocha、chai、babel 为主要必须）：

    ```
    $ npm install --save-dev babel-core babel-loader babel-eslint babel-preset-react babel-preset-es2015 eslint eslint-config-airbnb eslint-loader eslint-plugin-import eslint-plugin-jsx-a11y eslint-plugin-react webpack webpack-dev-server html-webpack-plugin chai mocha
    ```

2. 测试代码
    1. describe（test suite）：表示一组相关的测试。`describe` 为一个函数，第一个参数为 `test suite`的名称，第二个参数为实际执行的函数。
    2. it（test case）：表示一个单独测试，为测试里最小单位。`it` 为一个函数，第一个参数为 `test case` 的描述名称，第二个参数为实际执行的函数。

    在测试代码中会包含一个或多个 `test suite`，而每个 `test suite` 则会包含一个或多个 `test case`。

3. 整合 assertion 函数库 `Chai`

    所谓的 assertion（断言），就是判断代码的执行成果是否和预期一样，若是不一致则会发生错误。通常一个 test case 会拥有一个或多个 assertion。由于 Mocha 本身是一个测试框架，但不包含 assertion，所以我们使用 [Chai](http://chaijs.com/) 这个适用于浏览器端和 Node 端的 BDD / TDD assertion library。在 Chai 中共提供三种操作 assertion 介面风格：Expect、Assert、Should，在这边我们选择使用比较接近自然语言的 Expect。

    基本上，expect assertion 的写法都是类似：开头为 `expect` 方法 + `to` 或 `to.be` + 结尾 assertion 方法（例如：equal、a/an、ok、match）

4. Mocha 基本用法

    mocha 若没指定要执行哪个文件，预设会执行 `test` 文件夹下第一层的测试代码。若要让 `test` 文件夹中的子文件夹测试码也执行则要加上 `--recursive` 参数。 

    包含子文件夹：

    ```
    $ mocha --recursive
    ```

    指定一个文件

    ```
    $ mocha file1.js 
    ```

    也可以指定多个文件

    ```
    $ mocha file1.js file2.js
    ```

    现在，我们来编写一个简单的测试程序，亲身感受一下测试的感觉。以下是 `react-mocha-test-example/src/modules/add.js`，一个加法的函数：

    ```javascript
    const add = (x, y) => (
      x + y
    );

    export default add;
    ```

    接著我们编写测试这个函数的代码，测试是否正确。以下是 `react-mocha-test-example/src/test/add.test.js`：

    ```
    // test add.js
    import add from '../src/modules/add';
    import { expect } from 'chai';

    // describe is test suite, it is test case
    describe('test add function', () => (
      it('1 + 1 = 2', () => (
        expect(add(1, 1)).to.be.equal(2)
      ))
    ));
    ```

    在开始执行 `mocha` 后由于我们使用了，ES6 的语法所以必须使用 bable 进行转译，否则会出现类似以下的错误：

    ```
    import add from '../src/modules/add';
    ^^^^^^
    ```

    我们先做好配置 `.bablerc`，我们在之前已经有安装 `babel` 相关包和 `presets` 所以就会将 ES2015 语法转译。

    ```
    {
        "presets": [
          "es2015",
          "react",
         ],
        "plugins": []
    }
    ```

    此时，我们更改 `package.json` 中的 `scripts`，这样方便每次测试执行：

    若是使用本地端：

    ```
    $ ./node_modules/mocha/bin/mocha --compilers js:babel-core/register
    ```

    若是使用全局：

    ```
    $ mocha --compilers js:babel-core/register
    ```

    若是一切顺利，我们就可以看到执行测试成功的结果：

    ```
    $ mocha add.test.js

      test add function
        ✓ 1 + 1 = 2


      1 passing (181ms)
    ```

5. Mocha 指令参数

    在 Mocha 中有许多可以使用的好用参数，例如：`--recursive` 可以执行执行测试文件夹下的子文件夹代码、`--reporter 格式` 更改测试报告格式（预设是 `spec`，也可以更改为 `tap`）、`--watch` 用来监控测试代码，当有测试代码更新就会重新执行、`--grep` 撷取符合条件的 test case。

    以上这些参数我们可以都整理在 `test` 文件夹下的 `mocha.opts` 文件中当作设定数据，此时再次执行 `npm run test` 就会把参数也使用进去。

    ```
    --watch
    --reporter spec
    ```

6. 非同步测试

    在上面我们讨论的主要是同步的状况，但实际上在开发应用时往往会遇到非同步的情形。而在 Mocha 中每个 test case 最多允许执行 2000 毫秒，当时间超过就会显示错误。为了解决这个问题我们可以在 `package.json` 中更改：`"test": "mocha -t 5000 --compilers js:babel-core/register"` 文件。

    为了模拟测试非同步的情境，所以我们必须先安装 [axios](https://github.com/mzabriskie/axios)。

    ```
    $ npm install --save axios
    ```

    以下是 `react-mocha-test-example/src/test/async.test.js`：

    ```javascript
    import axios from 'axios';
    import { expect } from 'chai';

    it('asynchronous return an object', function(done){
      axios
        .get('https://api.github.com/users/torvus')
        .then(function (response) {
          expect(response).to.be.an('object');
          done();
        })
        .catch(function (error) {
          console.log(error);
        });
    });
    ```

    由于测试环境是在 Node 中，所以我们必须先安装 [node-fetch](https://github.com/bitinn/node-fetch) 来展现 promise 的情境。

    ```
    $ npm install --save node-fetch 
    ```

    以下是 `react-mocha-test-example/src/test/promise.test.js`：

    ```javascript
    import fetch from 'node-fetch';
    import { expect } from 'chai';

    it('asynchronous fetch promise', function() {
      return fetch('https://api.github.com/users/torvus')
        .then(function(response) { return response.json() })
        .then(function(json) { 
          expect(json).to.be.an('object');
        });
    });
    ```

7. 测试使用的 hook

    在 Mocha 中的 test suite 中，有 before()、after()、beforeEach() 和 afterEach() 四种 hook，可以让你设计在特定时间点执行测试。

    ```javascript
    describe('hooks', function() {
      before(function() {
        // 在 before 中的 test case 会在所有 test cases 前执行
      });
      after(function() {
        // 在 after 中的 test case 会在所有 test cases 后执行
      });
      beforeEach(function() {
        // 在 beforeEach 中的 test case 会在每个 test cases 前执行
      });
      afterEach(function() {
        // 在 afterEach 中的 test case 会在每个 test cases 后执行
      });
      // test cases
    });
    ```

## 动手实践
在上面我们已经先讲解了 `Mocha` + `Chai` 测试工具和基础的测试写法。现在接著我们要来探讨 React 中的测试用法。然而，要在 React 中测试 Component 以及 JSX 语法时，使用传统的测试工具并不方便，所以要整合 `Mocha` + `Chai` 官方提供的[测试工具](https://facebook.github.io/react/docs/test-utils.html)和 Airbnb 所设计的 [Enzyme](https://github.com/airbnb/enzyme)（由于官方的测试工具使用起来不太方便所以有第三方针对其进行封装）进行测试。

### 使用官方测试工具
我们知道在 React 一个重要的特色为 Virtual DOM 所以在官方的测试工具中有提供测试 Virtual DOM 的方法：Shallow Rendering（createRenderer），以及测试真实 DOM 的方法：DOM Rendering（renderIntoDocument）。

1. Shallow Rendering（createRenderer）

    Shallow Rendering 系指将一个 Virtual DOM 渲染成子 Component，但是只渲染第一层，不渲染所有子组件，因此处理速度快且不需要 DOM 环境。Shallow rendering 在单元测试非常有用，由于只测试一个特定的 component，而重要的不是它的 children。这也意味着改变一个 child component 不会影响 parent component 的测试。

    以下是 `react-addons-test-utils-example/src/test/shallowRender.test.js`：

    ```javascript
    import React from 'react';
    import TestUtils from 'react-addons-test-utils';
    import { expect } from 'chai';
    import Main from '../src/components/Main';

    function shallowRender(Component) {
      const renderer = TestUtils.createRenderer();
      renderer.render(<Component/>);
      return renderer.getRenderOutput();
    }

    describe('Shallow Rendering', function () {
      it('Main title should be h1', function () {
        const todoItem = shallowRender(Main);
        expect(todoItem.props.children[0].type).to.equal('h1');
        expect(todoItem.props.children[0].props.children).to.equal('Todos');
      });
    });
    ```

    以下是 `react-addons-test-utils-example/src/test/shallowRenderProps.test.js`：    

    ```javascript
    import React from 'react';
    import TestUtils from 'react-addons-test-utils';
    import { expect } from 'chai';
    import TodoList from '../src/components/TodoList';

    const shallowRender = (Component, props) => {
      const renderer = TestUtils.createRenderer();
      renderer.render(<Component {...props}/>);
      return renderer.getRenderOutput();
    }

    describe('Shallow Props Rendering', () => {
      it('TodoList props check', () => {
        const todos = [{ id: 0, text: 'reading'}, { id: 1, text: 'coding'}];
        const todoList = shallowRender(TodoList, {todos: todos});
        expect(todoList.props.children.type).to.equal('ul');
        expect(todoList.props.children.props.children[0].props.children).to.equal('reading');
        expect(todoList.props.children.props.children[1].props.children).to.equal('coding');
      });
    });
    ```

2. DOM Rendering（renderIntoDocument）
    
    注意，因为 Mocha 运行在 Node 环境中，所以你不会存取到 DOM。所以我们要使用 JSDOM 来模拟真实 DOM 环境。同时我在这边引入 `react-dom`，这样我们就可以使用 findDOMNode 来选取元素。事实上，findDOMNode 方法的最大优势是提供比 TestUtils 更好的 CSS 选择器，方便开发者选择元素。

    以下是 `react-addons-test-utils-example/src/test/setup.test.js`：    

    ```javascript
    import jsdom from 'jsdom';

    if (typeof document === 'undefined') {
      global.document = jsdom.jsdom('<!doctype html><html><head></head><body></body></html>');
      global.window = document.defaultView;
      global.navigator = global.window.navigator;
    }
    ```

    以下是 `react-addons-test-utils-example/src/components/TodoHeader/TodoHeader.js`：    

    ```javascript
    import React from 'react';

    class TodoHeader extends React.Component {
      constructor(props) {
        super(props);
        this.toggleButton = this.toggleButton.bind(this);
        this.state = {
          isActivated: false,
        };
      }
      toggleButton() {
        this.setState({
          isActivated: !this.state.isActivated,      
        })
      }
      render() {
        return (
          <div>
            <button disabled={this.state.isActivated} onClick={this.toggleButton}>Add</button>
          </div>
        );
      };
    }

    export default TodoHeader;
    ```

    需要留意的是若是 stateless components 使用 TestUtils.renderIntoDocument，要将 renderIntoDocument 包在 `<div></div>` 内，使用 `findDOMNode(TodoHeaderApp).children[0]` 取得，不然会回传 null。更进一步细节可以[参考这里](https://github.com/facebook/react/issues/4839)。不过由于我们是使用 `class-based` Component 所以不会遇到这个问题。

    以下是 `react-addons-test-utils-example/src/test/renderIntoDocument.test.js`：    

    ```javascript
    import React from 'react';
    import TestUtils from 'react-addons-test-utils';
    import { expect } from 'chai';
    import { findDOMNode } from 'react-dom';
    import TodoHeader from '../src/components/TodoHeader';

    describe('Simulate Event', function () {
      it('When click the button, it will be toggle', function () {
        const TodoHeaderApp = TestUtils.renderIntoDocument(<TodoHeader />);
        const TodoHeaderDOM = findDOMNode(TodoHeaderApp);
        const button = TodoHeaderDOM.querySelector('button');
        TestUtils.Simulate.click(button);
        let todoHeaderButtonAfterClick = TodoHeaderDOM.querySelector('button').disabled;
        expect(todoHeaderButtonAfterClick).to.equal(true);
      });
    });
    ```

    这种渲染 DOM 的测试方式类似于 JavaScript 或 jQuery 的 DOM 操作。首先要先找到要想操作的目标节点，而后触发想要执行的动作，在官方测试工具中拥有许多可以[协助选取节点的方法](https://facebook.github.io/react/docs/test-utils.html#scryrenderedcomponentswithtype)。然而由于其在使用上不够简洁，也因此我们接下来将介绍由 Airbnb 所设计的 [Enzyme](https://github.com/airbnb/enzyme)进行 React 测试。

### 使用 Enzyme 函数库进行测试
[Enzyme](https://github.com/airbnb/enzyme) 优势是在于针对官方测试工具封装成了类似 jQuery API 的选取元素的方式。根据官方网站介绍 Enzyme 将更容易地去操作选取 React Component：

> Enzyme is a JavaScript Testing utility for React that makes it easier to assert, manipulate, and traverse your React Components’ output.
Enzyme is unopinionated regarding which test runner or assertion library you use, and should be compatible with all major test runners and assertion libraries out there.

在 Enzyme 中选取元素使用 `find()`：

```javascript
component.find('.className'); // 使用 class 选取
component.find('#idName'); // 使用 id 选取
component.find('h1'); // 使用元素选取
```

接下来我们介绍 Enzyme 三个主要的 API 方法：

1. Shallow Rendering

    shallow 方法事实上就是官方测试工具的 shallow rendering 封装。同样是只渲染第一层，不渲染所有子组件。

    ```
    import React from 'react';
    import TestUtils from 'react-addons-test-utils';
    import { expect } from 'chai';
    import { shallow } from 'enzyme';
    import Main from '../../src/components/Main';

    describe('Enzyme Shallow Rendering', () => {
      it('Main title should be Todos', () => {
        const main = shallow(<Main />);
        // 判断 h1 文字是否如预期
        expect(main.find('h1').text()).to.equal('Todos');
      });
    });
    ```

2. Static Rendering

    render 方法是将 React 组件渲染成静态的 HTML 字串，并利用 Cheerio 函数库（这点和 shallow 不同）分析其结构返回对象。虽然底层是不同的处理引擎但使用上 API 封装起来和 Shallow 却是一致的。需要注意的是 Static Rendering 非只渲染一层，需要注意是否需要 mock props 传递。

    ```javascript
    import React from 'react';
    import TestUtils from 'react-addons-test-utils';
    import { expect } from 'chai';
    import { render } from 'enzyme';
    import Main from '../../src/components/Main';

    describe('Enzyme Staic Rendering', () => {
      it('Main title should be Todos', () => {
        const todos = [{ id: 0, text: 'reading'}, { id: 1, text: 'coding'}];
        const main = render(<Main todos={todos} />);
        expect(main.find('h1').text()).to.equal('Todos');
      });
    });
    ```

3. Full Rendering

    mount 方法 React 组件载入真实 DOM 节点。同样因为牵涉到 DOM 也要使用 JSDOM。

    ```javascript
    import React from 'react';
    import TestUtils from 'react-addons-test-utils';
    import { expect } from 'chai';
    import { findDOMNode } from 'react-dom';
    import { mount } from 'enzyme';
    import TodoHeader from '../../src/components/TodoHeader';

    describe('Enzyme Mount', () => {
      it('Click Button', () => {
        let todoHeaderDOM = mount(<TodoHeader />);
        // 取得 button 并模拟 click
        let button = todoHeaderDOM.find('button').at(0);
        button.simulate('click');
        // 检查 prop(key) 是否正确
        expect(button.prop('disabled')).to.equal(true);
      });
    });
    ```    

最后我们可以在 `react-addons-test-utils-example` 文件夹下执行：

```
$ npm test
```

若一切顺利就可以看到测试通过的消息！

```

  Enzyme Mount
    ✓ Click Button (44ms)

  Enzyme Shallow Rendering
    ✓ Main title should be Todos

  Enzyme Staic Rendering
    ✓ Main title should be Todos

  Simulate Event
    ✓ When click the button, it will be toggle

  Shallow Rendering
    ✓ Main title should be h1

  Shallow Props Rendering
    ✓ TodoList props check


  6 passing (279ms)

```

事实上 Enzyme 还提供更多的 API 可以使用，若是读者想了解更多 Enzyme API 可以 [参考官方文件](http://airbnb.io/enzyme/docs/api/index.html)。

## 总结
以上我们从 `Mocha` + `Chai` 的使用方式介绍到 React 官方提供的[测试工具](https://facebook.github.io/react/docs/test-utils.html) 和 Airbnb 所设计的 [Enzyme](https://github.com/airbnb/enzyme)，相信读者对于测试代码已经有初步的了解，若尚未掌握的读者不妨跟著上面的范例再重新走过一遍，接著我们要进到最后的 `GraphQL/Relay`的介绍。

## 延伸阅读
1. [React 测试入门教程](http://www.ruanyifeng.com/blog/2016/02/react-testing-tutorial.html)
2. [测试框架 Mocha 实例教程](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)
3. [Test Utilities](https://facebook.github.io/react/docs/test-utils.html)
4. [JavaScript Testing utilities for React](https://github.com/airbnb/enzyme)
5. [持续集成是什么？](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)
6. [Let’s test React components with TDD, Mocha, Chai, and jsdom](https://medium.freecodecamp.com/simple-react-testing-d9e25ec87e2)
7. [Unit Testing React-Native Components with Enzyme Part 1](https://kyrisu.com/2016/01/31/unit-testing-react-native-components-with-enzyme-part-1/)
8. [What React Stateless Components Are Missing](http://jaketrent.com/post/react-stateless-components-missing/)
9. [0.14-rc1: findDOMNode(statelessComponent) doesn’t work with TestUtils.renderIntoDocument #4839](https://github.com/facebook/react/issues/4839)
10. [Writing Redux Tests](http://redux.js.org/docs/recipes/WritingTests.html)
11. [【译】展望2016，React.js 最佳实践 (中英对照版)](http://blog.jimmylv.info/2016-01-22-React.js-Best-Practices-for-2016/)

（image via [Anthony Ng](https://cdn-images-1.medium.com/max/800/1*CrB6isZN6YXeM1rWmnjxHw.png)）

## :door: 任意门
| [回首页](https://github.com/aclk/reactjs101) | [上一章：附录二、用 React Native + Firebase 开发跨平台行动应用程序](https://github.com/aclk/reactjs101/blob/master/Appendix02/README.md) | [下一章：附录四、GraphQL/Relay 初体验](https://github.com/aclk/reactjs101/blob/master/Appendix04/README.md) |

| [纠错、提问或想法](https://github.com/aclk/reactjs101/issues) |
