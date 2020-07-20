# react hooks vs class size compare

hooks 出來過後，開始有人把 class refact 成 hooks  
其中一個優點是說 size 變小  
我有點忘記了，到底有多少差距？  
這邊自己做個實驗看看  

使用官方範例文件
- https://zh-hant.reactjs.org/docs

## hooks 的程式碼
```js
import React, { useState } from 'react';

function Example() {
  // 宣告一個新的 state 變數，我們稱作為「count」。
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

```js
import React, { useState, useEffect } from 'react';

function Example2() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

## class 的程式碼
```js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

```js
class Example2 extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }
  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

## result
![hooks](/assets/img/react_hooks_bundle_size.jpg)  
![class](/assets/img/react_class_bundle_size.jpg)  

`2.chunk` 跟 `main.chunk` 的差異
- 39880 B - 39770 B = **110 B** 的差距
- 749 B - 638 B = **111 B**

**這兩段程式碼，造成的差距為約共 200 Bytes**  

這很難作為未來開發的判斷，總之就是給自己一個參考