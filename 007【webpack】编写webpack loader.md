# 编写webpack loader

## 什么是 loader

loader 是一个导出为函数的javascript模块，接受一个输入source，经过loader的处理之后，返回一个新的source，下面展示的就是一个最简单的loader结构

```javascript
module.exports = function(source){
    return source;
}
```

## loader的执行顺序：`从后到前`

```javascript
module.exports = {
    ...
    module: {
        rules: [
            test: /\.less$/,
            use: ['style-loader', 'css-loader', 'less-loader']
        ]
    }
}
```

这里的执行顺序是:  `less-loader`->`css-loader`->`style-loader`

那么loader为什么要按照这种顺序执行呢？
函数组合一般有两种情况：

- Unix中的pipeline, 从左往右
- 函数中的Compose,从右往左

```javascript
compose = (f,g)=>(...args) =>f(g(...args));
```

webpack为了更好的满足函数编程的习惯，因此采用了compose的方式。其实类似 pipeline 从左往右的模式也是很容易的。

## loader-runner

[loader-runner](https://github.com/webpack/loader-runner)允许你在不安装webpack的情况下运行loaders，简单的说就是loader的执行环境，一般用来loaders的开发和调试。load-runner的用法很简单

```javascript
import { runLoaders } from "loader-runner";

runLoaders({
    // 资源的绝对路径（可以增加查询字符串）
    resource: "/abs/path/to/file.txt?query",
    // loader的绝对路径（可以增加查询字符串）
    loaders: ["/abs/path/to/loader.js?query"],
    // 基础上下文之外的loader上下文,
    context: { minimize: true },
    //读取资源的函数
    readResource: fs.readFile.bind(fs)
}, function(err, result) {
    ...
})
```

## loader参数的获取

我们知道在loader的使用过程中，经常是需要传递参数的，比如 `px2rem-loader` 就需要一个 `remUnit`的参数，那么在编写的loader的时候，怎么传参呢？这时就需要用到 `loader-utils` 这个神器
通过[loader-utils](https://github.com/webpack/loader-utils)的`getOptions`获取

```javascript
const loaderUtils = require('loader-utils');

module.exports = function(source){
    const options = loaderUtils.getOptions(this);
    return source;
}
```

## loader的异常处理

### 同步loader

同步loader的异常处理有两种方式：

- loader内直接通过throw抛出
- 通过 `this.callback` 传递错误,
this.callback是loader上下文中提供的函数

```javascript
this.callback({
    err: Error | null,
    content: string | Buffer,
    sourceMap?: SourceMap,
    meta? : any
})
```

```javascript
module.exports = function(source){
    const options = loaderUtils.getOptions(this);

    // 第一种方式 直接return
    // return source;

    // 第二种方式，throw Error
    // throw new Error('error')

    // 第三种方式，this.callback
    this.callback(new Error('error'), source)
}
```

> 一个细节： this.callback可以返回多个值，如： this.callback(null, source,1,2,3)；如果使用return的话，是支持返回一个值

### 异步loader

上面说的是同步loader的用法，实际过程中异步loader也是必不可少的，如果loader里要读取一个文件，或者执行一个耗时任务，都需要异步loader来实现。
其实异步loader的处理也很简单，通过 this.async 来返回一个异步函数。示例：

```javascript
module.exports = function(input){
    const callback = this.async();

    doReadFile(filePath).then(()=>{
            callback(null, input)
    })
}
```

## loader缓存

webpack4.x中已经默认开启了 loader 缓存，也可以通过手动的方式关闭缓存

```javascript
module.exports = function(input){
    this.cacheable(false);
}
```

缓存开启的前提条件：

- loader的结果在相同的输入下有相同的输出
- loader没有依赖其他loader

## loader如何进行文件输出

loader除了可以输出字符串之外，有时候还需要文件内容的转换再输出文件，比如 file-loader 就会输出一个换换之后的文件，那么loader里需要如何输入文件呢？

可以通过 this.emitFile 进行文件写入

```javascript
const loaderUtils = require('loader-utils')
module.exports = function(content){
    // 自动替换占位符的值，获取完整的文件路径
    const url = loaderUtils.interpolateName(this,'[hash].[ext]', {
        content
    });

    this.emitFile(url, content);
}
```

## 实战

编写一个自动合成雪碧图的loader，支持的语法：

```javascript
background: url('a.png?__sprite')
background: url('b.png?__sprite')
```

转换成

```javascript
background: url('sprite.png')
```

代码地址： [webpack-sprite-loader](https://github.com/zycFran/webpack-sprite-loader)