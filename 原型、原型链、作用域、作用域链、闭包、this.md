### 原型、原型链、作用域、作用域链、闭包、this
#### 什么是原型？
原型是JavaScript的重要特性之一，可以**让对象从其他对象继承功能特性**，所以JavaScript也被称为“基于原型的语言”。严格地说，原型应该是对象的特性，当函数其实也是一种特殊的对象。例如，我们对自定义的函数进行instance Object操作时，其结果是true。
```js
function fn() {}

fn instanceof Object; // true
```
#### 原型与构造函数
在js中我们是使用构造函数来新建一个对象的，每一个构造函数的内部都有一个prototype的属性值，这个属性值是一个对象，这个对象包含了可以由该构造函数的所有实例共享的属性和方法。
当我们使用构造函数新建一个对象后，在这个对象的内部将包含一个指针，这个指针指向构造函数的prototype属性对应的值，在ES5中这个指针被称为对象的原型。一般来说我们是不应该能够获取到这个值的，但是现在浏览器中都实现了`__proto__`属性来让我们访问这个属性，但是我们最好不要使用这个属性，因为它不是规范中的规定。
ES5中新增了一个Object.getPrototypeOf()方法，我们可以通过这个方法来获取对象的原型。当我们访问一个对象的属性时，如果这个对象内部不存在这个属性，那么它就会去它的原型对象中找到这个属性，这个原型链的尽头一般来说都是Object.prototype，所以这就是我们新建的对象为什么能够使用toString()等方法的原因。
**获取原型的方法**假如p是一个实例，p 获取原型的方法如下三种其中obj.constructor指向构造函数
- `obj.__proto__`
- `obj.constructor.prototype`
- `Object.getPrototypeOf(obj)`
```
function a() {
    this.a = 1;
    this.b = 2;
}
let obj = new a();
console.log(obj);
console.log(obj.constructor === a.prototype.constructor); // true
console.log(obj.constructor.prototype === a.prototype); // true
console.log(obj.prototype.constructor.prototype === a.prototype); // true
console.log(a.prototype.isPrototypeOf(obj)); // true
```
提示JavaScript对象是通过引用来传递的，我们创建的每个新对象实体中并没有一份属于自己的原型副本。当我们修改原型时，与之相关的对象也会继承这一改变。提示 浏览器中都实现了`__proto__`属性来让我们访问`[[Prototype]]`的值，但是我们最好不要使用这个属性，因为它不是规范中规定的，虽然我们在脚本中没有办法访问到`[[Prototype]]`属性，但是`isPrototypeOf()`方法是用于测试一个对象是否存在于另一个对象的原型链上。在ECMAScript 5中新增了一个方法叫`Object.getPrototype()`，这个方法返回`[[Prototype]]`的值。
#### 隐式原型和显式原型
**隐式原型通常在创建实例的时候就会自动指向构造函数的显示原型。**
例如，在下面的示例代码中，当创建对象a时，a的隐式原型会指向构造函数Object()的显示原型
```js
var a = {};
a.__proto__ === Object.prototype; // true
var b = new Object();
b.__proto__ === a.__proto__; // true
```
显示原型是内置函数（比如Date()函数）的默认属性，在自定义函数时（箭头函数除外）也会默认生成，生成的显示原型对象只有一个属性constructor，该属性指向函数自身。通常配合 new 关键字一起使用，当通过 new 关键字的创建函数实例时，会将实例的隐式原型指向构造函数的显式原型
```
function fn() {}
fn.prototype.constructor === fn; // true
```
隐式原型是否必须与显式原型配合使用呢？下面的代码声明了parent和child两个对象，其中对象child定义了属性 name 和隐式原型 proto，隐式原型指向对象 parent，对象parent定义了code和name两个属性。当打印child.name的时候会输出对象child的name属性值，当打印child.code时由于对象child没有属性code，所以会找到原型对象parent的属性code，将parent.code的值打印出来。同时可以通过打印结果看到，对象parent并没有显性原型属性。如果要区分对象child的属性是否继承自原型对象，可以通过hasOwnProperty()函数来判断。
```js
var parent = { code: 'p', name: 'parent' };
var child = { __proto__: parent, name: 'child' };
console.log(parent.prototype); // undefined
console.log(child.name); // "child"
console.log(child.code); // "p"
child.hasOwnProperty("name"); // true
child.hasOwnProperty("code"); // false
```
在这个例子中，如果对象 parent 也没有属性 code，那么会继续在对象 parent 的原型对象中寻找属性 code，以此类推，逐个原型对象依次进行查找，直到找到属性code或原型对象没有指向时停止。这种类似递归的链式查找机制被称作“原型链”。
#### 原型的属性
每当代码读取对象的某个属性时，首先会在对象本身搜索这个属性，如果找到该属性就返回该属性的值，如果没有找到，则继续搜索该对象对应的原型对象，以此类推下去。
因为这样的搜索过程，因此我们如果在实例中添加一个属性时，这个属性就会屏蔽原型对象中保存的同名属性，因为在实例中搜索到该属性后就不会再向后搜索了。
##### 属性判断
既然一个属性既可能是实例本身的，也有可能是其原型对象的，那么我们该如何来判断呢？
答：使用**hasOwnProperty()**
**JavaScript中，有一个函数，执行时对象查找时，永远不会去查找原型**
hasOwnProperty 所有继承了 Object 的对象都会继承到 hasOwnProperty 方法。这个方法可以用来检测一个对象是否含有特定的自身属性，和 in 运算符不同，该方法会忽略掉那些从原型链上继承到的属性。
在使用for-in循环时，返回的是**所有能够通过对象访问的、可枚举的属性，**其中包括了存在于实例中的属性，也包括了存在原型中的属性。需要注意的一点是，屏蔽了实例中不可枚举属性的实例属性也会在for-in循环中返回。提示 因此我们可以封装这样一个函数，来判断一个属性是否存在原型中
```js
function hasPrototypeProperty(object, name) {
    return !object.hasOwnProperty(name) && name in object;
}
```
**获取所有属性**如果想要获得对象上可枚举的实例属性，可以使用 Object..keys()方法，这个方法接受一个对象作为参数，返回一个包含可枚举属性的字符串数组。如果想要获取所有的实例属性，无论它是否可以枚举，我们可以使用 `Object.getOwnPropertyNames()`方法。

#### 什么是原型链
通过一个对象的proto可以找到它的原型对象，原型对象也是一个对象，就可以通过原型对象的`__proto__`，最后找到了我们的Object.prototype，从实例的原型对象开始一直到 Object.prototype就是我们的原型链。
在介绍原型时就引出了原型链的概念，其实上面讲原型就已经包含了许多原型链的知识。当我们访问一个对象的属性时，如果这个对象内部不存在这个属性，那么它就会去它的原型对象里找到这个属性，这个原型对象又会有自己的原型，于是就这样一直找下去，也就是原型链的概念。原型链的尽头一般来说就是Object.prototype，所以这就是我们新建的对象为什么能够使用toString()等方法的原因，这种类似递归的链式查找机制被称作“原型链”。

#### 案例
##### 在 Array 本地对象上加原型方法
用途是去重升序排序，最后返回新数组
```js
Array.prototype.distinct = function() {
    return [...new Set(this)].sort((a, b) => a - b)
};

console.log(["a", "b", "c", "d", "b", "a", "e"].distinct()) // ['a', 'b', 'c', 'd', 'e']
```
##### 通过原型链实现多层继承
假设构造函数 B() 需要继承构造函数 A()，就可以通过将函数 B() 的显式原型指向一个函数 A() 的实例，然后再对 B 的显式原型进行扩展。那么通过函数 B() 创建的实例，既能访问用函数 B() 的函数 b，也能访问函数 A() 的属性 a，从而实现了多层继承。
```js
function A() {}

A.prototype.a = function() {
    return "a";
};

function B() {}

B.prototype = new A();

B.prototype.b = function() {
    return "b";
};

var c = new B();

c.b(); // 'b'

c.a(); // 'a'
```
##### 函数也是对象
```js
function Foo(who) {
    this.me = who;
}

Foo.prototype.identify = function() {
    return "I am" + this.me;
}

function Bar(who) {
    Foo.call(this, who)
}

Bar.prototype = Object.create(Foo.prototype);

Bar.prototype.speak = function () {
    alert("Hello, " + this.identify() + ".")
};

var b1 = new Bar("b1");
var b2 = new Bar("b2");

b1.speak(); // Hello, I am b1
b2.speak(); // Hello, I am b2
```

#### 作用域和闭包
##### 定义
红宝书：闭包是指那些引用了另一个函数作用域中变量的函数，通常是嵌套在函数中实现的。
小黄书：当函数可以记住并访问所在的词法作用域时，就产生了闭包，即时函数是在当前词法作用域之外执行。
MDN：闭包是指那些能够访问自由变量的函数（其中自由变量，指在函数中使用的，但既不是函数参数arguments 也不是函数的局部变量的变量，其实就是另外一个函数作用域中的变量。）

所以：闭包是指有权访问另一个函数作用域中的变量的函数，**创建闭包的最常见方式**就是在一个函数内创建另一个函数，创建的函数可以访问到当前函数的局部变量。
##### 闭包的用途
1、在函数外能够访问到函数内部的变量。通过使用闭包，我们可以通过外部调用闭包函数，从而在外部访问到函数外的变量，可以使用方法来创建私有变量。
2、使已经运行结束的函数上下文中的变量对象继续留在内存中，因为闭包函数保留了这个变量对象的引用，所以这个变量不会被回收。其实闭包的本质就是作用域链的一个特殊的应用。
##### 产生的原因
在ES5中只存在两种作用域 ---- 全局作用域和函数作用域，当访问一个变量时，解释器会首先在当前作用域查找标识符，就去父作用域找，直到找到该变量的标识符或者不在父作用域中，这就是作用域链，值得注意的是，**每一个子函数都会拷贝上级的作用域，形成一个作用域链的链条。**
```js
var a = 1;
function f1() {
    var a = 2;
    function f2() {
        var a = 3;
        console.log(a); // 3
    }
}
```
**解析** 在这段代码中，f1 的作用域指向有全局作用与（window）和它本身，而 f2 作用域指向作用域（window）、f1和它本身。而作用域是从最底层向上找，直到找到全局作用域 window 为止，如果全局还没有的话会报错。
**闭包产生本质，**就是当前环境中存在指向父级作用域的引用。
```js
function f1() {
    var a = 2;
    function f2() {
        console.log(a); // 2
    }
    return f2;
}
var x = f1();
x();
```
**解析** 这里 x 会拿到父级作用域中的变量，**输出2** 因为在当前环境中，含有对 f2 的引用，f2 恰恰引用了window、f1和f2的作用域。因此 f2 可以访问到 f1 的作用域的变量，这里是返回函数的情况，回到闭包的本质，我们只需要让父级作用域的引用存在即可，因此我们还可以这么做：
```js
var f3;
function f1() {
    var a = 2;
    f3 = function() {
        console.log(a);
    };
}
f1();
f3();
```
**解析**：让 f1 执行，给 f3 赋值后，等于说现在 f3 拥有了window、f1和f3本身这几个作用域的访问权限，还是自底向上查找，最近是在 f1 中找到了 a，因此输出 2。在这里是外面的变量 f3 存在这父级作用域的引用，因此产生了闭包，形式变了，本质没有改变。

#### 如何使用
1、返回一个函数，上面已经举例
2、作为函数参数传递
```js
var a = 1;
function foo() {
    var a = 2;
    function baz() {
        console.log(a);
    }
    bar(baz);
}
function bar(fn) {
    // 这就是闭包
    fn();
}
// 输出2，而不是1
foo();
```
**3、在定时器、事件监听、Ajax请求、跨窗口通信、Web Workers或者任何异步中，只要使用了回调函数，实际上就是闭包以下的闭包保存的仅仅是window的当前作用域。**
```js
// 定时器
setTimeout(function timeHandler() {
    console.log('111');
}, 100)


// 事件监听
$('#app').click(function() {
    console.log('DOM Listener');
})
```
**4、（立即执行函数表达式）创建闭包，保存了全局作用域window和当前函数的作用域，因此可以全局的变量**
```js
var a = 2;
(function IIFE() {
    // 输出2
    console.log(a);
})();
```
#### 优缺点
- 为创建内部作用域而调用了一个包装函数（函数嵌套函数）
- 包装函数的返回值必须至少包含一个对内部函数的引用，这样就会创建涵盖整个包装函数内部作用域的闭包（函数内部可以引用外部的参数和变量）。
- 参数和变量不会被垃圾回收机制回收。
**优点：**
- 希望一个变量长期存储在内存中。
- 避免全局变量的污染。
- 私有成员的存在
**缺点：**
- 常驻内存；
- 增加内存使用量。
- 使用不当会很容易造成内存泄漏。
```js
function outer() {
    var name = 'jack';
    function inner() {
        console.log(name);
    }
    return inner;
}
outer()(); // jack
function sayHi(name) {
    return () => {
        console.log(`Hi! ${name}`);
    };
}
const test = sayHi('xiaoming');
test(); // Hi! xiaoming
```
**解析**
虽然 sayHi 函数已经执行完毕，但是其活动对象也不会被销毁，因为 test 函数仍然引用着 sayHi 函数中的变量 name，这就是闭包。但也因为闭包引用着另一个函数的变量，导致另一个函数已经不使用了也无法销毁，所以闭包使用过多，会占用较多的内存，这也是一个副作用。由于在ECMA2015中，只有函数才能分割作用域，函数内部可以访问当前作用域的变量，但是外部无法访问函数内部的变量，所以闭包可以理解为“定义在一个函数内部的函数。外部可以通过内部返回的函数访问内部函数的变量”。在本质上，闭包是将函数内部和函数外部连接起来的桥梁。

##### 案例
```
for (var i = 1;i <= 5; i++) {
    setTimeout(function timer() {
        console.log(i);
    }, 0);
}
// 6 6 6 6 6
for (var i = 1; i <= 5; i++) {
    setTimeout(function timer() {
        console.log(i);
    }, i * 1000);
}
// 6 6 6 6 6
```
因为 setTimeout 为宏任务，由于 JS 中单线程 eventLoop 机制，在主线程同步任务执行完成后才去执行宏任务，因此循环结束后 setTimeout 中的回调才依次执行，当输出 i 的时候当前因为作用域没有，往上一级再找，
**解决方法：**利用IIFE（立即执行函数表达式）
```js
// 当每次 for 循环时，把此时的 i 变量传递到定时器中
for (var i = 1; i <= 5; i++) {
    (function(j) {
        setTimeout(function timer() {
            console.log(j);
        }, 0);
    })(i);
}
```
提示：这种方法属于使用闭包解决
给定时器传入第三个参数，作为 timer 函数的第一个函数参数
```js
for (var i = 1; i <= 5; i++) {
    setTimeout(function timer(j) {
        console.log(j);
    }, 0, i);
}
```
**setTimeout 的参数**
**function**: 你想要在到期时间（delay 毫秒）之后执行的函数。
**code**：这是一个可选语法，你可以使用字符串而不是function，在delay 毫秒之后编译和执行字符串（使用该语法是不推荐的，原因和使用eval()一样，有安全风险）。
**delay(可选)**：延迟的毫秒数（一秒等于1000毫秒），函数的调用会在该延迟之后发生。如果省略该参数，delay取默认值0，意味着“马上”执行，或者尽快执行。不管是哪种情况，实际的延迟时间可能会比期待的（delay 毫秒数）值长，原因请查看实际延时比设定值更久的原因：最小延迟时间。
**arg1, ..., argN（可选）**：附加参数，一旦定时器到期，它们会作为参数传递给function。
#### 使用 ES6 中的
```js
let for (let i = 1; i <= 5; i++) {
    setTimeout(function timer() {
        console.log(i);
    }, 0);
}
```
**关于 let**
let 使 JS 发生革命性的变化，让 JS 有函数作用域变为了块级作用域，用 let 后作用域链不复存在。代码的作用域以块级为单位。
#### 模拟私有变量
```js
const book = (function() {
    var page = 100;
    return function() {
        this.auther = 'okaychen';
        this._page = function() {
            console.log(page);
        };
    };
})();

var a = new book();
a.auther; // 'okaychen'
a._page(); // 100
a.page(); // undefined
```
#### 使用闭包打印标签的index
```html
<ul id="test">
    <li>这是第一条</li>
    <li>这是第二条</li>
    <li>这是第三条</li>
</ul>
```
```js
// 方法一：
var lis = document.getElementById('test').getElementsByTagName('li');
for (var i = 0; i < 3; i++) {
    lis[i].index = i;
    lis[i].onclick = function() {
        alert(this.index);
    };
}
// 方法二：
var lis = document.getElementById('test').getElementByTagName('li');
for (var i = 0; i < 3; i++) {
    lis[i].index = i;
    lis[i].onclick = (function(a) {
        return function() {
            alert(a);
        };
    })(i);
}
```
#### 实现单例模式
```js
var SingleStudent = (function() {
    function Student() {}

    var _student;
    return function() {
        if (_student) return _student;

        _student = new Student();

        return _student;
    };
})();

var s = new SingleStudent();

var s2 = new SingleStudent();

s === s2; // true
```
#### 闭包与模块
```js
来自小黄书 考虑以下代码：
function CoolModule() {
    var something = 'cool';
    var another = [1, 2, 3];
    function doSomething() {
        console.log(something);
    }
    function doAnother() {
        console.log(another.join('!'));
    }
    return {
        doSomething: doSomething,
        doAother: doAother,
    };
}
var foo = CoolModule();
foo.doSomething(); // cool
foo.doAother(); // 1 ! 2 ! 3
```
这个模块在 JavaScript 中被称为模块。最常见的实现模块模式的方法通常被称为模块暴露，这里展示的是其变体。
首先，CoolModule() 只是一个函数，必须要通过调用它来创建一个模块实例。如果不执行外部函数，内部作用域和闭包都无法被创建。
其次，CoolModule()返回一个用对象字面量语法 { key: value, ...} 来表示的对象。这个返回的对象中含有对内部函数而不是内部数据变量的引用。我们保持内部数据变量是隐藏且私有的状态。可以将这个对象类型的返回值看做本质上是模块的公共API。
这个对象类型的返回值最终被赋值给外部的变量foo，然后就可以通过它来访问API中的属性方法，比如foo.doSomething()。
doSomething()和doAnother()函数具有涵盖模块实例内部作用域的闭包（通过调用CoolModule()实现）
#### 实现模块的单例模式
```js
var foo = (function CoolModule() {
    var something = 'cool';
    var another = [1, 2, 3];
    function doSomething() {
        console.log(something);
    }
    function doAnother() {
        console.log(another.join('!'));
    }
    return {
        doSomething: doSomething,
        doAnother: doAnother,
    };
})();
foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```
#### 将模块函数转换成IIFE
立即调用这个函数并将返回值直接赋值给单例的模块实例标识符 foo 命名将要作为公共 API 返回的对象，这是模块模式的另一个简单但强大的用法
```js
var foo = (function CoolModule(id) {
    function change() {
        // 修改公共API
        publicAPI.identify = identify2;
    }
    function identify1() {
        console.log(id);
    }
    function identify2() {
        console.log(id.toUpperCase());
    }
    var publicAPI = {
        change: change,
        identify: identify1
    };
    return publicAPI;
})("foo module");
foo.identify(); // foo module
foo.change();
```