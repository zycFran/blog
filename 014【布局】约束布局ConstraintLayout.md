# 约束布局

## Why?
从事前端开发，css布局相关的知识一定是必不可少的。前端主要工作的一部分就是**适配**页面内容在各种端设备上的显示效果，随着css的发展，也慢慢诞生出像Flex/Grid这样的自动布局方案。目前来看，Flex对于前端工程师来说应该是家喻户晓了，Grid貌似用的还不是很多。

目前各种端的布局适配主要采用的还是 rem 的方案。rem的原理：
> rem作用于非根元素时，相对于根元素字体大小；rem作用于根元素字体大小时，相对于其出初始字体大小(浏览器默认字体大小16px) 

rem的本质是一种基于屏幕宽度的等比缩放。rem也有一些弊端，其中最主要的就是字体大小不能使用rem,主要是因为字体的大小和字体宽度并不构成线性关系。（题外话：字体大小的同比缩放一般采用的是 em ） 

等比缩放在某些场景下也不适用，比如页面的头部和底部往往是需要固定高度，中间部分自适应，有的时候左侧菜单栏也需要固定宽度等。即页面只有部分需要自适应的情况下，等比缩放并不能解决所有的问题。这时候我们就需要另外一种布局方式：约束布局，ConstraintLayout

ConstraintLayout 最早是Google在2016年的 I/O大会上提出的，约束布局目前主要的作用场景也是在 Andriod/IOS 平台下默认支持，纯前端同学了解可能并不太多。

## What? 
ConstraintLayout即约束布局，这里的约束条件往往会使用数学关系进行表达。我们来看一个具体的例子：

![image](https://cdn.nlark.com/yuque/0/2020/png/86342/1578891603878-0f32b280-19e1-4b68-8eab-60a146cbb86e.png?x-oss-process=image%2Fresize%2Cw_746)
上图来源是 See Conf 第三届的分享，[精雕细琢，打造极致可视化图表体验](https://www.yuque.com/seeconf/2020/ysufx8) ，可以看出，在一个图表布局的场景中，可以将图表中的各个模块用一组数学表达式组合来表示，最后求解这个数学表达式就可以得到每个模块的pos(x,y,width,height)信息。

## How?

社区内已经有了相关解决方案，[AutoLayout.js](https://ijzerenhein.github.io/autolayout.js/)

AutoLayout.js  基于javascript实现了 IOS 上的Auto Layout约束布局系统及 [Visual Format Language](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage.html#//apple_ref/doc/uid/TP40010853-CH27-SW1)。 可以看一个简单的demo:

```js
var constraints = AutoLayout.VisualFormat.parse([
  'H:|[view1(==view2)]-10-[view2]|'
  'V:|[view1,view2]|'
], {extended: true});
var view = new AutoLayout.View({constraints: constraints});
view.setSize(400, 500);
console.log(view.subViews.view1); // {left: 0, top: 0, width: 195, height: 500}
console.log(view.subViews.view2); // {left: 205, top: 0, width: 195, height: 500}
```
上面的demo中创建了两个大小相同的view，间距为10

## 总结
最近关注前端可视化搭建这个方向比较多，布局系统在可视化搭建里面是一个比较有挑战性的问题。 
目前来看，约束布局可以提供一个很好的思路，将不同的view模块通过拖拽组合的方式配和一个配置性参数构建出约束性布局，进而实现页面的自动化布局。

未完待续...


## 参考
[精雕细琢，打造极致可视化图表体验](https://www.yuque.com/seeconf/2020/ysufx8) 

[UI高效布局实践](https://mp.weixin.qq.com/s/l6jnaxG3ySl3W1REQGCL6w?) 

[AutoLayout.js](https://ijzerenhein.github.io/autolayout.js/) 

[Visual Format Language](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage.html#//apple_ref/doc/uid/TP40010853-CH27-SW1) 
