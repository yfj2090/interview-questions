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

```js
class Index extends React.Component = () => {
  state = { count: 1 };

  handleClick = () => {

  }
}
batchedUpdate()

// 内部：
batchedUpdate(handleClick);

isBatchingEventUpdate = true;

this.setState()
this.setState()
this.setState()
```
