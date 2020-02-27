# 自制简版webpack

## webpack打包后的代码结构

- 打包出来的是一个 IFEE 闭包结构
- modules是一个数组, 每一项是一个模块初始化函数
- __webpack_require__用来加载模块，返回module.exports
- 通过 __webpack_require__(0)来启动程序

```javascript
(function (modules){
    var installedModules = {};
    function __webpack_require__(moduleId){
        if(installedModules[moduleId]){
            return installedModules[moduleId].exports
        }
        var module = installedModules[moduleId] = {
            i: moduleId,
            l: false,
            exports: {}
        };
        modules[moduleId].call(module.exports, module, module.exports,__webpack_requre__);
        module.l = true;
        return module.exports;
    }
    __webpack_require__(0)
})([
    /* 0 module*/
    (function (module, __webpack_exports__, __webpack_require__)){
        ...
    },
    /* 1 module*/
    (function (module, __webpack_exports__, __webpack_require__)){
        ...
    },
    /* 2 module*/
    (function (module, __webpack_exports__, __webpack_require__)){
        ...
    }
])
```

## 计划实现目标

- [x]  可以将 ES6 语法转换成 ES5
思路：
    - 通过 babylon 生成 AST
    - 通过 babel-core将 AST 重新生成源码

- [x] 支持分析模块之间的依赖关系
思路：
    - 通过 babel-traverse 的ImportDeclaration方法获取依赖属性

- [x] 生成的JS文件可以在浏览器中直接运行

## 实战


代码地址：[simple-webpack](https://github.com/zycFran/simple-webpack)