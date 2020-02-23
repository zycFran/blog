# 编写复杂的Typescript类型

题目链接： [传送门](https://github.com/LeetCode-OpenSource/hire/blob/master/typescript_zh.md)

比较核心的就是下面这句话

> 现在有一个叫 connect 的函数，它接受 EffectModule 实例，将它变成另一个一个对象，这个对象上只有EffectModule 的同名方法，但是方法的类型签名被改变`

意思就是 提取类 `EffectModule` 中的Function属性，并将其转换一下类型

```typescript
asyncMethod<T, U>(input: Promise<T>): Promise<Action<U>>  变成了
asyncMethod<T, U>(input: T): Action<U>

syncMethod<T, U>(action: Action<T>): Action<U>  变成了
syncMethod<T, U>(action: T): Action<U>
```

EffectModule 定义如下:

```typescript
interface Action<T> {
  payload?: T;
  type: string;
}

class EffectModule {
  count = 1;
  message = "hello!";

  delay(input: Promise<number>) {
    return input.then(i => ({
      payload: `hello ${i}!`,
      type: 'delay'
    });
  }

  setMessage(action: Action<Date>) {
    return {
      payload: action.payload!.getMilliseconds(),
      type: "set-message"
    };
  }
}
```

## Step1

那么按照题目的要求，第一步我们需要提取类的Function

``` typescript
type FunctionKeys<T> = {
    [k in keyof T]: T[k] extends Function? K:never
}[keyof T]
```

为了更加清楚，分解一下

```typescript
type TypesOfT = keyof T
// 把 T替换成 EffectModule
// type TypesOfT =  "count" | "message" | "delay" | "setMessage"

type Functions = {
    [k in keyof T]: T[k] extends Function? k:never
}
/**
type Functions = {
    count: never;
    message: never;
    delay: "delay";
    setMessage: "setMessage";
}
*/

type FunctionKeys<T> = Functions[TypesOfT] = {
    count: never;
    message: never;
    delay: "delay";
    setMessage: "setMessage";
}["count" | "message" | "delay" | "setMessage"]

// type FunctionKeys = "delay" | "setMessage"
```

## Step2

转换Function的类型, 首先把转换后的类型都写出来

```typescript
type asyncMethod<T, U> = (input: Promise<T>) => Promise<Action<U>>
type asyncMethodConnect<T, U> = (input: T) => Action<U>

type syncMethod<T, U> = (action: Action<T>) => Action<U>
type syncMethodConnect<T, U> = (action: T) => Action<U>
```

由于题目限定了`EffectModule`类中只包含`asyncMethod` 和 `syncMethod`两种类型的方法，因此可以采用`条件类型`的方法进行转换即可

```typescript
type MethodsConnect<U> = U extends asyncMethod<infer X, infer Y>
    ? asyncMethodConnect<X, Y>
    : U extends syncMethod<infer X, infer Y>
    ? asyncMethodConnect<X, Y>
    : nevers
```

`infer` 关键字的作用是 从一个方法反解其返回类型，比如可以从 `Promsie<T>` 中反解出 `T`。
详细用法可以参见[这里](https://jkchao.github.io/typescript-book-chinese/tips/infer.html)

## Step3

```typescript
type connect = (T: EffectModule)=> {
    [k in FunctionKeys<EffectModule>]:  MethodsConnect<EffectModule[k]>
}
/**
 type connect = (T: EffectModule) => {
    delay: asyncMethodConnect<number, string>;
    setMessage: asyncMethodConnect<Date, number>;
}
*/
```
