# Scope Hoisting原理

webpack3.0开始支持 `scope hoisting`，作为优化代码的一种手段。在webpack4.0之后，设置mode为production模式则会默认开启 `scope hositing`.
那么 `scope hositing`到底是什么呢？在回答这个问题之前，我们先看下webpack打包之后的bundle.js是怎么样的

```javascript
import action from './other-module.js';

var value = action();

export default value;
```

上面的代码经过webpack打包之后，会变成如下的闭包结构：

```javascript
(function(module, exports, WEBPACK_REQUIRE_METHOD) {
 'use strict';
  
  var action = WEBPACK_REQUIRE_METHOD(1);
  var value = action();

  exports.default = value;
});
```

可以看到，原代码转换成一个闭包结构，`import`导入的变量转换成 `WEBPACK_REQUIRE_METHOD`函数调用，这样隔离了作用域之后，保证各模块之间的变量不会互相干扰。一个简单的bundle.js会大概会是下面这个样子

```javascript
(function(modules) {
  var installedModules = {};

  // 替换 import 的代码
  function WEBPACK_REQUIRE_METHOD(id) {
    // if module was already imported, return its exports
    if (installedModules[id]) {
      return installedModules[id].exports;
    }

     // create module object and cache it
     var module = installedModules[id] = {
       id: id,
       exports: {}
     };

     // call module’s function wrapper
     modules[id](module, module.exports, WEBPACK_REQUIRE_METHOD);
  }

  // 调用 entry 模块
  WEBPACK_REQUIRE_METHOD(0);
})([
  /* 0 module */
  function() {},
  /* 1 module */
  function() {},
  /* n module */
  function() {}
]);
```

整个bundle.js就是一个自执行函数，在函数的最开始定义一个变量缓存了所有已经加载过的模块，然后申明一个方法替代了模块中的import，最后执行entry模块的代码。所有被import依赖的模块会被合并成一个数组传递给整个自执行函数。

下面假设一个场景：一个模块依赖了某个方法A，然后方法A又依赖方法B，方法B又依赖方法C...就是说依赖链非常的长。那么可以想象的出打包之后的bundle.js，module 数组将会很大，并且每个module都将被一段代码包裹成一个闭包调用，我们知道闭包是非常浪费内存的，同时每个依赖的方法都要通过一个`WEBPACK_REQUIRE_METHOD`执行一下，也会比较耗时，最后导致整个代码执行的性能就比较低。

解决办法就是 `scope hoisting`：即把那些只被依赖了一次的方法直接注入到被依赖的模块中，而不再被单独打包成一个模块。尽量减少了闭包调用。

```javascript
(function(){
  'use strict';

  var helper = WEBPACK_REQUIRE_METHOD(0);

  var action = function() {
    var value = helper();
    return value;
  };

  exports.action = action;
});
```

`scope hoisting`后的效果：

```javascript
(function() {
  'use strict';

  function helper() {
    /* inlined function from module */
  }

  var action = function() {
    var value = helper();
    return value;
  };

  exports.action = action;
});
```

不仅减少了内存，而且执行效率也更高了。

## 如何使用

- webpack4中直接开启 mode: production 则会默认开启`scope hoisting`

- 配置 `ModuleConcatenationPlugin` 这个插件即可:   
`new webpack.optimize.ModuleConcatenationPlugin()`

## 参考阅读

[more about scope hoisting ](https://medium.com/webpack/brief-introduction-to-scope-hoisting-in-webpack-8435084c171f)

