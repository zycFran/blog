
## 简要
- useState 会触发 re-render; useRef不会
- useState 和 useRef 都可以在re-render之后，记住值

## 为什么要了解 react re-render的触发时机？
- Re-render 会基于最新的 state 变化更新 ui
- Re-render 会触发 dom 的 reconciliation(协调)，有可能跟性能优化有关

## 什么会触发re-render
- 真正可以触发re-render的只有state change
看一个例子

```JSX
const Input = ({ value, onChange }) => (
  <input value={value} onChange={onChange} />
);
function App() {
  const [value, setValue] = React.useState("");
  const handleInputChange = e => {
    setValue(e.target.value);
  };
  return (
    <div className="App">
      <Input value={value} onChange={handleInputChange} />
      <Button>Button</Button>
    </div>
  );
}
```
通过react-tool-dev-utils发现，在用键盘在`input`中输入时，整个`app`组件都会re-render，即所有`app`的child组件也会`re-render`.
这样就会带来一个问题，明明只有input变化了，为什么`Button`也需要re-render, 这其实就是一次无效re-render，那么怎样让`Button`不会re-render

### 方法1
```JSX
const Input = () => {
  const [value, setValue] = React.useState("");
  const handleInputChange = e => {
    setValue(e.target.value);
  };
  return <input value={value} onChange={handleInputChange} />;
};
```
将input组件的state移至该组件内部，这样app内就不再会有state change，即Button不会跟着re-render.

### 方法2
我们需要思考一个问题，键盘输入的变化是不是一定需要触发input甚至一个组件的 re-render？
上面的value state让我们可以实时的收集input value的变化，但有时我们只需要在一次按钮点击的时候去收集input的value。

```JSX
function App() {
  const [value, setValue] = React.useState("");
  const valueRef = React.useRef();

  const handleClick = e => {
    setValue(valueRef.current.value);
  };

  return (
    <div className="App">
      <h4>Value: {value}</h4>
      <input ref={valueRef} />
      <Button onClick={handleClick}>Button</Button>
    </div>
  );
}
```
这里，input上不再绑定value, 我们再Button onClick时去setValue，这样，页面只会在setValue时，re-render一次。

## 总结
方法1 其实是给input绑定value属性，ui渲染时,input的值只能通过value值展示，此时，我们称input为一个受控组件。
对应的，在方法2 中，input不再绑定value，ui渲染不再受value这个props的影响，此时input是一个非受控组件。
需要注意的是，useRef的作用不仅仅是在re-render时缓存当前的值，同时它还能够缓存定时函数的状态，如 setInterval, debounce,throttle等.



[原文地址](https://www.codebeast.dev/usestate-vs-useref-re-render-or-not/#tldr)
