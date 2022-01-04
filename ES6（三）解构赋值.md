# 解构赋值

***

> 这是ES6提供的一种更简洁的赋值模式。解构：从数组和对象中提取值。

在ES5中我们想要把一个数组中的值分别赋给几个变量：

```javascript
let a=10
let b=20
let arr=[1,2,3,4,5]
a=arr[0]
b=arr[1]
let arr1=arr.slice(2)   //取arr[2]到arr[arr.length-1]为一个新数组
console.log(a,b,arr1) //1  2   [3,4,5]
```

上面这种操作比较麻烦，用解构赋值可以很快实现这个操作：

```javascript
let [a,b,...arr1]=arr
```

***

## 数组解构

#### 1.先声明，再分别赋值

不能在声明的同时赋值，`let [a,b]`是不合法的。

```js
let a, b;         // 先声明变量
[a, b] = [1, 2];  // 然后分别赋值
console.log(a);   // 1
console.log(b);   // 2
```

#### 2.解构前设置默认值

b解构取出的值是 `undefined`，但是可以提前设置默认值：

```js
let a, b;
let arr=[1]
[a = 5, b = 7] = arr;   //a===1  b===7

let arr2=[1,null]
[a = 5, b = 7] = arr2;   //a===1  b===null
```

b的值是`undefined`，因此还显示默认值，但如果b是`null`，则仍显示null

#### 3.变量交换

之前的做法：提供一个缓存变量temp

```js
let a = 1,b = 3;
let temp=a;
a=b;
b=temp;
```

ES6里交换，非常方便

```javascript
[a, b] = [b, a];
[a,b,c,d]=[d,c,b,a];
```

#### 4.解构 函数返回的数组

调用函数，返回一个数组，可以用解构形式的变量来接收

```js
function c() {
  return [10, 20];
}

let a, b;
[a, b] = c();  //a===10     b===20
```

#### 5.跳过某一项返回值

你也可以跳过一些没有用的返回值。举例：

```js
function c() {
  return [1, 2, 3];
}
let [a, ,b] = c();  //a===1  b===3
```

#### 6.单取某一个值

```javascript
const array=[1,2,3,4,5]
let [,,,num,]=array  //num===4
```

#### 7.reset ：`...` 剩余值

`...变量`是剩余值的省略表示作为一个数组赋值给`变量`

```javascript
let [a,b,...c]=[1,2,3,4,5]
console.log(a,b,c) //1  2   [3,4,5]
console.log(c)     //[3,4,5]  c是一个数组
```

``...数组`会把数组这些值拆开成一个个的变量

```javascript
console.log(...c)  //3 4 5   
console.log(c[0],c[1],c[2])  //3 4 5
```

`...`必须放在最后一项，表示剩余项

```js
let [a, ...b] = [1,2,3]; //a===1 b===[2,3]
let [a,...b,c]= [1,2,3]; //不合法
```

#### 8.嵌套数组的解构

```js
const arr = ['hello', [1,2,3], 'goodbye'];
const [string,[one,two,three]] = arr;
console.log(string,one,two,three); // hello 1 2 3
```

***

## 对象解构

#### 1.let声明符 解构

```js
let x = { 
  y: 22,
  z: true 
}; 
let { y, z } = x; // y===22   z==='true'
```

#### 2.无声明符 解构

```js
({ y, z } = { y: 1, z: 2 });  //圆括号 (...) 不能省略
```

#### 3.解构对象时，赋以新的变量名

获取对象x的属性名y，然后赋值给一个名称为y1的变量。

```js
let x = { y: 22, z: true };
let { y: y1, z: z1 } = o;  
//y1===22   z1===true
```

#### 4.解构前设置默认值

如果解构取出的对象值是 `undefined` ，也可以使用设置的默认值

```js
let { a = 10, b = 5 } = { a: 3 };     //a===3   b===5
```

#### 3+4.赋值给新变量名的同时提供默认值

```js
let { a: aa = 10, b: bb = 5 } = { a: 3 };  //aa===3  bb===5
```

#### 5.同时使用数组和对象解构

在解构中数组和对象可以联合使用：

```js
const props = [
  { id: 1, name: 'Fizz' },
  { id: 2, name: 'Buzz' },
  { id: 3, name: 'FizzBuzz' },
];
const [, , { name }] = props;   //name==="FizzBuzz"
```

#### 6.以对象形式解构数组

对象的key就是数组的索引下标

```javascript
let arr=[1,2,3,4,5]
let {0:first,[arr.length-1]:last}=[1,2,3,4,5]  //first===1    last===5
```

***

## 函数参数的解构赋值

#### 1.函数传参

```javascript
//数组传参：参数是一组有次序的值
function f1([x, y, z]) { return x+y+z}
f1([1, 2, 3]);  //调用传参

//对象传参：参数是一组无次序的值  形参和实参名需对应
function f({x, y, z}) { return x }
f({z: 3, y: 2, x: 1});  //1
```

函数`f1`的参数表面上是一个数组，但在传入参数的那一刻，数组参数就被解构成变量`x`和`y z`。在函数内部没有数组，只有变量`x`和`y z`。

#### 2.从函数返回多个值

函数只能返回一个值，将它们放在数组或对象里就可以返回多个值，有了解构赋值，取出这些值就非常方便。

```javascript
// 返回一个数组
function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一个对象
function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```

***

## 总结

解构赋值，主要分为三种：

- 数组

```javascript
[x,y]=[1,2]
[x,y,...z]=[1,2,3,4,5]
[x,y]=[1]     //x===1  y===undefined
[x,y=10]=[1]  //x===1  y===10
```

- 对象：要么`let`，要么`()`

```javascript
let {a,b } = {a:1,b:2};
({a,b} = {a:1,b:2});

({a:A,b:B} = {a:1,b:2});   //A===1  B===2  重命名

let {a=10,b=20}={a:1}      //a===1  b===20  给默认值
let {a:A=10,b:B=20}={a:1}  //A===1  B===20
let {0:first,[arr.length-1]:last}=[1,2,3,4,5]  //first===1   last===5
```

- 函数参数

```javascript
function fn({oldname:newname="默认名"}={})   //有新的变量名和默认值
function fn([a,b,...c]){  }

function fn(){
  let a=10
  let b=20
  return {a,b}
}
let [a,b]=fn() //调用函数
```

***

## 面试题

1.从对象`obj`中取值

```javascript
const obj = {
    a:1,
    b:2,
    c:3,
    d:4,
    e:5,
}
```

```javascript
const {a,b,c,d,e} = obj;
const f = a + d;
const g = c + e;
```

想创建的变量名和对象的属性名不一致，起别名：

```javascript
const {a:a1} = obj;
console.log(a1);// 1
```



