# node

node / browser 宿主环境
V8 心脏
node 人体

## 如何理解 node 中模块的概念

模块化概念。

### 软件工程的本质是什么？

- 管理数据 - 管理变量
  - DB: lesson - blob / chatList -- chat -- name, text
  - BE: 数据的组织，查询
  - FE: runtime 的时间，管理变量。
    - 管理变量的生命周期 [model] 和展示形式 [view]。
- 进行通信

JS 脚本语言
window

```js
var bar = 1;
var bar = 2;
```

什么是模块？模块就是通过 函数作用域，把变量和函数，进行隔离。

**闭包，就是模块化的基石。**

### 模块化的方案

```js
function foo() {
  var bar = 1;
  var baz = 2;
}

function foo2() {
  var bar = "hello";
  var baz = "world";
}
```

只做到了隔离，没有做到相互的通信。

```js
// react 状态提升。
var module = {
  exports: {},
};

function resolve(nmodule, exports) {
  // *** CJS 模块 ***
  var bar = "hello";
  var baz = "world";
  nmodule.exports = {
    bar,
    baz,
  };
}

resolve(obj, obj.foo);
```

webpack 打包以后的 runtime.js
webpack 最终打包，bundle.js
浏览器端，最耗性能是什么？ I/O

```js
var module = {
  "./src/A.js": {
    exports: {},
  },
  "./src/B.js": {
    exports: {},
  },
};

function require(id) {
  return module[id].exports;
}

(function (module, exports, require) {
  // ./src/A.js
  const foo = require("./src/B.js");
  const bar = 1;
  const baz = 2;
})(module["./src/A.js"], module["./src/A.js"].exports, require);
```

## 简述 require 的模块加载机制

commonjs 遵循的是，文件直接获取。node 是 commonjs 的实现。
在 node 端，我的文件，是不是可以直接， fs.read

### 为什么浏览器端，不能 require？

require 的模块加载机制 是通过文件的读取，和 script 上下文执行实现的，浏览器端不能读。

## 如何理解 node 的事件循环流程

```bash
              同步的代码
                |
      process.nextTick / promise...
                |
   ┌───────────────────────────┐
┌─>│           timers          │ 定时器： setTimeout / setInterval 红黑树
│  └─────────────┬─────────────┘
|  process.nextTick / promise...
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │ 执行延迟到下一个循环迭代的 I/O 回调
│  └─────────────┬─────────────┘
|  process.nextTick / promise...
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │ 系统内部使用
│  └─────────────┬─────────────┘
|  process.nextTick / promise...      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  process.nextTick / promise...      └───────────────┘
|  ┌─────────────┴─────────────┐
│  │           check           │ setImmdiate
│  └─────────────┬─────────────┘
│  process.nextTick / promise...
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │ 关闭回调函数 socket.on('close', cb)
   └───────────────────────────┘
```

注意：每个框被称为事件循环机制的一个阶段。

### 做个题

```js
async function async1() {
  console.log("async1 started"); // 2
  await async2();
  console.log("async end"); // 8
}
async function async2() {
  console.log("async2"); // 3
}
console.log("script start."); // 1
setTimeout(() => {
  console.log("setTimeout0"); // 10
  setTimeout(() => {
    console.log("setTimeout1"); // 12
  }, 0);
  setImmediate(() => {
    console.log("setImmediate"); // 11
  });
}, 0);

async1();
process.nextTick(() => {
  console.log("nextTick"); // 7
});

new Promise((resolve) => {
  console.log("promise1"); // 4
  resolve();
  console.log("promise2"); // 5
}).then(() => {
  console.log("promise.then"); // 9
});
console.log("script end."); // 6
```

## 如何描述 异步 I/O 的流程

### 什么是阻塞/非阻塞？

系统在接收输入的时候，再到输出的过程中，能不能接收其他的输入？

- 食堂打饭的阿姨，能不能打着你的，同时给别人打？ 阻塞的。
- 餐厅的服务员，再给你下了单以后，能不能去给别人下单？ 非阻塞。

解析中

思考一个问题：多个任务执行时，存在 I/O 和 cpu 计算任务，如何合理地利用资源？

#### 方案 1：多线程

线程之间要切换，代码也要枷锁。

#### 方案 2：单线程 + 异步 I/O

- I/O 是不能阻塞 CPU 的执行
- 不要带来锁的问题
- 性能要 OK

轮询
你直接通知我

### 模块化的历史

commonjs 规范，是以 node.js 为代表；
amd 规范的实现，是以 require.js 为代表；
cmd 规范的实现，是以 sea.js 为代表；

```js
define("foo", ["loadsh", "jquery"], function (lodash, jquery) {
  const { debounce } = lodash;

  return {
    name: "xxxx",
  };
});

require.config({
  paths: {
    jquery: "www.xxx.xxx.jquery.cdn.js",
  },
});

require(["foo", "lodash"], function (a, _) {
  console.log("1");
});
```

cmd：

```js
define(functio(require, exports, module) {
  // 1. 直接加载
  var a = require('./a') // 这里引用 js 模块
  // 2. 异步加载模块
  var  b = require.async(['./b,../c'], function() {
    c.doSomeThing();
    b.doSomeThing();
  })
  // 3. 使用模块系统内部的路径解析机制来解析并返回模块路径。该函数不会加载模块，只返回解析后的绝对路径。
  console.log(require('./d')); //http://example.com/path/to/d.js
  // 这里是模块代码
})
```

umd：

```js
(function (global, factory) {
  if (typeof module === "object" && exports !== "undefined") {
    module.exports = factory();
  } else if (typeof define === "function") {
    if (define.amd) {
      define("module", factory);
    } else if (define.cmd) {
    }
  } else {
    global.module = fatory();
  }
})(this, function () {
  return "module";
});
```

## 简述 V8 的垃圾回收机制以及关键点

### 为什么会产生垃圾。

```js
window.foo = function () {};

window.bar = Object.create({});

window.bar = 1;
```

栈 | window 的指针 |
堆 | window 的变量 | foo 的地址 | bar 的地址
堆 | function() {...} | {} |

### 垃圾回收，是如何实现的。

引用计数。
a -> b b -> a

标记清除。
我从根节点出发，所有的能够索引到的，都不是垃圾，其他的，就是垃圾。

- GC root
  - 全局 window / global 对象
  - DOM 树
  - 存在栈上的变量。

### V8 怎么样清除，两个垃圾回收器 / 代际假说

- Major 主垃圾回收器 / 老生代 / 老而不死
- Minor 副垃圾回收器 / 新生代 / 朝生夕死 / Scavenge

## 内存泄露是什么，常见的内存泄露原因以及排查方法是什么

1. 全局变量
2. 函数闭包
3. 事件监听

检测方法：

1. heapdump / npm
2. chrome devtools

## websocket 与 常规的 http 有何区别

websocket 是一个双向的通信协议。客户端可以通过 upgrade 一个 http，升级 websocket，和服务器保持长链接。
http 是一个在 tcp 协议纸上的单项协议。

## 简述对于 node 的多进程架构的理解

- Master - Worker 的一个模式，主从模式。
- fork 复制出一个独立的进程。
- libuv 进行提供的

## 如何创建子进程有几种方式，以及子进程 crash 后如何自动重启

node ./index.js [express]
fork 子进程的方式，8 个进程，在给你工作。
spawn fork exec execfile

- spawn

````js
// 在父进程中
const cp = spawn("node", "child.js", {
  cwd: path.resolve(process.cwd(), "./worker"),
  // stdio: ["pipe", "pipe", 2],
  stdio: [process.stdin, process.stdout, process.stderr],
});

// 在子进程中
process.stdout.write("sum:" + sum);

- fork
stdio 默认是 ipc 的方式。

- exec
```js
const cp = execFile('node --version', {
  cwd: path.resolve(process.cwd(), './worker')
}, function(err, stdout, stderr) {

})
````

- execfile

```js
const cp = execFile(
  "node, [--version]",
  {
    cwd: path.resolve(process.cwd(), "./worker"),
  },
  function (err, stdout, stderr) {}
);
```

传参的方式是数组
#### 如何重启
- 一般在 生产环境中，使用 pm2 进行进程守护，原理是，父进程监听子进程的 退出事件，然后 restart。
- 进行健康检查，定时启动一个进程，查询目标进程的健康状态。


## 简述 koa 的中间件原理
```js

```