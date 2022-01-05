# ES6（四）symbol 数据类型

***

> symbol是ES6 引入了一种新的数据类型，表示独一无二的值。其余几种分别为：字符串（String）、数值（Number）、布尔值（Boolean）、空值（undefined）、空值（null）、大整数（bigInt）、对象（Object）。

本篇整理Symbol，顺便整理以下数据类型的重点问题：

- 为什么 `ES6` 要提出 `Symbol`？

- `Null` 和 `Undefined` 有什么区别？
- 五种falsy值分别是？

- `0.1 + 0.2 !== 0.3` 你如何解决这个问题？
- `BigInt` 解决了什么问题？

***

## Symbol

ES6 引入了一种新的原始数据类型 `Symbol`，表示独一无二的值

```js
let name = Symbol();
typeof name  // "symbol"
```

### 作用：

1. 表示一个独一无二的变量，任何两个`symbol`都是不相等的

```javascript
let a=Symbol('lucy')
let b=Symbol('lucy')
a===b  //false
```

2. symbol变量可以作为对象属性名，用来避免**对象属性名冲突**

   **只有symbol和string类型的值才可以作为对象的属性名**

```js
let name = Symbol();
let a = {  age:23  };

a[name] = 'lucy';
--------------------------
a = {
  [name]: 'lucy'
};
--------------------------
Object.defineProperty(a, name, { value: 'xu' });

// 以上写法都可以获得属性名为symbol/string的属性值
console.log(a[name])  //xu
```

### 在对象中查找Symbol类型的属性名

`symbol`的变量不会出现在 `Object.keys()`的结果中，但是`Object.getOwnPropertySymbols()`可以

```javascript
Object.keys(a);  //["age"]  
Object.getOwnPropertySymbols(a)  //name
```

### Symbol.for(key)

使用现有的key来搜索现有的symbol变量，找到则返回这个变量，如果不存在则新建一个值。

```
const s1 = Symbol.for('foo');
const s2 = Symbol.for('foo');
console.log(s1 === s2); // true
```

### Symbol.for( )与Symbol( )区别

这两种写法，都会生成新的 Symbol。

Symbol.for()生成的变量是全局的，不会每次调用就返回一个新的 Symbol 类型的值，而是会先检查给定的`key`是否已经存在，如果不存在才会新建一个值。

比如，如果你调用`Symbol.for("cat")`30 次，每次都会返回同一个 Symbol 值，但是调用`Symbol("cat")`30 次，会返回 30 个不同的 Symbol 值。

### Symbol.keyFor( )

返回一个已登记的**全局** Symbol 类型值的`key`。由Symbol.for()创建

```javascript
let s1 = Symbol.for("foo");
Symbol.keyFor(s1) // "foo"

let s2 = Symbol("foo");
Symbol.keyFor(s2) // undefined
```

上面代码中，变量`s2`属于未登记的 Symbol 值，所以返回`undefined`。

### 为什么 ES6 要提出 Symbol？

 ES5 的对象属性名都是字符串，这容易造成**属性名的冲突**。比如，你使用了一个他人提供的对象，但又想为这个对象添加新的方法，新方法的名字就有可能与现有方法产生冲突，如果能保证每个属性的名字都是独一无二的就好了，这样就从根本上防止属性名的冲突。

***

## 数据类型

由于JS在声明变量时不需要提前规定变量的类型，在程序运行的过程中，类型会被自动确定

```javascript
var a = 42;  // Number
a = "bar";   // String
a = true;    // Boolean
```

### 1.null与undefined

- null表示一个空对象指针，`Object.prototype===null`

- `undefined` 表示未定义的变量，此处应该有一个值，但是还没有定义，常见于以下几种：

1. 变量声明未赋值时；
2. 调用函数时，应该提供的参数没有提供时；
3. 对象没有赋值的属性；
4. 函数默认的返回值 ；

```javascript
var i;    // undefined

function f(x){console.log(x)}
f() // undefined

var  o = new Object();
o.p // undefined

var x = f();
x // undefined
```

- `undefined`转为数值时为NaN，`null` 转为数值时为0。

```javascript
Number(null)   // 0
5 + null      // 5
Number(undefined)  // NaN
5 + undefined   // NaN
```

- 特殊：`typeof null === "object"`

### 2.判断布尔值：五种falsy值

五种空值和假值，分别为 undefined，null，false，""，0，NAN

### 3.Number：0.1+0.2 !== 0.3

```javascript
0.1 + 0.2 
0.30000000000000004
```

原因：`JavaScript` 是以 `64` 位双精度浮点数存储所有 `Number` 类型值，浮点数的存储过程的进制转换，精度会丢失，

解决：将数字转成整数

```javascript
function add(num1, num2) {  //num1=0.58  num2=0.1
 const a = (num1.toString().split('.')[1] || '').length;  //2
 const b = (num2.toString().split('.')[1] || '').length;  //1
 const c = Math.pow(10, Math.max(a, b));  //10^2=100
 return (num1 * c + num2 * c) / c;  //(0.58*100+0.1*100)/100
}
```

```javascript
typeof NaN === 'number'
```

### 4.BigInt 

number类型能表示的值最大为2^53

```javascript
Math.pow(2,53)          //9007199254740992
Math.pow(2,53)+1        //9007199254740992
Number.MAX_SAFE_INTEGER //9007199254740991
```

`BigInt` 可以表示任意大的整数，把123 写成 123n，或者BigInt(123) 都可以，在与number类型值运算时，要先将number类型值转换成BigInt类型。

```javascript
BigInt(Math.pow(2,53)+2)   //9007199254740994n
BigInt(Math.pow(2,53))+2n  //9007199254740994n
```



