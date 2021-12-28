# ES6（二）箭头函数

***

箭头函数作为ES6新出的语法，最重要的意义就是：（1）消灭了this的绑定（2）使代码更简洁。

整理过程中涉及到的内容有：

- ES3：具名/匿名（立即）执行函数
- ES6：箭头函数写法
- this的指向
- 强制绑定：call/apply/bind方法 区别
- ES6：箭头函数的意义

***

## ES3的函数

### （1）具名函数

```javascript
//具名函数
function xxx(p1,p2){
  console.log('hello')
}
```

对象内的函数的省略写法方式：

```javascript
const vm=new Vue({
  data:function(){  //可以省略:function   简写为data(){...}
  ...
  }
})
```

### （2）匿名函数

```javascript
let xxx=function (p1,p2){
  console.log('hello')
}
```

该声明将函数赋值给一个全局变量xxx，想省略这个变量名：**立即执行函数** （属于匿名函数）有两种写法： 

```javascript
let i=0
(function(j){ //j来接收的形参
  console.log(j)
}(i))    //i是传入的实参 

(function(j){ 
  console.log(j)
})(i)
```

***

## ES6的箭头函数

有以下几种声明方式：

```javascript
let xxx=(p1,p2)=>{
  console.log(p1+p2)
}

let xxx= p1 =>{   //如果传参只有一个值 可以不写( )
  console.log(p1)
}

let xxx= p1 => console.log(p1)  //如果函数体只有一句话，可以不写{ }

let xxx=(p1,p2)=> { return p1+p2 }
let xxx=(p1,p2)=>p1+p2  //return 和{ }必须同时删掉/加上

xxx(1,2)  //调用
```

箭头函数的最重要的意义就是：**消灭了this的绑定，让this不那么难用**。解释这句话之前我们需要先搞懂关于this的一切。

***

## arguments和this

除了**箭头函数**，所有函数都有，不写也有

```javascript
function fn(p1,p2,p3){
  console.log(arguments)  //[1, 2, 3]伪数组
  console.log(this)       //window
}
fn(1,2,3)             //调用fn : 传递arguments
fn.call(window,1,2,3) // 调用call方法 : 传递this
```

arguments是一个包含所有普通参数的**伪数组**

this是函数中的一个**隐藏参数**，指向了一个**对象**，是一个是未来出现的，来调用这个函数的对象，在这个函数里不想给它起名字而已。

```javascript
const obj={
  Name:'lucy',
  age:'23',
  sayHi:function(/* this */){  //隐藏的this
    console.log(`你好，我是${this.Name}，今年${this.age}岁了`)
  }
}
```

两种调用函数的方式：<font color="red">**小白调用法 / 大师调用法**</font>

```javascript
obj.sayHi()   //相当于obj.sayHi(obj)  隐式地将obj作为this传给sayHi  隐藏了太多细节   
obj.sayHi.call(obj) //推荐 显示传递  手动将this写清楚
```

## this到底指向的是什么？

仅仅依靠函数的声明，是没有办法判断this到底是什么的，我们需要时刻记住，在函数**调用**时，<font color="red">**this永远指向调用它的那个对象！this就是call/apply/bind（）里的第一个参数！**</font>后面的例子会不断强化这个概念。

### （1）定义在全局的函数：调用时，this===window

```javascript
var a=1
window.a===this.a  //true
```

>  用let声明的对象不能成为window对象的属性，因此sayHi()不等于window.sayHi()，如果是用var声明的可以

```javascript
let sayHi=obj.sayHi //赋值
sayHi()  //你好，我是undefined，今年undefined岁了
sayHi.call(window)
```

要时刻注意是函数内的this只在**调用时**才可以判断指向的是谁，sayHi()前面没有调用的对象，这里的**this就是window**，window里面没有Name，age属性，因此是undefined

### （2）构造函数中的this：谁被new了，this指向谁 

```javascript
//构造函数  this指向那个实例化的对象
function Person(name,age){
  this.name=name || 'tom'
  this.age=age ||18
  this.sayHi=function (){
     console.log(`你好,我是${this.name},今年${this.age}岁了`)
  }
}

const per1=new Person()  
per1.sayHi()     //  你好,我是tom,今年18岁了
per1.sayHi.call(per1)  

const per2=new Person('Jack',20)
per2.sayHi()  //  你好,我是Jack,今年20岁了
per2.sayHi.call(per2,'Jack',20) 
```

### （3）事件绑定函数中的this：谁调用函数，this指向谁

```html
<div class='btn'>
  点我
</div>
```

点击按钮，打印“点我”

```javascript
const b=document.querySelector('.btn')  //获取DOM对象
b.onclick=function (){ //事件触发时，绑定的方法中this指向当前的元素
  console.log(this.innerText) 
}
```

点击鼠标，打印undefined     **绑定this丢失**

```javascript
const b=document.querySelector('.btn')  //获取DOM对象
b.onclick=function (){
  setTimeout(function (){  //setTimeout调用的函数  全称是window.setTimeout
    console.log(this.innerText) //this是window
  },0)
}
```

***

## 强制绑定

这时，有一个obj2对象，我们不想对它重新定义sayHi方法，可以通过**call/apply/bind**三种方法，调用obj对象的函数时，这时函数内部this指向为obj2，这就是**强制绑定**。

### call

call第一个参数是：this的指向的对象
call第二个参数 ~ 第N个参数：对应函数的第一个形参 ~ 第N-1个形参

```javascript
obj2={
  Name:'Lily',
  age:'20'
}
obj.sayHi.call(obj2) //你好，我是Lily，今年20岁了
```

调用一个普通函数，函数里没有this，call第一个参数用undefined或者null占位

```javascript
function add(x,y){
  return x+y
}
add.call(undefined,1,2)  //3
```

***

### apply

apply只传入两个参数，第一个参数是this的指向的对象，第二个参数必须为**数组/伪数组**的形式
apply + arguments ：有时不知道/不想统计传入参数的个数：

```javascript
//返回传入参数里的最大值  
function fn(arr){
  return Math.max.apply(Math,arr) 
}
arr=[1,2,3,4,5,6,7,8,9]
fn(arr) //9
```

定义一个 log 方法，让它可以实现console.log 方法，由于传入参数的个数不确定，所以需要用到apply方法：

```javascript
function log(){
  console.log.apply(console, arguments);
};
log(1);    //1
log(1,2);    //1 2
```

用于方便地实现数组合并：

```javascript
const arr1=[1,2,3]
const arr2=[4,5,6]
arr1.push(arr2) //[1,2,3,[4,5,6]] 
arr1.push.apply(arr1,arr2) //[1,2,3,4,5,6]
```

***

### bind

bind( )方法会返回一个新的**绑定函数**，不会立即执行。只有**调用**这个绑定函数时，this指向bind()方法的第一个参数，传入 bind() 方法的第二个~第N个参数为原函数的第一个形参 ~ 第N-1个形参。

```javascript
const obj={
  Name:'lucy',
  age:'23',
  sayHi:function (){
    console.log(`你好，我是${this.Name}，今年${this.age}岁了`)
  }
}
obj.sayHi()           //你好，我是lucy,今年23岁了
obj.sayHi.call(obj)   //你好，我是lucy,今年23岁了
obj.sayHi.bind(obj)() //你好，我是lucy,今年23岁了
```

### 总结

- this的**强制绑定**：call，apply，bind都可以改变this的指向，调用它们仨的一定是函数。
- 传参不同：call，bind传参一致，apply只允许传两个参数，第二个一定是数组/伪数组
- 执行时机：call，apply立即执行，bind需要再调用执行

三者的调用格式有些区别：

```javascript
方法.call(this指向对象，参数1，参数2，...，参数N)
方法.apply(this指向对象，[参数1，参数2，...，参数N]) 
方法.bind(this指向对象，参数1，参数2，...，参数N)()  
```

用一个故事来记忆：

> 猫吃鱼，狗吃肉，有天狗想吃鱼了
>
> 猫.吃鱼.call(狗，鱼1，鱼2，鱼3)
> 
> 猫.吃鱼.call(狗，[鱼1，鱼2，鱼3])
> 
> 猫.吃鱼.bind(狗，鱼1，鱼2，鱼3)()

***

## 再回到箭头函数的意义

ES5中，这个this可能指向的是window / 实例对象 / 普通对象，**调用时**再进行判断，比较难用。使用小白调用法往往会隐藏很多细节，为了加强记忆，我们可以用call这种写法，指定传的第一个参数填好这个this的真实指向。

ES6中规定了箭头函数就没有this的绑定，仅依靠函数**声明**就可以轻松判断this的指向。

（1）没有this的绑定，因为this真的太难用了

```javascript
this  //控制台最外面的this就是window变量
let fn=()=>console.log(this) 
fn() //window
fn.call({name:'frank'}) //window
```

非要在箭头函数里加`this`，那这个this的值就是**上一层 非箭头函数里的this**，箭头函数的this是由**外层作用域**决定的，作用域只有全局域，函数域，没有代码块域。如果箭头函数是在全局定义的，那就是最外面的window变量，call也改变不了

```javascript
var obj = {
  name: 'obj',
  foo1: () => { 
    console.log(this.name)  //this是window  
  },
  foo2: function () {
    console.log(this.name)  
    return () => {  
      console.log(this.name) //this是上一级作用域的this
    }
  }
}
var name = 'window'
obj.foo1() //window
obj.foo2()()//obj  obj
```

没有了this，怎么用箭头函数实现代替类似的功能，找到当前调用这个函数的对象：在this隐藏的位置传一个self

```javascript
let obj={
  name:'obj'
  hi:(self)=>{  
    console.log(self.name)
  }
}

obj.hi(obj)  //箭头函数不能隐藏任何细节，传的什么都要写清楚
```

（2）让代码更简洁，快速赋值

```javascript
let array=[1,2,3,4,5]
array.map(n=>n*n).map(n=>n+1)
/*
for(let i=0;i<array.length;i++){
  array[i]=array[i]*array[i]
  array[i]=array[i]+1
}
*/
```

***

## 面试题

### 1. 全局调用函数：this指向了window

```javascript
var a = 10;
function foo () {
  console.log(this.a)
}
foo();  //10  this是window
```

```javascript
let a = 10 
const b = 20

function foo () {
  console.log(this.a)  
  console.log(this.b)
}
foo();  //undefined   因为window对象没有a变量 
console.log(window.a)//undefined
```

```javascript
var a = 1  //全局变量
function foo () {
  var a = 2 //局部变量
  console.log(this)   //window
  console.log(this.a) //1 window.a
  console.log(a)      //2  就近原则
}
foo() //this是window
```

调用全局定义的函数，函数内嵌套的函数也被调用时，还是window来调用的

```javascript
var a = 1    
function foo () {
  var a = 2  
  function inner () {  //inner还是window调用的
    console.log(this.a)
  }
  inner()  
}

foo()   //1
```

### 2.this指向了<font color="red">最后</font>调用它的那个对象

```javascript
var a = 2
var obj = { 
  a: 1, 
  foo  //var obj = { foo }就相当于是var obj = { foo: foo }
}
function foo () {
  console.log(this.a)
}
obj.foo()  //this是obj对象
```

`obj.foo()===window.obj.foo()`只看最后谁调用了它，obj比window更靠后一点

### 3.隐式绑定this指向丢失

把函数赋值给一个**全局变量**，或者当成**参数**传给另一个函数调用，函数内this的指向会改变：

```javascript
var a = 2;
var obj = { a: 1, foo };
var foo2 = obj.foo;  //另一个变量来给函数取别名会发生隐式丢失
var obj2 = { a: 3, foo2: obj.foo }
function foo () {
  console.log(this.a)
};
obj.foo();  //1   this是obj
foo2();     //2   this是window
obj2.foo2() //3   this是obj2    window.obj2.foo2()

function doFoo (fn) {
  console.log(this)
  fn()
}
doFoo(obj.foo) //window  2
obj2.doFoo(obj.foo)  //this是obj2 { a:3, doFoo: f }   2
```

### 4.强制绑定call apply bind

```javascript
function foo () {
  console.log(this.a)
}
var obj = { a: 1 }
var a = 2

foo()  //2
foo.call(obj)   //1
foo.apply(obj)  //1
foo.bind(obj)   //没调用，不会执行
```

```javascript
var obj1 = {
  a: 1
}
var obj2 = {
  a: 2,
  foo1: function () {
    console.log(this.a) 
  },
  foo2: function () {
    setTimeout(function () { //函数作为setTimeout的参数，obj2丢失了this的绑定，this是window
      console.log(this)  
      console.log(this.a)
    }, 0)
  },
  foo3: function () {
    setTimeout(function () {
      console.log(this)
      console.log(this.a)
    }.call(obj1), 0) //call来强制绑定到obj1
  }
}
var a = 3

obj2.foo1()  //2   this是obj2  
obj2.foo2()  //window  3   this是window
obj2.foo3()  //obj1    1   this是obj1 
```

区别：函数.call() 与  函数().call()

```javascript
function foo () {
  console.log(this.a)
}
var obj = { a: 1 }
var a = 2

foo()         //2  this是window
foo.call(obj) //1  this是obj
foo().call(obj) //2   Uncaught TypeError: Cannot read property 'call' of undefined  
//foo()返回值是undefined 对undefined没有call方法 
```

```javascript
function foo () {
  console.log(this.a)
  return function () {  
    console.log(this.a)
  }
}
var obj = { a: 1 }
var a = 2

foo()  //2     返回匿名函数 但是没有调用
foo.call(obj)   //1  返回匿名函数，call立即执行
foo().call(obj) //2  1  先执行foo()  外层是this是window  再call立即执行匿名函数  this是obj
```

### 5.箭头函数与this

```javascript
var name = 'window'
var obj1 = {
  name: 'obj1',
  foo1: function () {
    console.log(this.name)
    return () => {
      console.log(this.name)  //this就是上一层this
    }
  },
  foo2: () => {
    console.log(this.name)  //window
    return function () {
      console.log(this.name)
    }
  }
}
var obj2 = {
  name: 'obj2'
}
obj1.foo1.call(obj2)()  //obj2  obj2
obj1.foo1().call(obj2)  //obj1  obj1
obj1.foo2.call(obj2)()  //window  window ???重点
//obj1.foo2.call(obj2) 无效  等同于obj1.foo2()   返回一个函数，再调用这个函数的是window
obj1.foo2().call(obj2)  //window  obj2

```

### 6.综合题

```javascript
let obj={
  foo:'bar',
  fn:function(){
    let self=this
    console.log(this.foo)  //this是obj，属性foo的值为'bar' 
    console.log(self.foo)  //同上
    (function(){           //立即执行函数：末尾括号没有传入参数 this就是window
      console.log(this.foo)  //undefined
      console.log(self.foo)  //self是fn域内的局部变量，还是obj
    }())
  }
}
obj.fn( )  // bar bar  undefined bar    === obj.fn(obj)  
```

***

明确以上概念后，希望今后不再有this的困扰~
