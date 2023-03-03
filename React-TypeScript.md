# React 和 TypeScript
此文參考 [這篇](https://juejin.cn/post/7021674818621669389)
單獨寫 TS 時沒有遇到甚麼大問題，可是當 React 和 TS 結合時，就複雜很多，於是打算做一篇 React + TS 的筆記記錄問題  

這篇筆記建立在有 React 和 TS 的基礎  

可以在[codeSandbox](https://codesandbox.io/s/8w934)做線上測試

# React 組件
React 有兩種組件
1. 類組件
2. 函數組件
3. 
先來看看他們是怎麼搭配 TS 來使用的

## 1. 類組件
1. React.Component<P, S={}>
2. React.PureComponent<P, S={}. SS={}>
3. 
這兩個都是泛型，第一個參數是 props 類型的定義，第二個是 state 類型的定義，都不是必需的，可以省略

```ts
import * as React from "react";
import { render } from "react-dom";
// import { Component, PureComponent } from 'react';

interface IProps {
  name: string;
}
interface IState {
  count: number;
}
class App extends React.Component<IProps, IState> {
  constructor(props: IProps) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        {this.state.count}
        {this.props.name}
      </div>
    );
  }
}

const rootElement = document.getElementById("root");
render(<App name="thisWeb" />, rootElement);

```

`React.PureComponent<P, S={}, SS={}>` 也是差不多的   
不過他在 re-renders 時，相同的 props 和 state 會被跳過，所以效能較好  

不過有時候我們不知道 props 的類型，這時泛型就發揮功能了

```ts
// ./playgrounds/ClassComponent2.tsx
import * as React from "react";

interface IProps<P> {
  readonly name: P;
}

class MyComponent<P> extends React.Component<IProps<P>> {
  readonly name: P;

  constructor(props: IProps<P>) {
    super(props);
    this.name = props.name;
  }

  render() {
    return <div>Hello, {this.name}</div>;
  }
}

export default MyComponent;
```
```ts
// ./src/index.tex
import * as React from "react";
import { render } from "react-dom";

import ClassComponent2 from "./../playgrounds/ClassComponent2";

interface IComponent2Props {
  name: string;
}

const App = (props) => {
  return <ClassComponent2<IComponent2Props> name="thisWeb" />;
};

const rootElement = document.getElementById("root");
render(<App />, rootElement);

```

## 函數組件

一般來說函數組件可以這樣寫

```ts
interface IProps {
  readonly name: string;
}

const MyComponent = (props: IProps) => {
  const {name} = props;
  return (
    <div className="myClass">
      Hello, {name}
    </div>
  )
}

export default MyComponent
```

當然，函數類型還可以使用 `React.FunctionComponent<P={}>` 來定義，也可以使用簡寫 `React.FC<P={}>`，他也是一個泛型

```ts
import * as React from "react";

interface IProps {
  readonly name: string;
}

const MyComponent: React.FC<IProps> = (props: IProps) => {
  const { name } = props;
  return <div className="myClass">Hello, {name}</div>;
};

export default MyComponent;
```

不確定 props 的類型時一樣用泛型來解決
```ts
import * as React from "react";

interface IProps<P> {
  readonly name: P;
}

function MyComponent<P>(props: IProps<P>) {
  const { name } = props;

  return <div>Hello, {name};</div>;
}
export default MyComponent;
```
要注意的是，如果你使用箭頭函數會報錯，必須用 `extends` 關鍵字來定義泛型參數

```ts
const MyComponent = <P extends any>(props: IProps<P>) {
  const { name } = props;
  return (
  	<span>
    	{name}
    </span>
  );
}
```

# React 內置類型

## 1. JSX.Element

先來看看 JSX.Element 類型的聲明
```ts
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> { }
  }
}
```
1. declare global 告訴 TypeScript，這段程式碼是在宣告全域的型別定義
2. name space JSX 宣告一個 JSX  命名空間
3. interface Element extends React.ReactElement<any, any> 告訴 TS，JSX 元素的類型繼承自 React.ReactElement，並接受任何類型的 prop 和 children
4. { } 表示 JSX 命名空間中沒有任何其他定義  

可以發現 JSX.Element 是 ReactElement 的子類型，而且沒有增加屬性，也就是說它們是等價的  

JSX.Element 通常由編譯 JSX 或執行 React.createElement 獲得

```ts
const jsx = <div>hello</div>
const ele = React.createElement("div", null, "hello");
<p> // <- ReactElement = JSX.Element
  <Custom> // <- ReactElement = JSX.Element
    {true && "test"} // <- ReactNode
  </Custom>
</p>
```

## 2. ReactElement
在來看看 `ReactElement` 是甚麼

```ts
interface ReactElement<P = any, T extends string | JSXElementConstructor<any> = string | JSXElementConstructor<any>> {
   type: T;
   props: P;
   key: Key | null;
}
``` 
可以看出 React.ReactElement 是一個 interface，包含 type、porps、key 三個屬性值

而 `T extends string | JSXElementConstructor<any> = string | JSXElementConstructor<any>` 用 extends 關鍵字表示 T 的類型必須符合 string 或`JSXElementConstructor` ，而等號後面表示默認值是 stirng 或 `JSXElementConstructor`

`JSXElementConstructor` 是 React 中的一個類型，用來表示 React 元素的建構函數  


## 3. React.ReactNode
```ts
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;

interface ReactNodeArray extends Array<ReactNode> {}
type ReactFragment = {} | ReactNodeArray;
type ReactNode = ReactChild | ReactFragment | ReactPortal | boolean | null | undefined;
```
ReactNode 是一個聯合類型，可以是 string、number、ReactElement、null、boolean、ReactNodeArray，所以 ReactElement 類型的變量可以直接賦值給 ReactNode，但反過來不行

```ts
class MyComponent extends React.Component {
	render() {
    	return <div>hello world</div>
    }
}
// 正确
const component: React.ReactNode<MyComponent> = <MyComponent />;
// 錯誤
const component: React.ReactNode<MyComponent> = <OtherComponent />;
```

# React Hooks

## 1. useState
React 會根據設置的 state 初始值來自動推導 state 的類型  
但也可以像下面這樣來自定義 state  

```ts
const [conut, setCount] = useState<number>(1);

// 若初始值為 null 需要顯示聲明 state 類型
const [count, setCount] = useState<number | null>(null);
```

## 2. useEffect
useEffect 主要作用是處理 side effect function，第一個參數是函數，第二個參數是一組值，這組值改變時，第一個參數的函數才會執行  
```ts
useEffect(() => {
  const subscription = props.source.subscirbe();
  return () => {
    subscription.unsubscribe();
  }
}, [props.source])
```
當函數返回值不是函數或是 effect 函數定中為定義的內容時，就會報錯
```ts
useEffect(() => {
  subscribe();
  return null
})
```

## 3. useRef
useRef 主要用來獲取 React 元素的引用，包括 DOM 元素、React 組件  
```ts
interface InputProps {
  value: string;
  onChange: (value: string) => void;
}

function MyInput(props: InputProps) {
  const inputRef = useRef<HTMLInputElement>(null);

  const handleChange = () => {
    if (inputRef.current) {
      props.onChange(inputRef.current.value);
    }
  };

  return <input type="text" ref={inputRef} value={props.value} onChange={handleChange} />;
}

```