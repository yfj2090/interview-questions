框架一般问什么？
1-3 年：会熟练使用

- 生命周期，使用上的。
- 指令，use API

3-5 年：

- 思想
- 方案，封装，亮点
- 性能
- 源码

5-7 年：

- 框架无关，框架对比，核心思想（宿主环境）
- 源码，怎么影响你的工作

## 如果平时使用 react/vue，那么 vue/react 是否需要很深入？

react --

## vue 有哪些生命周期？以及各个生命周期做了些什么？

生命周期，就是我在一系列的流程中，在流程中间，插入一下代码片段，让它执行。
webpack plugin.

- beforeCreate
  - 组件的 options 都未被创建，el，data，methods data computed 都还不能用
- created
  - 实例已经完成创建了，watch，event，data 初始化都完成了，没有挂载，$el 无法获取
- beforeMount
  - 现在数据已经被劫持了，下面是渲染到界面上去了
- Mounted
- beforeUpdate
  - 已经是 nextTick 了。
- updated
  - 已经经历了一系列的 patch， diff， 调用 updated。
- beforeDestory

- destory
  - 在一些清理逻辑完成以后，父子关系，watcher，

如果说，你想展示你对源码比较了解。

## data 是一个函数的原因以及如何理解 vue 的模块化？

一起写一段代理。

```js
const data = { message: "hello world" };

const com1 = new Vue({
  el: "#app1",
  data,
});

const com2 = new Vue({
  el: "#app2",
  data,
});
```

## vue 的指令有哪些，如何书写自定义指令？

Vue 允许我们通过以全局注册和局部注册两种方式，添加自定义指令。

```js
// el 直接要绑定的元素
// binding，一个对象，包含：
// name
// value
// oldValue
// arg
// modifiers

Vue.directive("resize", {
  // 只调用一次，指令第一次绑定元素时调用，主要进行初始化
  bind(el) {},
  // 被绑定元素插入父节点的时候嗲用
  inserted() {},
  // 所在组件的 Vnode 更新时调用
  update() {},
  // 所在组件的 Vnode 更新后调用
  componentUpdated() {},
  // 只调用一次
  unbind() {},
});
```

```js
directive: {
  xxx: {
    bind() {},
  }
}
```

v-copy
v-loadmore
v-preload
v-lazyload

v-debounce
v-throttle

v-darggable

更偏向于，给元素、组件，做功能增强；
而不是去组合、加工，处理元素

## 组件间不同传参方式有何优劣？

有哪些？

1. props / $emit - 用于父子组件之间通信

- 优点
  - 简单，常见，props 有类型检查
- 缺点
  - 跨级上优缺点。

2. $ref / $children | $parent - 用于指向性通信

- 优点
  - 能够拿到斧子组件的实例的
- 缺点
  - 难以维护，你是打破了数据封装的。

3. $attrs / $listener - 隔代等监听型通信

- 常用对一些原生组件的一些封装

4. EventBus - 隔代、兄弟等非直接通信

- $emit, $on
- 优点
  - 原理简单，多层组件的事件传播
- 缺点
  - 很难模块化
  - 多人开发，容易造成一些 Bug
  - $on .$off

5. provide / inject - 隔代广播等

- 解决一层层传递的问题
- 非响应式

6. vuex - 整体状态机

- 多层组件的事件传播
- 单项数据流
- 统一状态管理

平衡熵。一个系统的混乱程度。

## vue 是如何实现数据驱动双向绑定的？响应式是如何实现的？

```js
// vue2
export function Vue(option = {}) {
  this.__init(option);
}

// initMixin
Vue.prototype.__init = function (options) {
  this.$options = options;
  this.$el = options.el;
  this.$data = options.data;
  this.$methods = options.methods;
  // data 的处理
  proxy(this, this.$data);
  observer(this.$data);
  new Compiler(this);
};

// this.$data.msg // this.msg
function proxy(target, data) {
  Object.keys(data).forEach((key) => {
    Object.defineProperty(target, key, {
      enumerable: true,
      configurable: true,
      get() {
        return data[key];
      },
      set(newVal) {
        if (newVal !== data[key]) {
          data[key] = newVal;
        }
      },
    });
  });
}

function observer(data) {
  new Observer(data);
}

class Observer {
  constructor(data) {
    this.walk(data);
  }

  walk(data) {
    if (data & (typeof data === "object")) {
      Object.keys(data).forEach((key) =>
        this.defineReactive(data, key, data[key])
      );
    }
  }

  defineReactive(obj, key, value) {
    let that = this;
    this.walk(value);

    let dep = new Dep();
    Object.defineProperty(target, key, {
      enumerable: true,
      configurable: true,
      get() {
        // {{ num }}
        Dep.target && dep.add(Dep.target);
        return data[key];
      },
      set(newVal) {
        if (newVal !== data[key]) {
          data[key] = newVal;
          that.walk(newVal);
          dep.notify();
        }
      },
    });
  }
}

// new Watcher(vm, 'num', () => { 更新视图显示 })
// new Watcher(this.vm, key, val => {node, textContent = val})

class Watcher {
  constructor(vm, key, cb) {
    this.vm = vm;
    this.key = key;
    this.cb = cb;
    // 此时，Dep.target 作为一个全局变量去理解，放的就是这个 watcher
    Dep.target = this;
    //
    this.__old = vm[key];
    // dep, target 干掉
    Dep.target = null;
  }

  update() {
    let newVal = this.vm[this.key];
    if (newVal !== this.__old) {
      this.cb(newVal);
  }
}

// 给一个数据，都要有一个依赖
class Dep {
  constructor() {
    this.watchers = new Set();
  }

  add(watcher) {
    if (watcher && watcher.update) this.watcher.add(watcher);
  }

  notify() {
    this.watchers.forEach((watc) => watc.update());
  }
}

class Compiler {
  constructor(vm) {
    this.el = vm.$el;
    this.vm = vm;
    this.methods = vm.$methods;
    this.compile(vm.$el)
  }

  compile(el) {
    let childNodes = el.childNodes;
    Array.from(childNodes).forEach(node => {
      if (node.nodeType === 3) {
        this.compileText(node);
      }
    })
  }

  compileText(node) {
    // {{ msg }}
    let reg = /\{\{(.+?)\}/;
    let value = node.textContent;
    if (reg.test(value)) {
      let key = RegExp.$1.trim();
      node.textContent = value.replace(reg, this.vm[key]);
      new Watcher(this.vm, key, val => {
        node.textContent = val
      })
    }
  }
}
```

## v-model 的含义是什么？不同版本有何差异？

Vue2 就是一个语法糖

```html
<el-input :value="foo" @input="foo = $event.target.value" />
```

.sync 的修饰符
<input :value="foo" @update:value="foo = $event.target.value" />

Vue3
为了让 v-model 更多的针对多个属性进行双向绑定

1. 去掉 sync，原本的功能，由 v-model 来代替
2. 对自定义组件使用 v-model 时，value -> modelValue

<input :value="modelValuee" @update:modelValue="foo = $event.target.value" />

## vue3 和 vue2 Diff 对比

Vue3 使用了最长上升子序列
runtime-core/src/renderer.ts 2494 行
莱文斯坦最短编辑距离。
[c, f, e] 和 [d, e, h, c]
可能有一些新增的、删减的、需要移动的。
old new 的顺序，泛化成数字。
[2,3,4,2,1,5]

## computed 和 watch 有何异同

computed:

- 缓存，不支持异步。
- 一般一个属性，可以由其他属性计算而来，可以用。

watch：

- 无缓存，可异步
- immdiate
- deep

## MVVM 的含义以及如何实现一个迷你版 MVVM 框架？

model-view: render -- 双向绑定
view-model: DOM 监听

## vue3.0 的特性有哪些？如何理解组合式编程？

- Proxy
- composition API
- patchFlag
- openBlock
- monorepo
- typescript

```html
<div>
  <div>hello</div>
  <div>{{ msg }}</div>
</div>
```

## vue-router 的核心功能？ $route 和 $router 有何区别

$router 路由器，访问这个项目的路由结构
router.beforeEach()
router.afterEach()
router.push()
router.replace()
router.go()
router.back()
router.forward()

$route，静态的信息。
fullpath，hash，meta，query， path， params

## vuex 的状态管理流程？如何正确使用状态机

```js
export default {
  name: "xxx",
  props: {
    msg: String,
  },
  mounted() {
    console.log(this.$store.state.age);
    console.log(this.$store.getters.getAgeStr);
    this.$store.commit("setAge", 19);
    this.$store.dispatch("setAgeAsync", 19);
  },
  computed: {
    ...mapState(["age"]),
    ...mapState(["getAgeStr"]),
  },
  directives: {
    xxx: {
      bind() {},
    },
  },
};
```

```js
import Vue from "vue";
import Vuex from "vuex";

Vue.use(vuex);

export default new Vue({
  state: {
    age: 18,
  },
  getter: {
    getAgeStr(state) {
      return `my age is ${state.age}`;
    },
  },
  mutations: {
    setAge(state) {
      state.age = 19;
    },
  },
  actions: {
    setAgeAsync(content, payload) {
      return new Promise((resolve, reject) => {
        content.commit("setAge", payload);
      });
    },
  },
  modules: {},
});
```
