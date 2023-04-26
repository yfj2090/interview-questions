### JS

#### 面试方式

特点：逐步挖掘、层层深入

#### 类型检测 & 快速区分

1. JS 有几种基础数据类型？几种新增？ \*
   JS 8 中基础数据类型： undefined null boolean number string object | symbol bigInt

symbol 独一无二 且 不可变 => 全局变量冲突、内部变量覆盖
bigInt 任意精度正数 安全地存储和操作大数据，即便超出了 number 的安全整数范围

2. 基础数据类型通常会如何进行分类？使用起来有什么区别？使用过程中你是如何区分他们的？ \*\*\*
   可以分为：原始数据类型 + 引用数据类型
   原始数据类型： undefined null boolean number string
   引用数据类型： 对象、数组、函数

效果不同
原始数据类型直接赋值后，不存在引用关系
属性的引用关系

存储位置不同：
栈：原始数据类型 => 先进后出栈维护结构 => 栈区由编译器自动分配释放 => 临时变量方式
堆：引用数据类型 => 堆内存由开发者进行分配 => 直到应用结束

原始数据放置在栈中，空间小、大小固定、操作频繁
引用类型数据量大、大小不固定，赋值给的是地址

3. 如何进行类型区分判断？几种对类型做判断区分的方式？\*
   typeof

```js
typeof 2; // number
typeof true; // boolean

// 问题
typeof {}; // object
typeof []; // object
```

=> 有哪些需要注意的特例？

```js
typeof null; // object
typeof NaN; // number
```

instanceof

```js
  2 instanceof Number // true
  [] instanceof Array // true
```

那你能说说或者手写一下 instanceof 的原理实现？ \*\*\*
通过翻户口本，查家庭信息

```js
function myInstance(left, right) {
  // 获取对象的原型
  let _proto = Object.getPrototypeOf(left);
  // 构造函数的 prototype
  let _prototype = right.prototype;

  while (true) {
    if (!_proto) {
      return false;
    }

    if (_proto === _prototype) {
      return true;
    }

    _proto = Object.getPrototypeOf(_proto);
  }
}
```

constructor

```js
((2).constructor ===
  Number(
    // true
    []
  ).constructor) ===
  Array; // true
```

=> 隐患？ \*\*\*
constructor 代表的是构造函数指向的类型，可以被修改的

```js
function Fn() {}
Fn.prototype = new Array();

var f = new Fn();
```

原汁原味的 toString，防止 toString 被重写
Object.prototype.toString.call()
```js
  let a = Object.prototype.toString;

  a.call(2)
  a.call([])
```
=> 为啥这里要用 call？同样是检测 obj 调用 toString, obj.prototype.toString().call(obj)结果不一样？为什么？**

保证 toString 是 Object 上的原型方法，根据原型链知识，优先调用本对象属性 => 原型链

=> 当对象中有某个属性和 Object 的属性重名时，使用的顺序是什么样的？如果说优先使用 Object 属性，如何做？**

#### 类型转换
1. isNaN 和 Number.isNaN 的区别？ **
isNaN 包含了一个隐式转化。
  isNaN => 接受参数 => 尝试参数转成数值型 => 不能被转换数值的参数 返回 true => 非数字值传入返回 true

  Number.isNaN => 接受参数 => 判断参数是否为数字 => 判断是否为 NaN => 不会进行数据类型转换