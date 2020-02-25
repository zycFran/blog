# 编写webpack plugin

提到 webpack plugin， 大家通常会将loader和plugin进行对比。
loader简单的说就是处理各种各样的静态资源。但是plugin更加强大，plugin可以说贯穿了整个webpack打包的过程，从初始化到最后资源的输出都会用到plugin。
plugin和loader还有一点区别就是，plugin没有像 loader-runner 那样的独立运行环境，只可以在webpakc环境中运行。

## 插件的基本结构

```javascript
class MyPlugin{
    apply(compiler){
        compiler.hooks.done.tap('MyPlugin', (stats)=>{
            // 处理插件逻辑
            console.log("hello world")
        })
    }
}
module.exports = MyPlugin
```

### 插件的使用

```javascript
plugins: [new MyPlugin()]
```

## 插件如何传参

通过构造函数进行参数传递

```javascript
module.exports = class MyPlugin{
    constructor(options){
        this.options = options
    }
    apply(){
        console.log(this.options);
    }
}
```

## 错误处理

错误处理分为两个阶段，第一个阶段是参数校验阶段，可以直接throw的方式抛出。

第二个阶段，已经进入hooks逻辑，通过 `compilation` 对象的warnings和errors接收

```javascript
compilation.warning.push("warning")

compilation.errors.push("error")
```

## 文件写入

和loaders一样，plugin同样也会生成一些资源需要进行文件写入。这时可以通过comilation 上的 assets 写入文件。代码示例：
>文件写入还需要 依赖[webpack-sources](https://github.com/webpack/webpack-sources)

```javascript
const {RawSource} = require("webpack-sources");
module.exports = class MyPlugin{
    constructor(options){
        this.options = options;
    }
    apply(compiler){
        const {name} = this.options;
        compiler.hooks("emit", (compilation, cb)=>{
            compilation.assets[name] = new RawSource("demo");
            cb();
        })
    }
}
```

webpack文件生成是在`emit`这个hook阶段生成的。这个时间，需要在plugin中监听这个hook，获取 `compilation` 这个对象，最后将资源内容设置到 assets 这个对象中即可。webpack最终在生成文件的时候，触发`emit`，就是读取compilation.assets内容，输出到磁盘目录。

## 编写插件的插件

我们除了可以通过插件来扩展webpack的能力，也可以用插件来扩展插件的能力。
其实现方式也不复杂，主要还是通过暴露hooks的方式进行自身扩展。这里面有一个比较经典的例子：html-webpack-plugin

- html-webpack-plugin-after-chunks (Sync)
- html-webpack-plugin-before-html-generation (ASync)
- html-webpack-plugin-after-assets-tags(ASync)
- html-webpack-plugin-after-html-processing(ASync)
- html-webpack-plugin-after-emit(ASync)

上面就是 html-webpack-plugin暴露出来的一些不同生命周期的Hooks，开发者可以基于此定制很多自己的业务逻辑。

## 实战

编写一个压缩构建资源为zip包的插件

要求

- 生成的zip包文件名称可以通过插件传入
- 需要使用compiler对象上的特点hooks进行资源生成

代码地址： [webpack-zip-plugin](https://github.com/zycFran/webpack-zip-plugin)

