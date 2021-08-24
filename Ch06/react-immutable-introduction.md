# ImmutableJS 入门教学

![ImmutableJS](./images/immutable.png "ImmutableJS")

## 前言
一般来说在 JavaScript 中有两种数据类型：Primitive（String、Number、Boolean、null、undefinded）和 Object（Reference）。在 JavaScript 中对象的操作比起 Java 容易很多，但也因为相对弹性不严谨，所以产生了一些问题。在 JavaScript 中的 Object（对象）数据是 Mutable（可以变的），由于是使用 Reference 的方式，所以当修改到复制的值也会修改到原始值。例如下面的 `map2` 值是指到 `map1`，所以当 `map1` 值一改，`map2` 的值也会受影响。

```javascript
var map1 = { a: 1 };
var map2 = map1;
map2.a = 2
```

通常一般做法是使用 `deepCopy` 来避免修改，但这样做法会产生较多的资源浪费。为了很好的解决这个问题，我们可以使用 `Immutable Data`，所谓的 Immutable Data 就是一旦建立，就不能再被修改的数据数据。

为了解决这个问题，在 2013 年时 Facebook 工程师 Lee Byron 打造了 [ImmutableJS](https://facebook.github.io/immutable-js/)，但并没有被预设放到 React 工具包中（虽然有提供简化的 Helper），但 `ImmutableJS` 的出现确实解决了 `React` 甚至 `Redux` 所遇到的一些问题。

以下范例即是引入了 `ImmutableJS` 的效果，读者可以发现，虽然我们操作了 `map1` 的值，但会发现原本的 `map1` 并未受到影响（因为任何修改都不会影响到原始数据），虽然使用 `deepCopy` 也可以模拟类似的效果但会浪费过多的计算资源和内存，`ImmutableJS` 则可以容易地共享没有被修改到的数据（例如下面的数据 `b` 即为 `map1` 所 `map2` 共享），因而有更好的性能表现。

```javascript
import Immutable from 'immutable';

var map1 = Immutable.Map({ a: 1, b: 3 });
var map2 = map1.set('a', 2);

map1.get('a'); // 1
map2.get('a'); // 2
```

## ImmutableJS 特性介绍
ImmutableJS 提供了 7 种不可修改的数据类型：`List`、`Map`、`Stack`、`OrderedMap`、`Set`、`OrderedSet`、`Record`。若是对 Immutable 对象操作都会回传一个新值。其中比较常用的有 `List`、`Map` 和 `Set`：

1. Map：类似于 key/value 的 object，在 ES6 也有原生 `Map` 对应

  ```javascript
  const Map= Immutable.Map;

  // 1. Map 大小
  const map1 = Map({ a: 1 });
  map1.size
  // => 1

  // 2. 新增或取代 Map 元素
  // set(key: K, value: V)
  const map2 = map1.set('a', 7);
  // => Map { "a": 7 }

  // 3. 删除元素
  // delete(key: K)
  const map3 = map1.delete('a');
  // => Map {}

  // 4. 清除 Map 内容
  const map4 = map1.clear();
  // => Map {}

  // 5. 更新 Map 元素
  // update(updater: (value: Map<K, V>) => Map<K, V>)
  // update(key: K, updater: (value: V) => V)
  // update(key: K, notSetValue: V, updater: (value: V) => V)
  const map5 = map1.update('a', () => (7))
  // => Map { "a": 7 }

  // 6. 合并 Map
  const map6 = Map({ b: 3 });
  map1.merge(map6);
  // => Map { "a": 1, "b": 3 }
  ```

2. List：有序且可以重复值，对应于一般的 Array

  ```javascript
  const List= Immutable.List;

  // 1. 取得 List 长度
  const arr1 = List([1, 2, 3]);
  arr1.size
  // => 3

  // 2. 新增或取代 List 元素内容
  // set(index: number, value: T)
  // 将 index 位置的元素替换
  const arr2 = arr1.set(-1, 7);
  // => [1, 2, 7]
  const arr3 = arr1.set(4, 0);
  // => [1, 2, 3, undefined, 0]

  // 3. 删除 List 元素
  // delete(index: number)
  // 删除 index 位置的元素
  const arr4 = arr1.delete(1);
  // => [1, 3]

  // 4. 插入元素到 List
  // insert(index: number, value: T)
  // 在 index 位置插入 value
  const arr5 = arr1.insert(1, 2);
  // => [1, 2, 2, 3]

  // 5. 清空 List
  // clear()
  const arr6 = arr1.clear();
  // => []
  ```

3. Set：没有顺序且不能重复的列表

  ```javascript
  const Set= Immutable.Set;

  // 1. 建立 Set
  const set1 = Set([1, 2, 3]);
  // => Set { 1, 2, 3 }

  // 2. 新增元素
  const set2 = set1.add(1).add(5);
  // => Set { 1, 2, 3, 5 }
  // 由于 Set 为不能重复集合，故 1 只能出现一次

  // 3. 删除元素
  const set3 = set1.delete(3);
  // => Set { 1, 2 }

  // 4. 取联集
  const set4 = Set([2, 3, 4, 5, 6]);
  set1.union(set4);
  // => Set { 1, 2, 3, 4, 5, 6 }

  // 5. 取交集
  set1.intersect(set4);
  // => Set { 2, 3 }

  // 6. 取差集
  set1.subtract(set4);
  // => Set { 1 }
  ```

## ImmutableJS 的特性整理
1. Persistent Data Structure
  在 `ImmutableJS` 的世界里，只要数据一被创建，就不能修改，维持 `Immutable`。就不会发生下列的状况：

  ```javascript
  var obj = {
   a: 1
  };

  funcationA(obj);
  console.log(obj.a) // 不确定结果为多少？
  ```

  使用 `ImmutableJS` 就没有这个问题：

  ```javascript
  // 有些开发者在使用时会在 ``Immutable` 变数前加 `$` 以示区隔。

  const $obj = fromJS({
   a: 1
  });

  funcationA($obj);
  console.log($obj.get('a')) // 1
  ```

2. Structural Sharing
  为了维持数据的不可变，又要避免像 `deepCopy` 一样复制所有的节点数据而造成的资源损耗，在 `ImmutableJS` 使用的是 Structural Sharing 特性，亦即如果对象树中一个节点发生变化的话，只会修改这个节点和和受它影响的父节点，其他节点则共享。

  ```javascript
  const obj = {
    count: 1,
    list: [1, 2, 3, 4, 5]
  }
  var map1 = Immutable.fromJS(obj);
  var map2 = map1.set('count', 4);

  console.log(map1.list === map2.list); // true
  ```

3. Support Lazy Operation

  ```javascript
  Immutable.Range(1, Infinity)
  .map(n => -n)
  // Error: Cannot perform this action with an infinite size.

  Immutable.Range(1, Infinity)
  .map(n => -n)
  .take(2)
  .reduce((r, n) => r + n, 0);
  // -3
  ```

4. 丰富的 API 并提供快速转换原生 JavaScript 的方式
  在 ImmutableJS 中可以使用 `fromJS()`、`toJS()` 进行 JavaScript 和 ImmutableJS 之间的转换。但由于在转换之间会非常耗费资源，所以若是你决定引入 `ImmutableJS` 的话请尽量维持数据处在 `Immutable` 的状态。

5. 支持 Functional Programming
  `Immutable` 本身就是 Functional Programming（函数式程式设计）的概念，所以在 `ImmutableJS` 中可以使用许多 Functional Programming 的方法，例如：`map`、`filter`、`groupBy`、`reduce`、`find`、`findIndex` 等。

6. 容易实现 Redo/Undo 历史回顾

## React 性能优化
`ImmutableJS` 除了可以和 `Flux/Redux` 整合外，也可以用于基本 react 性能优化。以下是一般使用性能优化的简单方式：

传统 JavaScript 比较方式，若数据型态为 Primitive 就不会有问题：

```javascript
// 在 shouldComponentUpdate 比较接下来的 props 一否一致，若相同则不重新渲染，提升性能
shouldComponentUpdate (nextProps) {
    return this.props.value !== nextProps.value;
}
```

但当比较的是对象的话就会出现问题：

```javascript
// 假设 this.props.value 为 { foo: 'app' }
// 架设 nextProps.value 为 { foo: 'app' },
// 虽然两者值是一样，但由于 reference 位置不同，所以视为不同。但由于值一样应该要避免重复渲染
this.props.value !== nextProps.value; // true
```

使用 `ImmutableJS`：

```javascript
var SomeRecord = Immutable.Record({ foo: null });
var x = new SomeRecord({ foo: 'app'  });
var y = x.set('foo', 'azz');
x === y; // false
```

在 ES6 中可以使用官方文件上的 `PureRenderMixin` 进行比较，可以让程式码更简洁：

```javascript
import PureRenderMixin from 'react-addons-pure-render-mixin';
class FooComponent extends React.Component {
  constructor(props) {
    super(props);
    this.shouldComponentUpdate = PureRenderMixin.shouldComponentUpdate.bind(this);
  }
  render() {
    return <div className={this.props.className}>foo</div>;
  }
}
```

## 总结
虽然 `ImmutableJS` 的引入可以带来许多好处和性能的提升但由于引入整体档案较大且较具侵入性，在引入之前可以自行评估看看是否合适于目前的专案。接下来我们将在后面的章节讲解如何将 `ImmutableJS` 和 `Redux` 整合应用到实务上的范例。

## 延伸阅读
1. [官方网站](https://facebook.github.io/immutable-js/)
2. [Immutable.js初识](http://www.w3cplus.com/javascript/immutable-js.html)
3. [Immutable 详解及 React 中实践](https://github.com/camsong/blog/issues/3)
4. [为什么需要Immutable.js](http://zhenhua-lee.github.io/react/Immutable.html)
5. [facebook immutable.js 意义何在，使用场景？](https://www.zhihu.com/question/28016223)
6. [React 巢状 Component 性能优化](https://blog.wuct.me/react-%E5%B7%A2%E7%8B%80-component-%E6%95%88%E8%83%BD%E5%84%AA%E5%8C%96-b01d8a0d3eff#.3kf4h1xq1)
7. [PureRenderMixin](https://facebook.github.io/react/docs/pure-render-mixin.html)
8. [seamless-immutable](https://github.com/rtfeldman/seamless-immutable)
9. [Immutable Data Structures and JavaScript](http://jlongster.com/Using-Immutable-Data-Structures-in-JavaScript)

（image via [risingstack](https://risingstack-blog.s3.amazonaws.com/2016/Jan/immutable_logo_for_react_js_best_practices-1453211749818.png)）

## :door: 任意门
| [回首页](https://github.com/aclk/reactjs101) | [上一章：React Router 入门实战教学](https://github.com/aclk/reactjs101/blob/master/Ch05/react-router-introduction.md) | [下一章：Flux 基础概念与实战入门](https://github.com/aclk/reactjs101/blob/master/Ch07/react-flux-introduction.md) |

| [纠错、提问或想法](https://github.com/aclk/reactjs101/issues) |
