# ES6（五）Promise

***

本篇内容还涉及到以下知识点：

- 同步与异步
- 回调的作用  异步与回调关系     
- 回调地狱
- Promise基本语法  作用   链式编程
- Promise.then( )   Promise.all( )  Promise.race( ) 

***

## 同步

由上到下顺序执行，可以立刻拿到结果

```javascript
console.log('a')
console.log('b')
...
```

举例：挂号，食堂打饭

## 异步

**非阻塞**：不能直接拿到结果，主动让出这个位置，让JS先去执行后面的代码，等到**某个时候**再执行这行代码

把setTimeout放到调用栈里，到时间2秒后再调用这个setTimeout里的函数

```javascript
console.log('a')
console.log('b')
setTimeout(()=>{ console.log('轮到我的号码了！')}，2000)   //异步
console.log('c')
console.log('d')
// a b c d 轮到我的号码了！
```

举例：餐厅排队取号，号到我了再来吃饭

拿到结果的方法（如何知道该轮到这个异步任务执行了）：

- 轮询：每隔多久自己去问一下
- 回调：餐厅打电话通知你。具体做法：我们先写好这个函数，不调用它，等到异步任务完成时，浏览器会调用该函数，并向函数中传参。

### 哪些任务是异步的

- setTimeout：延时打印
- ajax请求：网络请求
- AddEventListener：事件绑定

### 异步与回调的关系

区别：异步任务不仅仅可以用到**回调**，还可以**轮询**；回调函数也不仅仅用在**异步**，还可以用在**同步**。

函数返回值在**setTimeout、AJAX、AddEventListener**内部，就是异步函数

***

## 回调 *callback*

写给别人调用的函数：我写给你，你**回**头**调**一下

### 不传参的回调

```javascript
function f1(){
  console.log('hi')
}
function f2(fn){
  fn()
}
f2(f1) 
```

**f1是回调**：f1是我写好的，自己没有调用，传给f2（让浏览器）调用的函数，

### 传参的回调：

```javascript
function f1(x){
  console.log(x)
}
function f2(fn){
  fn('hi')
}
f2(f1) //hi
```

### 回调的应用

这是一个摇骰子函数fn：

```javascript
function fn(){  
  setTimeout(()=>{
    return parseInt(Math.random()*6)+1
  },1000)
  //return undefined
}
```

- fn( )没有写返回值，默认return undefined，不是异步函数

- 箭头函数有返回值，return 0~5之间的随机数，才是异步函数

```javascript
const n=fn()
console.log(n) //undefined
```

如何拿到异步结果？回调

```javascript
function f1(x){  //我声明的函数
  console.log(x)
}
function fn(f){
  setTimeout(()=>{
    f(parseInt(Math.random()*6)+1)
  },1000)
}
fn(f1) //让fn调用f1，f1是回调
```

可以将f1变成箭头函数，简化为

```javascript
fn(x=>{console.log(x)})
fn(console.log)
```

### 回调的作用

由于异步任务不能直接拿到结果，我们为浏览器写一个回调函数（留电话号码），在异步任务完成时**调用**回调函数（打电话），把**结果**作为参数传给该函数（电话里告诉可以来吃了）

### 问题：如果异步任务有两种结果：成功/失败 

解决：让回调函数接受两种参数

```javascript
var fs = require('fs');
fs.readFile('./1.txt',  (err, data) => {
    if(err){
      console.log('失败')
      return 
    }else{
      console.log(data.toString()); 
    }
})
```

但是这样的写法**不规范**，有人用success+error，有人用success+fail

### 回调地狱

三个文件的输出顺序是不确定的，后执行的可能会先输出。若要保证输出顺序，在前一个异步操作的回调函数中调用后一个异步操作。

```javascript
var fs = require('fs');
fs.readFile('./1.txt',  (err, data) => {
    if (err) {
        throw err
    }
    fs.readFile('./2.txt', (err, data) => {
        if (err) {
            throw err
        }
        fs.readFile('./3.txt', (err, data) => {
            if (err) {
                throw err
            }
            console.log(data.toString());
        })   
        console.log(data.toString());
    })
    console.log(data.toString());
})
```

当异步操作越多，这种嵌套的层级也就越复杂，像波形拳一样，使代码变得看不懂，不利于代码维护，这就是回调地狱。

***

## Promise

### Promise是什么

在异步任务中，往往需要用到的回调中，promise是为了避免回调地狱的一种异步的解决方案。

### Promise的作用

- 规范回调的名字以及顺序

- 拒接回调地狱，使代码的可读性更强。
- 方便地捕获错误

如何解决：链式编程

### promise对象的三个状态：

- pending 初始状态[待定]
- fulfilled  操作成功[实现]
- rejected 操作失败[拒绝]

### 基本语法

传入构造函数的参数是一个**函数**

```javascript
let p=new Promise(传一个函数)  //初始状态是pending
```

这个函数有**两个参数**，resolve和reject：

```javascript
let p=new Promise(function(resolve,reject){ ... })
let p=new Promise((resolve,reject)=>{ ... })  //也可写成箭头函数
```

返回值p是一个Promise对象，有两个属性，state状态，result结果

```javascript
let p=new Promise(function (resolve,reject){ //初始状态是pending
  if(/*  异步操作成功  */)
    resolve('success');    //状态变成fulfilled
  else
    reject('fail');     //状态变成rejected
})
```

### Promise.then()

<font color="red">一旦**状态改变**</font>，（pending—>fulfilled或者pending—>rejected），就会触发`Promise.then(()=>{},()=>{})`，传入两个参数，都是函数第一个箭头函数是resolve回调的函数，第二个是reject回调的函数，返回的也是一个Promise对象。

```javascript
let p=new Promise(function (resolve,reject){   //初始状态是pending
    resolve('success');    //实参 
})
let p1=p.then(
  (res)=>{console.log(res)},  //形参，使用p对象里resolve调用的实参
  (err)=>{console.log(err)})  //输出success
//p: Promise {<fulfilled>: "success"}
//p1:Promise {<fulfilled>: undefined}
```

### 链式编程

由于promise.then()返回的也是一个promise实例对象，在其后可以继续链接任意个then，前一个then( )的**回调结果**将作为参数传递给下一个then回调

```javascript
let p=new Promise(function (resolve,reject){   
    resolve('success');    //实参 
})
let p_=new Promise(function (resolve,reject){   
    resolve('success again!');    //实参 
})

let p1=p.then((res)=>{   //res里接受的是p的resolve里传的值
  console.log(res)      
  return p_
})  //success
let p2=p1.then((res)=>{  //res接受的是p_的resolve里传的值
  console.log(res)
})  //success again!
```

链式编程解决上面三个文件的异步顺序问题：

```javascript
let p1 = new Promise((resolve, reject) => {
  fs.readFile('./1.txt', (err, data) => {  err?reject(err):resolve(data) })
})
let p2 = new Promise((resolve, reject) => {
  fs.readFile('./2.txt', (err, data) => {  err?reject(err):resolve(data) })
})

p1.then((data) => {   //data接收到的是resolve包裹的值
    console.log(data.toString());
    return p2;        // return 返回的结果会在后面紧接着的then方法中被函数接收
   }, (err) => {
    console.log('出错了', err)
}).then((data) => {   // then里的data就是p2
    console.log(data);
} )
```

这样的代码冗余太多，每次读取一个文件，都要创建一个新的promise实例，将这个过程**封装**一下：

```javascript
function pReadFile(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, (err, data) => {  err?reject(err):resolve(data)  })
  })
}

let p1 = pReadFile('./1.txt');
let p2 = pReadFile('./2.txt');

p1.then((data) => {
    console.log(data.toString());
    return p2;
}, (err) => {
    console.log('出错了', err)
}).then((data) => {
    console.log(data.toString());
} )
```



### Promise.all([p1,p2])

用来处理多个异步任务，传入一个数组，将**多个**Promise实例包装成**一个新的**Promise实例返回，成功和失败的返回值不同。

- 所有请求都成功时，才返回一个结果**数组**，等所有都成功后才执行，以最慢的任务为准。
- 有一个失败了就算失败，返回**最先被reject失败状态的值**。

```javascript
let p1=new Promise((resolve,reject)=>{
  resolve('成功了')
})
let p2=new Promise((resolve,reject)=>{
  resolve('我也成功了')
})
let p3=new Promise((resolve,reject)=>{
  reject('失败')
})
let p3=Promise.reject('失败')

let p4=Promise.all([p1,p2]).then((result)=>{
  console.log(result)    //['成功了','我也成功了']
}).catch((error)=>{
  console.log(error)
})

Promise.all([p1,p2,p3]).then((results)=>{
  console.log(results)    
}).catch((error)=>{
  console.log(error)      //'失败'
})
```

### Promise.race([p1,p2])  

赛跑，谁先完成就执行谁，以最快的任务为准，Promise.race([p1,p2,p3])里面哪个结果获得更快，就返回那个结果，不管结果本身是成功还是失败状态

```javascript
let p1=new Promise((resolve,reject)=>{
  setTimeout(()=>{
    resolve('成功，我是1号')
  },1000)
})

let p2=new Promise((resolve,reject)=>{
  setTimeout(()=>{
    resolve('成功，我是2号')
  },500)
})

Promise.race([p1,p2]).then((result)=>{
  console.log(result)    
}).catch((error)=>{
  console.log(error)     
})
```

### 总结

- Promise就是对异步操作进行封装，

- 思想就是，**then()或catch()中写回调函数**，然后通过resolve()或reject()来调用。
- 因为Promise.then()返回的也是一个Promise实例，通过**链式的调用**解决回调地狱问题
- 一般有异步操作我们都要使用promise对异步操作进行封装，养成良好的习惯。
  



