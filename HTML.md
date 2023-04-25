# HTML

## HTML DOCTYPE 的含义？ 什么是 HTML 的标准模式与混杂模式？

HTML 的文档类型声明，doctype，说明这个页面是用什么来编写的。

- h5 html5 有一个比较宽送的语法，基本上完全向后兼容。

```html
<!DOCTYPE html>
```

- h4.0.
  - strict 结构中不能有出现格式或表现的内容
    - `<b></b>`，`<p font='5'></p>`
  - tansitional

```html
<!-- strict html -->
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<!-- strict xhtml -->
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">

<!-- transitional html -->
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<!-- transitional xhtml -->
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
```

## HTML5 有哪些语义化标签及其特性？HTML 元素有哪些分类与特性？

让我们根据结构化的内容，选择合适的标签。

- seo 有利；
- 代码的可读性，更好；
- 标签上加上 alt，title；
- accessibility 方便一些其他的设备进行解析。

## 如何检测浏览器是否支持 HTML5 特性？

- canvas
- video，audio
- 本地缓存的支持 localStorage， Worker
- article， footer， header
- form： calender， date
- esm es module, script 是不再需要 type 属性了。

1. 检查特定的属性和方法

```
!!navigator.geolocation
!!window.localStorage
!!window.Worker
```

2. 创建一个元素，看看特定元素，有没有属性和方法

```js
document.createElement("canvas").getContext();
document.createElement("video").canPlayType;
```

3. 第三方库
   http://modernizr.cn/

## HTML 中 meta 的作用？

## HTML 的标签有哪些可以优化 SEO？

1. meta 中的相关属性
   `<meta name="author" content="aaa@gmail.com">`
   `<meta name="description" content="XXX XRM XXX 系统">`
   `<meta name="keywords" content="XXX XRM XXX 系统">`

2. 标签
3. title
4. meta
5. header
6. nav
7. article
8. aside
9. footer

A：

1. 首先要保证 SSR 的；
2. meta 中相关的属性；
3. 语义化标签，以一些结构化的为主。 费效比比较低。

## DOM 和 BOM 有什么区别？

JavaScript 在浏览器环境下，一般由三部分组成。

- ECMAScript 核心。描述了 JS 的语法和基本对象；
- DOM 文档对象模型，document。 你有一些 API，可以操作 文档。
- BOM 浏览器对象模型， browser。你有一些 API，可以操作 浏览器。

## 如何实现移动端适配？

理解设备像素比，window.devicePixelRatio

### 1px 问题

先放大 200%，然后 scale(0.5)

### rem 方案

rem 指的是 html 的 font-size 的大小。

## 如何禁用页面中的右键、打印、另存为、复制等功能？

```js
// 右键
document.onmousedown = function(event) {
  if (event.button === 2) {
    return false
  }
}

document.oncontextmenu = function(event) {
  return false
}

// 复制
// <body oncopy="nocopy()"></body>
function nocopy(event) {
  event.returnValue = false
}

// f12
document.onkeydown = function(e) {
  if (window.event & window.event.keyCode === 123) {
    window.event.returnValue = false
}
```

## href="javascript:void(0)"和 href="#"区别是什么？

href="#" 我的锚点，默认是 #top，会让你网页往上走。
javascript:void(0) 死链接，组织时间。

## 对 target="\_blank"的理解？有什么安全性问题？如何防范？

target="\_blank"， 类似于 window.open，会出现问题：你的子页面，会拿到你当前的句柄。
window.opener
问题：

```js
if (window.opener) {
  window.opener.location.href = "bad.html";
}
```

解决办法：（浏览器应该已经修复过了）

```html
<a href="x.html" target="_blank" rel="noopener noreferer">跳转</a>
```

解决办法：

```js
var otherWindow = window.open("xxx");
otherWindow.opener = null;
```

## 简述页面的储存区别？什么是本地存储？怎么做离线缓存？

1. cookie
   每个 cookie 不能超过 4kb
   每个域 20 个
   [key, value, domain, expireTime, httpOnly, secure, ss]
2. web storage
   localStorage：永久存储，只要不清空内存就一直存在
   sessionStorage：储存在会话
   5MB
3. indexedDB：浏览器端的数据库 ---之前的版本[webSQL]
4. application cache
   1. pwa
   2. service worker

## 什么是 canvas？什么时候需要使用 canvas？

canvas 的中文：画布。
动画方案：

- css div：普通的网页；
- svg 和传统的 html 差别不大。html 处理矢量绘图的能力不足。
- canvas 2d
- canvas webGL OpenGL 的 ES 规范，在 web 端的实现。利用 GPU 去渲染一些，3d/2d 的图形。

## 什么是 PWA？

渐进式网页应用。
核心技术：

- app manifest
- service worker 客户端代理的工作
- web push

## 什么是 Shadow DOM？

web component 做到真正的组件化。

- 原生规范，无需框架
- 原生使用，无需编译
- 真正意义上的 css scope

```js
customElements.define(
  "shadow-test",
  class extends HTMLElement {
    connectedCallback() {
      const shadow = this.attachShadow({ mode: "open" });
      shadow.innerHTML = "this is a shadow dom";
    }
  }
);
```

<!-- stencil 框架 -->

## iframe 有哪些应用？

- 最常见的一种微前端手段
- ajax 上传文件
- 广告
- 跨域

## 如何处理 iframe 通信？

- 同域下面

```js
document.domain = "baidu.com";
frame.contentWindow.xxx;
```

- post message

## 什么是 web worker？为什么要使用 web worker？

## ？什么是 SSO 打通？怎么做前端沙盒模式？

## 浏览器的渲染和布局逻辑是什么？

- DOM 树构建
- CSS 树构建
- 渲染树构建
- 页面布局
- 页面绘制

## 页面的重绘回流是什么？

- 回流就是重排

## 怎样计算首屏和白屏的时间？常统计的页面性能数据指标包括？

window.Performance()可以拿到想要的所有时间。
首屏渲染时间一般是：responseEnd -> fetchStart 这两者的时间差。

FP FCP

PerformanceObserver

## 页面上有哪些领域可以做进一步的性能优化？

- visibility:hidden --> display: none
- 避免使用 table
- 避免层级过多
- dom insert -- fragment
- requestIdelCallback

FCP 首屏渲染不要太多内容
CLS 不要重流重绘，避免大量的插入 dom 或者操作 dom
FID 输入延时，用在输入的时候不要做太多操作

## 浏览器之间的线程调度是怎么做的？
任务队列（宏任务）——> UI 主线程 --> 微任务 --> I/O network --> 对应的资源进行处理这个过程 --> 任务队列(底部) [循环这个过程]

与 UI 线程互斥

浏览器主进程、GPU 进程、网络进程、多个渲染进程、多个插件进程