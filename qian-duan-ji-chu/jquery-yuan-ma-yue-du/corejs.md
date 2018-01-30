# src/core.js

小时候的语文考卷会出现这样的题目：

> 请将下列句子的主谓宾找出来：
>
> 1. 苏州园林是我国各地园林的标本 （答案：园林是标本）
> 2. ...

我们也来找一下 **src/core.js** 的主谓宾

```js
define([], function() {
  
  jQuery = function ( selector, context ) {};                // <1>
  jQuery.fn = jQuery.prototype = {};                         // <2>
  jQuery.extend = jQuery.fn.extend = function() {};
  jQuery.extend({});
  
  return jQuery;
});
```

define 是 AMD \(Asynchronous Module Definition\) 定义 JavaScript 模块及依赖关系的语法，暂且忽略它背后的故事。define 的第一个参数规定了 src/core.js 的依赖，第二个参数是一个函数，它的返回值即为本模块的输出。

有了对 define 的基本理解，本模块做的事情就十分清晰了：

1. 定义一个叫做 jQuery 的函数
2. 定义 jQuery 的 prototype，同时给它的 prototype 一个别名 fn。有了 prototype 我们就可以利用 JavaScript 的原型链来构造对象
3. 在 jQuery.prototype 下额外定义一个复杂的函数 extend，扎实的英文基础告诉我这个函数可以用来扩展 jQuery 的功能；同时为了方便，给 extend 函数加了个捷径，从 jQuery 可以直接拿到 extend 函数
4. 利用 extend 函数为 jQuery 扩展一些功能
5. 输出 jQuery

#### jQuery 与 jQuery.prototype

平时我们使用 jQuery 时，场景大致是这样：

```js
$('div.container').text()
$('button[type="submit"]').on("click", function(e) { //... })
```

我每次用它，我都很好奇 **$\('string'\) **究竟是什么样的操作，现在看了 **&lt;1&gt;、&lt;2&gt;** 两行代码茅塞顿开。**$\('string'\) **实际上利用 **jQuery** 函数构造了一个新的对象，这个对象在原型链的下一级就是 **jQuery.prototype** ，因此这个对象也拥有 jQuery.prototype 中定义的所有能力，我们常用的 text、data、 prop、on、off 等这些烂熟于心的 api 也必定来源于这个对象。

