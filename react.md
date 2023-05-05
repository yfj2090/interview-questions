# react

## 如何理解 react 组件状态？

什么是状态？
**web 是一个状态机**

1. 对 react 中，useState / this.setState 函数，意味着状态发生改变；
1. 源码里，有一个 createUpdate 的函数要执行
1. shared.pending = update；
1. processUpdateQueue 中去消费这个 update
1. react 是依靠状态进行视图更新的，状态的变化，意味着视图迭代到了下一个状态
1. 在开发工程中，我们的思路只需要围绕着，在一些事件发生以后，当期界面的状态是什么样子。

对于 react 来说，相比较 vue，状态的概念更加重要，我们可以把状态理解成为一种“快照”。

```vue
<template>
  <div>{{ msg }}</div>
</template>
<script>
export default {
  data() {
    return {
      msg: 'hello'
    }
  },
  methods: {
    handleClick: () {
      this.msg = 'luyi'
    }
  }
}
// get: Dep.target & dep.add(Dep.target)
// set: dep.notify()
</script>
```

react 修改数据的方法：

```js
this.setState({});
const [state, dispatch] = useState(initState);
dispatch(action);
forceUpdate();
ReactDOM.render();
```

### 状态是同步还是异步？

V17 版本

在通常情况下，React 17 的 lagecy 模式，或者更低版本，同步和异步，要看调用的方式。
如果 setState 在当前的任务中执行，则是异步的（batchedUpdate）
如果 使用了异步任务（定时器、事件、网络请求类似的）去执行 state 更新，则是同步的。
因为：**isBatchingEventUpdate 以及变成了 false**。

在 18 中，都是异步的。

### 如何在异步情况，批量更新？

```js
setTimeout(() => {
  unstable_batchedUpdates(() => {
    this.setState();
    this.setState();
    this.setState();
  });
});
```

### 如果我想让某一个更新，优先级高一些。

```js
setTimeout(() => {
  this.setState();
});

ReactDOM.flushSync(() => {
  this.setState();
});
```

```js
class Index extends React.Component = () => {
  state = { count: 1 };

  handleClick = () => {
    // ...this.setState
    // todo ...
    setTimeout(() => {
      this.setState()
      this.setState()
      this.setState()
    })
    // todo ...
  }
}
batchedUpdate()

// 内部：
batchedUpdate(handleClick);

isBatchingEventUpdate = true;

setTimeout(() => {
  this.setState()
  this.setState()
  this.setState()
})
```

### react 组件的生命周期以及各阶段区别是什么？

constructor

- 初始化 state
- 绑定 this，如果我们有一些防抖、节流，可以在这里做。
- 劫持生命周期

getDerivedStateFromProps[state]

- 代替 componentWillReceiveProps
- 返回值，与 state 合并完，作为 shouldComponentUpdate 的第二个参数 newState

```js
static getDerivedStateFromProps(newProps) {
  const { type } = newProps;
  swtich(type) {
    case  'age':
        return { name: '年龄', defaultValue: 18 },
    case 'sex':
        return { name: '性别', defaultValue: '男' }
  }
}
```

### 为什么你是一个静态方法？

- 希望这个方法，是一个纯函数

componentWillReceiveProps

- 可以监听父组件是否执行 render
- props 改变，判断是否执行更新 state

componentWillUpdate

- 获取组件更新前的状态。

getSnapshotBeforeUpdate

- 配合 componentDidUpdate 一起，计算形成一个 snapShot 传给 componentDidUpdate

```js
getSnapshotBeforeUpdate(prevProps, prevState) {
  const snapshot = {
    // 一些组件的位置信息
  }
  return snapsot;
}

componentDidUpdate(prevProps, prevState, snapshot) {

}
```

componentDidUpdate

- 更新完以后调用的，切记如果 setState 要注意，否则很容易死循环
- 配合 getSnapshotBeforeUpdate

componentDidMount

- 可以做一些 DOM 的操作，比如 DOM 的事件监听
- 请求接口

shouldComponentUpdate

- 性能优化，看返回值，是否更新组件。

### react 的 hooks 是怎么模拟生命周期的？

```js
import { useEffect } from "react";

const MockListCycle = (props) => {
  useEffect(() => {
    console.log("mock: componentDidMount");
    return () => {
      console.log("mock: componentWillUnmount");
    };
  }, []);
  useEffect(() => {
    console.log("mock: componentDidUpdate");
  }, [props]);
};
```

## react 的事件创建以及合成事件方法

1. 初始化注册

在底层有这样的两个对象。

```js
const registrationNameModules = {
  onClick: SimpleEventPlugin,
  onChange: ChangeEventPlugin,
  onBlur: SimpleEventPlugin,

  // ...
};

const registrationNameDependencies = {
  onClick: ["click"],
  onChange： ['blur', 'change', 'click'...]
};
```

2. 事件收集，注册事件
   当我发现有一个 onChange 事件，他会绑定 ['blur', 'change', 'click', 'focus', 'keydown', 'keyup']

   deps = registrationNameDdependencies['onChange'];
   deps.forEach(event => {
    <!-- 绑定原生的事件监听器 -->

   })

3. 事件触发

```js
export default function Index() {
  const handle1 = () => console.log(1);
  const handle2 = () => console.log(2);
  const handle3 = () => console.log(3);
  const handle4 = () => console.log(4);

  return (
    <div onClick={handle3} onClickCapture={handle4}>
      <button onClick={handle1} onClickCapture={handle2}>
        click
      </button>
    </div>
  );
}
[handle4, handle2, handle1, handle3];
// 4,2,1,3
```

### react 为什么要做一套自己的事件系统？

- 兼容性，不同浏览器，兼容性不一致
- 如果说大量的 dom，绑定特别多的事件，GC 会有问题。性能好一些。
- 这个系统，多 SSR 和跨端很友好。

### 为什么 V17 版本之后，事件会放在 app 上，而不是 document 上？

- document 上，在一些微前端，是不是就不好处理。

## 什么事 HOC？以及高阶组件的传参和事件

高阶组件。
高阶函数就是一个将函数作为参数并且返回值也是函数的函数。
高阶函数是以组件作为参数，返回组件的函数。返回的组件把传进去的组件进行功能强化。

### HOC 的分类与应用场景有哪些？

1. 属性代理

```js
function HOC(Component) {
  return class Advanced extends React.Component {
    render() {
      return <Component {...this.props} />;
    }
  };
}
```

优点：

- 组件松耦合，又能对 props，组件能力进行增加。withRouter()
- 隔离一些业务组件的渲染
- 可以嵌套

缺点：

- 一般无法获取原始组件的状态，ref。
- 无法直接继承静态属性

1. 反向继承

```js
class Index extends React.Component {
  render() {
    return <div>hello luyi</div>;
  }
}

function HOC(Component) {
  return class warpComponent extends Component {};
}
```

优点：

- 方便获取组件内部的状态：
- 静态属性无需特殊处理

缺点：

- hook 不能用
- 耦合度很高，cdm 多个的时候。

## react 的 hook 理解，他在做什么？

workInProgress ->
memoizedState [next]

- menoizedState = 0
- memoizedState = "你好"
- memoizedState = { current: null }
- memoizedState = { create () => console.log() }

### Hooks 出现本质上原因是？

- 让函数组件也能做类组件的事，可以处理一些副作用，能拿 ref
- 解决逻辑复用难的问题
- OOP 放弃，拥抱 FP

### 为什么 useAPI 前面不能加判断

```js
function A() {
  const [data, setData] = useState(null);
}
```

## 为什么 useState 要使用数组而不是对象？

const { state, setState } = useState()

结构赋值的问题：
数组，依次排序的，所以可以随便命名。
对象不行，没有次序。

## useEffect 和 useLayoutEffect 的区别

修改 DOM，改变布局就用 useLayoutEffect，其他情况就用 useEffect。

### React.useEffect 回调函数和 componentDidMount / componentDidUpdate 执行时机有什么区别？

useEffect 在 react 的执行栈中，是异步执行的，而 cdm / cdu 是同步执行的。
useEffect 代码不会阻塞浏览器绘制，和 cdm / cdu 的执行时机更像 { 都是在 commitLayoutEffects 中执行 }

### 你听说过 useInsertionEffect 吗？

useInsertionEffect 执行时间，更加提前一些。
css-in-js

useInsertionEffect 的执行在 dom 更新之前，所以此时使用 css-in-js，避免浏览器重绘和重排，性能更好。

## redux 遵循的三原则

1. 单向数据流
   newState = reducer(staet, actions)
2. state 只读
   唯一 state 的方法，只有 dispatch 一个 action
3. 纯函数执行。
   每一个 reducer 都是纯函数

## redux 中的发布订阅和中间件思想。

state = {};

compose 的实现

```js
const compose = (...funcs) => {
  return funcs.reduce((f, g) => (x) => f(g(x)));
};
```

- redux 是一种发布订阅的核心实现方式，以 store 为数据中心，使用 dispatch 修改数据。使用 subscribe 订阅数据。dispatch 时，会通知所有的 subscribe 的函数。
- redux 把副作用交给 compose 以中间件的形式去处理
  - 核心就是强化 dispatch
  - 还有就是处理副作用

### React-Redux， Redux， React 三者关系

react 是一个 UI 框架
redux 是一个数据管理工具
React-Redux，基于 redux（作为一个数据管理工具，是无法触发视图更新的），react-redux = redux + consumer/Provider

### 如何理解单一事件来源


vue 和 react 的生命周期做一个比较吗？
1. 没什么关系，如果非要比的话
2. vue，主要多了一些 create 相关的声明周期，作用是 Vue 初始化干得事情比较多。
3. react，有很多中间过程处理的事件，原因是 运行时框架，中间过程需要有大量的钩子，也需要一些性能优化的手段。