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
- import binding 是 immutable的

由上可以看出`ES6`的模块依赖关系是确定的，和运行时无关，在代码不运行的情况下就可以静态分析出不需要的代码。而`CommonJs`模块的是动态的，可以基于不同条件导入，因此只有在运行时才能真正知道需要加载哪些模块。

## 扩展

- `Tree Shaking`不仅可以对js代码进行优化，css同样也可以。可以使用[`Purgecss`](https://github.com/FullHuman/purgecss-webpack-plugin)这个插件删除无用的css代码，配置示例：




## 推荐阅读
[]