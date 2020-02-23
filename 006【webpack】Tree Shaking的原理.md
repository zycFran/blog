# Tree Shaking 原理

`Tree Shaking` 又称摇树优化，是前端优化代码体积的一种手段。简单的说就是把无用的代码在打包(压缩)阶段直接删除。比如，我们在开发中需要引用`Antd`这个组件库中的`Button`组件，在打包发布阶段时，我们希望只引入`Button`组件而不需要其他组件代码，`Tree Shaking`可以帮助我们做到这一点，大大缩减了打包后的代码体积。

`Tree Shaking`最早是由 rollup 提出，之后 webpack2.x 也借助 UglifyJs 实现了。目前，webpack4.x 已默认支持，开启方式:

- 在 .babelrc 文件中，设置 module:false
- 设置 mode 为 production 模式

## 原理

值得注意的是，`Tree Shaking` 只对 `ES2015 (ES6)`引用的模块有效，对使用 `require` 的 `CommonJs` 模块无法适用。这是因为 `Tree Shaking` 的实现原理是在编译阶段对代码的依赖关系进行分析，然后对未使用的代码块进行标记，最后在压缩阶段进行删除。

`ES6`的模块的特点：

- 只能作为模块顶层的语句出现
- import 的模块名只能是字符串常量
- import binding 是 immutable 的

由上可以看出`ES6`的模块依赖关系是确定的，和运行时无关，在代码不运行的情况下就可以静态分析出不需要的代码。而`CommonJs`模块的是动态的，可以基于不同条件导入，因此只有在运行时才能真正知道需要加载哪些模块。

## 副作用

我们知道纯函数是没有副作用的，只要给定相同的输入，输出一定也是相同。那么副作用是什么呢？副作用就是给定相同的输入，输出结果不一定，或者这个函数可能会影响外部变量或外部环境。官方给的定义是:

`在导入时会执行特殊行为的代码，而不是仅仅暴露一个 export 或多个 export`
`举例说明，例如 polyfill，它影响全局作用域，并且通常不提供 export`

因此，具有副作用的代码也会让`Tree Shaking`失效，解决办法可以参见[官方文档](https://webpack.docschina.org/guides/tree-shaking/)


## export default

```javascript
// ClassA.js
 class ClassA {
    run (){
        console.log("this is ClassA")
    }
}
export default ClassA

// ClassB.js
const ClassB = function(){}
ClassB.prototype.run = function(){
     console.log("this is ClassB")
 }

// main.js
import ClassA from "./ClassA";
import ClassB from "./ClassB";
```

打包后的结果，使用es6语法定义的ClassA被成功移除，使用prorotype定义的ClassB没有移除。这里代码是基于`babel 7`进行了转换

```javascript
// dist/main.js
function(e, t, n) {
    "use strict";
    n.r(t);
    var r = function() {};
    r.prototype.run = function() {
      console.log("this is ClassB");
    };
  }
```

那么上面的 ClassA 和 ClassB经过babel转换后的代码到底有什么区别呢？

```javascript
// ClassA.js
var ClassA =
  /*#__PURE__*/
  (function() {
    function ClassA() {
      _classCallCheck(this, ClassA);
    }
    _createClass(ClassA, [
      {
        key: "run",
        value: function run() {
          console.log("this is ClassA");
        }
      }
    ]);

    return ClassA;
  })();

 // ClassB.js
var ClassB = function ClassB() {};
ClassB.prototype.run = function () {
    console.log("this is ClassB");
};
```

可以看出ClassB.js的代码几乎是没有改变，而ClassA.js则被babel转换成了通过_createClass创建的方式，同时增加了注释` /*#__PURE__*/` . 
不难推断出，在原型链上定义函数其实是有副作用的。 再看个[例子](https://juejin.im/post/5a5652d8f265da3e497ff3de)：

```javascript
var V8Engine = (function () {
  function V8Engine () {}
  V8Engine.prototype.toString = function () { return 'V8' }
  return V8Engine
}())
var V6Engine = (function () {
  function V6Engine () {}
  V6Engine.prototype = V8Engine.prototype // <---- side effect
  V6Engine.prototype.toString = function () { return 'V6' }
  return V6Engine
}())
console.log(new V8Engine().toString())
```

`V6Engine虽然没有被使用，但是它修改了V8Engine原型链上的属性，这就产生副作用了`，因此，直接把V6Engine 给删了，其实是不对的。基于此，babel 7在代码上通过`/*@__PURE__*/`这样的注释声明此函数无副作用。  
另外也可以在package.json中配置 `sideEffects: false`，也可以达到效果.

## IIFE(立即调用函数表达式)

直接看代码

```javascript
// util.js
var square = (function(x) {
  console.log("square");
})();

export function cube(x) {
  console.log("cube");
  return x * x * x;
}
```

```javascript
// main.js
import { cube } from "./utils.js";
console.log(cube(2));
```

打包之后的结果：

```javascript
// dist/main.js
 function(e, t, r) {
    "use strict";
    r.r(t);
    var n;
    console.log("square"); // ifee函数没有被消除
    console.log(((n = 2), console.log("cube"), n * n * n));
  }
```

其实这个原因也比较好理解, 因为IIFE比较特殊，它在被翻译时(JS并非编译型的语言)就会被执行，Webpack不做程序流分析，它不知道IIFE会做什么特别的事情，所以不会删除这部分代码

## `Tree Shaking` 的应用

 `Tree Shaking` 除了消除无用的js之外，还可以：

- `Tree Shaking`不仅可以对 js 代码进行优化，css 同样也可以。可以使用[`Purgecss`](https://github.com/FullHuman/purgecss-webpack-plugin)这个插件删除无用的 css 代码

- `Tree Shaking`可以用于多平台 web 代码打包，主要原理是通过DefinePlugin注入环境变量，将不同平台的代码在编译阶段直接打包即可.

## 推荐阅读

[webpack 官方文档](https://webpack.docschina.org/guides/tree-shaking/)  
[你的 Tree-Shaking 并没什么卵用](https://juejin.im/post/5a5652d8f265da3e497ff3de)  
[Webpack Tree shaking 深入探究](https://juejin.im/post/5bb8ef58f265da0a972e3434#heading-7)  
[Webpack Tree shaking 深入探究](https://juejin.im/post/5bb8ef58f265da0a972e3434#heading-7)  
[体积减少80%！释放webpack tree-shaking的真正潜力](https://juejin.im/post/5b8ce49df265da438151b468)  
[基于Tree-shaking的多平台Web代码打包实践](https://mp.weixin.qq.com/s?__biz=MzI3NTU3ODc2MQ==&mid=2247484310&idx=1&sn=0968dd1deb4cff1f1a72ce5b5c95be3b&chksm=eb03e970dc74606692d8031f18a046232bc19a14f6bb8dccf517f79beaa5f9d7f185e2e63a0e&mpshare=1&scene=1&srcid=&sharer_sharetime=1582163072910&sharer_shareid=7e979d6f403d06ab8d710ddf48fc4aa2&exportkey=AZBsFzIc0g023vKuTk2sL3g%3D&pass_ticket=MjrQNeXBaQztWM6f%2BTKBVrWRmT87%2FwrtGjJI%2BwUlrGgkzQglOb243JhD4QTGZEtX#rd)
