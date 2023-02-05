# typescript note 前言 
這裡紀錄學習 ts 的筆記，和遇到的問題  

# 1. 原始資料型別
JS 的型別分為兩種
1. 原始資料型別(Primitive data types)：包含 
   * boolean 布林值
   *  
   * string
   * null
   * undefined
   * symbol
1. 物件型別(object types)

在 ts 可以向下面這樣定義變數的型別
```ts
let isDone: boolean = false;

let num: number = 2;
let notANum: number = NAN;
let infinityNum: number = infinity;

let str: string = 'hello world';
```
在定義型別後便不能賦予其他型別給變數  

## void、null、undefined
儘管 JS 沒有空值(void)的概念，在 TS 中，仍可以用 `void` 表示**沒有任何返回值的函式**
```ts
function alertName(): void {
  alert('hello world');
}
```
順便示範了函式如何使用 ts 來定義返回值的型別  

## Null 和 Undefined
```ts
let u: undefined = undefined;
let n: null = null
```
undefined 和 null 是所有型別的**子型別**  
也就是說 nudefinde 和 null 可以賦值給其他型別的變數  
```ts
let num: number = undefined;
```
而 void 不行

## 任意值 any
任意值來表示允許變數為任意型別
```js
let myFavoriteNum: any = 'seven';
myFavoriteNum = 7;
```
在任意值上訪問任何屬性和呼叫任何方法都是被允許的

## 未宣告型別的變數
變數在宣告且**未賦值**的情況會被定義為任意型別(any)  
但若有賦值的情況下會被自動判斷型別
```ts
let num;
num = 1;
num = 'one'

let str = 1;
str = 'one' // 錯誤
```

## 聯合型別 Union Types
聯合型別表示取值可以為多種型別中的一種
```ts
let myFavoriteNum: number | string;
myFavoriteNum = 'seven';
myFavoriteNum = 7;
```
要注意的是  
當 ts 不確定變數到底是哪一種型別時  
我們只能使用聯合型別中共同有的屬性或方法
```ts
function getLength(something: string | number) {
  return something.length;
}
// 錯誤
```
上面 length 不是 string 和 number 


# 物件的型別-介面 interface
上面試介紹原始型別定義型別的方法  
而物件型別定義型別的方法比較不一樣  
是用 interface  
```ts
interface Person {
  name: string;
  age: number;
}

let tom: Person = {
  name: 'tom',
  age: 25
}
```
介面習慣第一個字大寫  
除此之外，儘管介面寫起來有點像是物件  
不過屬性之間要用 `;` 區隔，而非 `,`

要注意的是  
定義變數的屬性不能比介面少或多
```ts
interface Person {
  name: string,
  age: number,
}

let tom: Person = {
  name: 'tom',
}
// 錯誤

let rose: Person = {
  name: 'Rose',
  age: 18,
  gender: 'female'
}
// 錯誤
```
**賦值時，變數的形狀必須和界面的形狀保持一致**

## 可選屬性
但有時候我們希望不要完全匹配一個形狀，就可以使用可選屬性  
使用方法是多加一個 `?`
```ts
interface Person {
  name: string,
  age?: number,
}

let tom: Person = {
  name: 'tom',
}
```
不過仍是不允許多加未定義的屬性

## 任意屬性
有時候我們希望介面允許有任意的屬性  
```ts
interface Person {
  name: string;
  [propName: string]: any;
}

let tom: Person = {
  name: 18,
  gender: 'male',
}
```
要注意的是這是固定寫法，且 propName 也只能是 string  
此外，一旦定義了任意屬性，所有可選屬性和確定屬性也必須要是任意屬性型別的子集：
```ts
interface Person {
  name: string;
  age?: number;
  [propName: string]: string;
}
// 錯誤，age 必須也要是 string 或他的子集
```
## 唯讀屬性
有時候我們希望物件中的一些屬性只能在建立時被賦值
```ts
interface Person {
  readonly id: number;
  name: string;
}
```
給唯讀屬性賦值的機會只有在宣告物件的時候  
後來的賦值都是不允許的
```ts
interface Person {
  readonly id: number;
  name: string;
  age: number;
  [propName: string]: any;
}
let tom = {
  name: 'tom',
  age: 18
}
tom.id = 2
// 錯誤，只能

let tom = {
  id: 2,
  name: 'tom',
  age: 18
}
```
# 陣列的型別
```ts
let arr1: number[] = [1, 2, 3];

// 也可使用 陣列泛型
// 泛型可以參考後面的介紹
let arr2: Array<number> = [1, 2, 3]
```
除此之外也可以用介面表示陣列  
```ts
interface StrArr {
  [index: number]: string
}
```
不過這樣比較複雜，一般用在類陣列(例如：arguments)較多，像是
```ts
function sum(): string {
  let args: {
    [index: number]: number;
    length: number;
    callee: functionl
  } = arguments;
}
```
事實上，常用的類陣列都有自己的介面定義，例如  
1. IArguments
2. NodeList
3. HTMLCollection
等，所以可以改成
```ts
let sum(): string {
  let args: IArguments = arguments;
}
```

當然，陣列也可以使用 `any`
```ts
let arr: any[] = [1, 2, 3, '1', '2', '3', false]
```
或是聯合型別
```ts
let arr: (string | number)[] = ['1', 1];
```

# 函式的型別
函式是 JS 中的**一等公民**  
且有兩種常用的定義函式的方法
```ts
// 函示宣告 (function Declaration)
function sum(x, y) {
  return x + y;
}

// 函式表達式 (Function Expression)
const mySum = function(x, y) {
  return x + y;
}
```
因為函式會有輸入跟輸出，所以在 ts 中必須對兩個都進行約束  
必且在使用時是不允許輸入多餘(或少於要求)的參數
```ts
function sum(x: number, y: number): number {
  return x + y;
}

// 以下錯誤
sum(1, 2, 3);
sum(1);
```
函式表示式約束型別的方式比較特別
```ts
let mySum: (x: number, y: number) => number = function(x: number, y: number): number {
  return x + y;
}
```
這樣才能對等號左邊的變數以及等號右邊的匿名函式都進行型別定義  
不過若等號左側沒有寫型別的定義， ts 也會透過賦值操作進行型別推論  
要注意這裡的箭頭並不是箭頭函式  

除此之外，函式中的參數也可以有可選參數，使用方法一樣加 `?`，但可選參數不能放在必要參數前面

## 剩餘參數
可以用陣列的方式定義剩餘參數，因為它本身就是陣列
```ts
function push(array: any[], ...items: any[]) {
  items.forEach((item) => {
    array.push(item)
  })
}
```

## 過載
過載允許一個函式接受不同數量或型別的參數時，做出不同的處理  
你可能會想到用聯合型別
```ts
function reverse(x: number | string): number | string {
  if (typeof x === 'number') {
    return Number(x.toString().split().reverse().join(''));
  } else if (typeof x === 'string') {
    return x.split('').reverse().join('');
  }
}
```
然而這樣有一個缺點，就是不能夠精確的表達，輸入為數字的時候，輸出也應該為數字，輸入為字串的時候，輸出也應該為字串。  

這時我們可以使用過載來定義多個 reverse 函式型別
```ts
function reverse(x: number): number;
function reverse(x: string): string;
function reverse(x: number | string): number | string {
    if (typeof x === 'number') {
        return Number(x.toString().split('').reverse().join(''));
    } else if (typeof x === 'string') {
        return x.split('').reverse().join('');
    }
}
```
注意，TypeScript 會優先從最前面的函式定義開始匹配，所以多個函式定義如果有包含關係，需要優先把精確的定義寫在前面。

# 型別斷言 Type Assertion 
型別斷言 可以手動指定一個值的型別
```ts
<型別>值
// 或
值 as 型別
```
在 tsx 語法（React 的 jsx 語法的 ts 版）中必須用後一種。

之前提到過，當 TypeScript 不確定一個聯合型別的變數到底是哪個型別的時候  
**我們只能訪問此聯合型別的所有型別裡共有的屬性或方法：
**
```ts
function getLength(something: string | number): number {
    return something.length;
}
// 錯誤
```
但有時候，我們需要在還不確定型別的時候就訪問其中一個型別的屬性或方法，此時就可以用型別斷言來幫變數預設型別
```ts
function getLength(something: string | number): number {
    if ((<string>something).length) {
        return (<string>something).length;
    } else {
        return something.toString().length;
    }
}
```
注意，儘管可以預設型別，但仍不能斷言成聯合型別以外的型別

# 型別別名 和 字串字面量型別
型別別名用來給型別起新名字，常用於聯合型別
```ts
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
    if (typeof n === 'string') {
        return n;
    } else {
        return n();
    }
}
```
字串字面量 (String Literal) 型別用來約束取值只能是某幾個字串中的一個。

```ts
type EventNames = 'click' | 'scroll' | 'mousemove';
function handleEvent(ele: Element, event: EventNames) {
    // do something
}

handleEvent(document.getElementById('hello'), 'scroll');  // 沒問題
handleEvent(document.getElementById('world'), 'dbclick'); // 報錯，event 不能為 'dbclick'
```

注意，**型別別名**與**字串字面量型別****都是使用 type 進行定義**

# 元組
數組合並了相同型別的物件，若使用聯合型別或`any`可以合併多種型別
而元組（Tuple）可以合併不同型別的物件，且固定它們的位置，注意也不能進行增刪  
但可以透過函式操作元組，但仍不能新增額外的型別
```ts
let tom: [string, number] = ['Tom', 25];
tom = [24, 'tom'] // 錯誤
tom[2] = 'shan' // 錯誤
tom.push('shan') // 可以
tom.push(true) // 錯誤
```

# 列舉 enum
列舉用在型別用於取值被限定在一定範圍內的場景  
比如一週只能有七天，顏色限定為紅綠藍等。
```ts
enum Days {Sun, Mon, Tue, Wed, Thu, Fri, Sat};
```
列舉成員會被賦值為從 0 開始遞增的數字，同時也會對列舉值到列舉名進行反向對映：
```ts
enum Days {Sun, Mon, Tue, Wed, Thu, Fri, Sat};

console.log(Days["Sun"] === 0); // true
console.log(Days["Mon"] === 1); // true
console.log(Days["Tue"] === 2); // true
console.log(Days["Sat"] === 6); // true

console.log(Days[0] === "Sun"); // true
console.log(Days[1] === "Mon"); // true
console.log(Days[2] === "Tue"); // true
console.log(Days[6] === "Sat"); // true
```
也可以手動賦索引值
```ts
enum Days {Sun = 7, Mon = 1, Tue, Wed, Thu, Fri, Sat};

console.log(Days["Sun"] === 7); // true
console.log(Days["Mon"] === 1); // true
console.log(Days["Tue"] === 2); // true
console.log(Days["Sat"] === 6); // true
```
不過若手動賦值和列舉像遞增重複的話，會被後面的蓋過去
```ts
enum Days {Sun = 3, Mon = 1, Tue, Wed, Thu, Fri, Sat};

console.log(Days["Sun"] === 3); // true
console.log(Days["Wed"] === 3); // true
console.log(Days[3] === "Sun"); // false
console.log(Days[3] === "Wed"); // true
```
## 字串列舉 string enums
字串 enums 沒有 number enums 可以自動遞增的行為
```ts
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}
```

# 類別
TS 除了實現所有 es6 中 class 的功能，還新增了一些新的用法  

這裡先來講講類別的概念
* 類別(Class)：定義了一件事物的抽象特點，包含它的屬性和方法
* 物件（Object）：**類別的實例**，透過 new 產生
* 面向物件（OOP）的三大特性：**封裝、繼承、多型**
* 封裝（Encapsulation）：將對資料的操作細節隱藏起來，只暴露對外的介面。外界呼叫端不需要（也不可能）知道細節，就能透過對外提供的介面來訪問該物件，同時也保證了外界無法任意更改物件內部的資料
* 繼承（Inheritance）：子類別繼承父類別，子類別除了擁有父類別的所有特性外，還有一些更具體的特性
* 多型（Polymorphism）：由繼承而產生了相關的不同的類別，對同一個方法可以有不同的響應。比如 Cat 和 Dog 都繼承自 Animal，但是分別實現了自己的 eat 方法。此時針對某一個實例，我們無需瞭解它是 Cat 還是 Dog，就可以直接呼叫 eat 方法，程式會自動判斷出來應該如何執行 eat
* 存取器（getter & setter）：用以改變屬性的讀取和賦值行為
* 修飾符（Modifiers）：修飾符是一些關鍵字，用於限定成員或型別的性質。比如 public 表示公有屬性或方法
* 抽象類別（Abstract Class）：抽象類別是供其他類別繼承的基底類別，抽象類別不允許被實例化。抽象類別中的抽象方法必須在子類別中被實現
* 介面（Interfaces）：不同類別之間公有的屬性或方法，可以抽象成一個介面。介面可以被**類別實現（implements）**。一個類別只能繼承自另一個類別，但是可以實現多個介面

## TS 中類別的用法
* public: 可以在任何地方備訪問到，預設所有的屬性和方法都式 public
* private: 不能在宣告它的類別的外部訪問
* protected: 和 private 類似，但他允許在子類別中被訪問 (可以被繼承)
* readonly: 只能在宣告時賦值，注意若它和其他修式符同時存在，需要寫在後面
* abstract(抽象類別): 他是類本身的修飾符，使類不允許被實例化，他必須被子類別實現
```ts
abstract class Animal {
    public name;
    public constructor(name) {
        this.name = name;
    }
    public abstract sayHi();
}

class Cat extends Animal {
    public eat() {
        console.log(`${this.name} is eating.`);
    }
}

let cat = new Cat('Tom');
// 錯誤，因為沒有實現抽象類別的方法 sayHi()
```
上面是錯誤例子，下面是正確
```ts
abstract class Animal {
    public name;
    public constructor(name) {
        this.name = name;
    }
    public abstract sayHi();
}

class Cat extends Animal {
    public sayHi() {
        console.log(`Meow, My name is ${this.name}`);
    }
}

let cat = new Cat('Tom');
```

# 類別與介面
實現（implements）是面向物件中的一個重要概念。  
一般來講，一個類別只能繼承自另一個類別，有時候不同類別之間可以有一些共有的特性  
這時候就可以把特性提取成介面（interfaces），用 implements 關鍵字來實現。這個特性大大提高了面向物件的靈活性。

舉例來說，門是一個類別，防盜門是門的子類別。  
如果防盜門有一個報警器的功能，我們可以簡單的給防盜門新增一個報警方法。  
這時候如果有另一個類別，車，也有報警器的功能，就可以考慮把報警器提取出來，作為一個介面，防盜門和車都去實現它：
```ts
interface Alarm {
  alert();
}
class Door {}
class SecurityDoor extends Door implements Alarm {
  alert() {
    console.log('SecurityDoor alert');
  }
}

class Car implements Alarm {
  alert() {
    console.log('Car alert');
  }
}
```
一個類別可以實現多個介面
```ts
interface Alarm {
    alert();
}

interface Light {
    lightOn();
    lightOff();
}

class Car implements Alarm, Light {
    alert() {
        console.log('Car alert');
    }
    lightOn() {
        console.log('Car light on');
    }
    lightOff() {
        console.log('Car light off');
    }
}
```
此外，介面和類一樣是可以被繼承的
```ts
interface Alarm {
    alert();
}

interface LightableAlarm extends Alarm {
    lightOn();
    lightOff();
}
```
介面也可以繼承類別
```ts
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z: number;
}

let point3d: Point3d = {x: 1, y: 2, z: 3};
```
# 泛型 Generics
是指在定義函式、介面、類別時，不預先指定具體的型別，而是在使用時在指定型別的一種特性  
比如說：
```ts
function createArray<T>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}

createArray<string>(3, 'x'); // ['x', 'x', 'x']
```
上面我們在函式名後面添加 `<T>`，其中 T 用來指代任意輸入的型別，並給後面的 `value: T` 和輸出 `Array<T>` 中使用

## 多個型別參數
定義泛型時，可以一次定義多個型別的參數
```ts
function swap<T, U>(tuple:[T, U]): [U, Y] {
  return [tuple[1], tuple[0]];
}
swap([7, 'seven']); // ['seven', 7]
```
## 泛型約束
在函式內部使用泛型變數的時候，由於事先不知道它是哪種型別，所以不能隨意的操作它的屬性或方法：
```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);
    return arg;
}
// 錯誤，因為 arg 不一定有 length 屬性
```
所以我們可以給泛型約束，只允許傳入包含 length 屬性的變數
```ts
interface Lengthwise {
  length: number;
}
function logginIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}
```
此時如果呼叫 loggingIdentity 的時候，傳入的 arg 不包含 length，那麼在編譯階段就會報錯了：

## 泛型介面
泛型也可以用在介面中
```ts
interface CreateArrayFn {
  <T>(length: number, value: T): Array<T>;
}
let createArr: CreateArrayFn;
createArr = function<T>(length: number, value: T): Array<T> {
  let result: T[] = [];
  for (let i = 0; i < length; i++) {
    result[i] = value;
  }
  return result;
}

createArr(3, 'x'); // ['x', 'x', 'x']
```
進一步，我們可以把泛型引數提前到介面名上，更好閱讀：
```ts
interface CreateArrayFunc<T> {
    (length: number, value: T): Array<T>;
}
```

## 泛型類別
與泛型介面類似，泛型也可以用於類別的型別定義中
```ts
class GenericNumber<T> {
  zeorValue: <T>;
  add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

## 泛型引數的預設型別
```ts
function createArray<T = string>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}
```