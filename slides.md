---
theme: seriph
background: https://source.unsplash.com/collection/94734566/1920x1080
class: text-center
highlighter: shiki
lineNumbers: false
drawings:
  persist: false
css: unocss
title: React Hooks 使用分享
---

# React Hooks 使用分享

分享人：汪雪平

---

# 谈谈之前的类组件

在 v16.8 之前，组件的标准写法是 class，如下

```jsx
import React, { Component } from "react";

export default class Button extends Component {
  constructor() {
    super();
    this.state = { buttonText: "点击我" };
    this.handleClick = this.handleClick.bind(this);
  }
  handleClick() {
    this.setState(() => {
      return { buttonText: "已经被到点击" };
    });
  }
  render() {
    const { buttonText } = this.state;
    return <button onClick={this.handleClick}>{buttonText}</button>;
  }
}
```

可以看到，仅仅只是一个简单的 小按钮，就需要写很多代码

---

# Hooks 的含义

Hook 这个单词的意思是"钩子"。

React Hooks 的意思是，组件尽量写成纯函数，如果需要外部功能和副作用，就用钩子把外部代码"钩"进来。 React Hooks 就是那些钩子。

你需要什么功能，就使用什么钩子。React 默认提供了一些常用钩子，你也可以封装自己的钩子。

所有的钩子都是为函数引入外部功能，所以 React 约定，钩子一律使用 use 前缀命名，便于识别。你要使用 xxx 功能，钩子就命名为 usexxx。

下面介绍常用的内置钩子

---

# 常用的内置钩子

1. useState()
2. useEffect()
3. useCallback()
4. useRef()
5. useMemo()

---

# useState() 状态钩子

`useState()` 用于为函数组件引入状态（state）。纯函数不能有状态，所以把状态放在钩子里面

我们使用 useState 改造下开头的案例

```jsx
import { useState } from "react";

export default function Button() {
  const [buttonText, setButtonText] = useState("点击我");
  const handleClick = () => setButtonText("已经被到点击");
  return <button onClick={handleClick}>{buttonText}</button>;
}
```

可以看到代码量明显变少，且更加直观

---

# useEffect() 副作用钩子

useEffect() 用来引入具有副作用的操作，最常见的就是向服务器请求数据。

```js
useEffect(() => {
  // 执行请求
  return () => {
    // 组件卸载的时候执行
  }
}, [dependencies]);
```

上面的用法中，useEffect 接受两个参数，第一个参数是函数，可以理解为 callback
，第二个参数是一个数组，给出 Effect 的依赖性，只要这个数组中的内容发送变化，useEffect 就会重新执行

第二个参数可以省略，这样每次组件渲染就会执行 useEffect

如果第二个参数给一个空数组，则表示没有依赖性，则 useEffect 只会在组件第一次渲染的时候执行

useEffect 的第一个参数，可以返回一个函数，改函数会在组件卸载的时候执行

---

# useEffect 案例

实现翻页功能

```jsx
import { useState, useEffect } from "react";

export default function Page() {
  const [page, setPage] = useState(1);
  const [list, setList] = useState([]);
  useEffect(() => {
    fetch("http://xxx")
      .then((response) => response.json())
      .then((data) => {
        setList(data);
      });
  }, [page]);
  const handleNext = () => setPage((page) => page++);
  return (
    <>
      list: {list}
      <button onClick={handleNext}>下一页</button>
    </>
  );
}
```

每次点击就会更新 page ，随之重新触发 useEffect 的执行

---

# useCallback() 缓存函数

因为组件的每次重新渲染就会让函数重新执行一次，useCallback 就是用来解决 函数被重复创建的问题

比如上个案例中使用的 handleNext，每一次渲染都会重新创建一次

```js
const handleNext = useCallback(() => setPage((page) => page++), []);
```

这样 handleNext 函数就只会被创建一次

---

# useRef() 返回可变的对象

先看下面这段代码

```jsx
export default function App() {
  const [val, setVal] = useState(1);
  useEffect(() => {
    setTimeout(() => {
      console.log(val);
    }, 5000);
  }, []);
  return <button onClick={() => setVal(100)}>点击修改</button>;
}
```

进来之后立马点击以下，5s后 `console.log` 会打印什么？

https://codesandbox.io/s/bold-butterfly-w4op8z?file=/src/App.js

---

# 使用 useRef 改造下

```jsx
export default function App() {
  const ref = useRef(1);
  useEffect(() => {
    setTimeout(() => {
      alert(ref.current);
    }, 5000);
  }, []);
  return <button onClick={() => (ref.current = 100)}>点击修改</button>;
}
```

https://codesandbox.io/s/wizardly-gagarin-mf5245?file=/src/App.js

能够看出来 useRef 返回的值 每次拿到都是最新的，还要一点需要注意，修改 ref.current 不会触发重新渲染

像组件实例，dom 元素这种 需要每次拿到最新值的变量可以使用 useRef 来获取

```jsx
const inputRef = useRef(null)
<input ref={inputRef}/>
<input ref={val => ref.current = val}/>
```
---

# useMemo() 缓存值

```js
const [page] = useState(1);
const pageMsg = useMemo(() => `展示${page}`, [page]);
```

和 useEffect 一样，只有依赖发送变化的时候，才会重新执行回调函数，从而计算出新的内容

由于 react 的特点，不仅可以用来缓存变量，还可以缓存 Dom，用来优化渲染

```jsx
const ListDom = useMemo(() => <div>{list}</div>, [list]);
```

到这里常用的内置 hooks 都聊完了，实际上 react 还提供了 很多内置的 hooks，具体的可以查看官网

---

# 自定义 hooks

hooks 最大的优点是，能够自定义 hooks 来封装常见的逻辑

[如下，封装一个倒计时 hooks](https://codesandbox.io/s/funny-rgb-rp9rzq?file=/src/App.js:0-576)

```jsx
import { useState, useEffect, useRef } from "react";
function useCountDown(inCount) {
  const [count, setCount] = useState(inCount);
  const timerRef = useRef(0);
  useEffect(() => {
    timerRef.current = setInterval(() => setCount((val) => --val), 1000);
    return () => clearTimeout(timerRef.current);
  }, []);
  useEffect(() => {
    if (count <= 0) {
      clearTimeout(timerRef.current);
    }
  }, [count]);
  return count;
}
export default function App() {
  const count = useCountDown(10);
  return <div className="App">{count}</div>;
}
```

---

# 自定义hooks的规范

1. 使用 usexxx 命名规范
2. 一个 hooks 只做一件事，包括内置的 hooks，比如 useEffect 切记别把所有的事情都写在一个 useeffect 里面

[社区精选hooks](https://ahooks.js.org/zh-CN/hooks/use-request/index)

---

# 使用 hooks 实现全局状态

之前为了使用全局状态，需要引入 redux，mbox 等第三方库来实现

[模拟 redux 的全局状态方式](https://codesandbox.io/s/falling-platform-ytl11d?file=/src/App.js)