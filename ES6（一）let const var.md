# ES6 (一) var let const 

>  ECMAScript 6（ES6）是于2015年6月正式发布的javascript语言的标准规范，下面介绍ES6最基础的新增——声明命令let const。

最近查阅了很多资料，终于**彻底搞懂**了var let const，还顺便捋顺了以下几个重要的概念：

- 作用域：全局域 函数域
- 函数的执行时机
- 全局变量/局部变量的隐式声明
- 变量提升
- 立即执行函数
- 块级作用域
- 暂时死区TDZ

## 声明命令

```javascript
/* ES3提出 */
a=1      
var a=1  
/* ES6提出 */
let a=1  
const a=1 
```

下面逐个介绍这四种声明命令，但为防止混乱，需要先回顾一遍以下几点概念。

## 作用域

> 变量的调用范围：单向可访问的，**在函数局部域**可以访问并修改**全局域**内的变量   

```javascript
//这是全局域
function fn(){
  //这是局部域
  var x=10  //x是在fn域内的局部变量
}
fn()
console.log(x)  //报错  
```

### 函数的执行时机

没有调用，就没有执行

```javascript
var x=10
function fn(){
  x=20
}
console.log(x)  //10
```

调用了，才执行，才真正修改了这个变量的值：

第三行，`x=20`没有带任何声明符号，于是向**上一级作用域**去寻找它的声明符号，由局部=>全局，此时的x就是全局变量

```javascript
var x=10
function fn(){
  x=20 
}
fn()  //调用 执行
console.log(x)  //20
```

第三行，如果在函数内重新**声明**了x，那么这个x就是函数内的局部变量

```javascript
var x=10     //x是全局域的变量
function fn(){
  var x=20   //重新声明，x是fn域私有的变量
  console.log(x) //20  就近访问原则
}
fn() //20
console.log(x)  //10  x仍是全局变量
```

总结：在函数内使用x变量时，会在当前函数域寻找声明，找到了就使用这个**局部变量**x，没有声明就说明它不是个局部变量，向**上一级域**寻找x声明，一直找到全局域，发现也没有声明，那么JS引擎会在**全局**加上**隐式声明**，默认值是undefined。

如何**验证**一个变量是否是全局变量？打印window.变量名

```javascript
//var x
function fn(){
  x=10  //一个变量不声明直接赋值，会在全局进行隐式声明
}
fn()
console.log(x)   //10
console.log(window.x)  //10
```

## 总结

- 作用域是单向可访问的：内部作用域（函数局部域）可以访问外部作用域（全局域）的变量，外部不可以访问内部的**私有变量**
- 私有变量：在当前局部作用域下**声明**的变量，如果没有声明符号不算，在调用时，会向上一级寻找声明，一直找到全局作用域为止，如果一直找不到，JS引擎会在全局**隐式声明**。此外，形参相当于**函数域**内的局部变量：

```javascript
var w=10
function fn(w){  //这是一个形参 函数的形参会作为函数的局部变量的隐式声明  
  //相当于var w
  w=20
  console.log(w)
}
fn(w)  //20
console.log(w)  //10  这是一个变量
```

- 就近原则：变量的使用会优先访问自己作用域的局部变量，找不到才向上寻找。

明确了以上概念，再回到这四种声明命令来：

***

## a=1

可以分一下四种情况讨论：

```javascript
var a=10
function fn(){
  var a=1  //有声明符，fn域内的局部变量
  console.log(a)
}
fn()  //1
console.log(a)  //10
-----------------------------
var a=10
function fn(){
  a=1  //没有声明符，向上一级寻找，a是全局变量
  console.log(a)
}
fn()  //1
console.log(a)  //1
console.log(window.a)  //1
-----------------------------
var a=10
function fn(){
  var a=5
  fn2()
  function fn2(){
    a=1  //没有声明符号，向上一级寻找，a是fn域内的局部变量
  }
  console.log(a)
}
fn()   //1
console.log(a)  //10
-----------------------------
a=1  //全局变量
console.log(a)  //1
console.log(window.a)  //1
```

因此，判断一个`a=1`是**局部变量**还是**全局变量**就这么麻烦，需要看它所处的函数作用域内是否有var声明，没有再向上一层作用域去寻找，直到找到全局域都没有声明，就**隐式声明**为全局的，写这样的代码很容易造成歧义，不推荐使用。

***

## var a=1

（1）var 存在变量提升，在声明之前使用的值为`undefined`。

```javascript
console.log(x)  //undefined
var x=10
console.log(X)  //10
```

（2）在**同一个**作用域内，var可以**重复声明**同一个变量

（3）var的作用域：**全局域、函数域**。在ES6出来之前，只有**全局域**和函数的{ }内构成**函数域**这两种作用域，函数内部可以嵌套的其他函数构成了一层层的函数作用域，最外面的是全局域，由if for 里面{  }包起来的并不是局部域，仍然是全局域。它们关不住var声明的变量。

```javascript
for(let i=0;i<5;i++){
  var x=10  //x是全局变量，for关不住它
}
console.log(x)  //10 
```

因此，建议把var都写在这个作用域的最开头。

不过这样也不可避免由于变量提升带来一个问题：var会**暴露局部变量**，无论你在代码块里写什么，var a的声明都会提升到这个域的最开头，在全局就会挂载到window对象上，变成了window.a，我们并不希望这样的事情发生。

```javascript
var arr=[]
for(var i=0;i<5;i++){   //for循环不构成作用域，i是一个全局变量
  arr.push(function (){
    console.log(i)
  })
}
arr[1]()    //在调用的时候才执行打印i，i值早已是循环完的5
```

一个解决办法是使用**IIFE 匿名函数中的立即执行函数**，希望在每次循环时，把当前i的值截留下来：立即执行函数立刻将i的值传给了j：立即执行函数执行完毕了，才能进入下一次循环。

```javascript
var arr=[]
for(var i=0;i<5;i++){
  (function(j){  //接收的形参  局部变量  相当于var j 
    arr.push(function (){
      console.log(j) 
    })
  })(i)         //实参
}
arr[1]()  //1
arr[2]()  //2
```

这是2015年ES6发布之前，JS语言设计上的失误，大部分语言都存在**块级作用域**，很遗憾的是，var声明的变量并没有这种作用域，只有全局域和函数域，在当时只能用立即执行函数来弥补这个失误。

***

## let a=1

（1）let 没有变量提升

```javascript
console.log(x)  //报错  
let x=10;
console.log(X)  //10
```

（2）在**同一个**作用域内，let不可以被重复声明。

```javascript
{
  let a=1
  console.log(a)  //1
  a=2   //let一个变量只能一次  
  console.log(a)  //2
}
```

（3）let的作用域：**块级作用域**，在最近的代码块{ }之间  （函数块，for( ){ }，if{ }只要是{ }，JS就认为是块级作用域  ）

```javascript
{
  let a=1
}
console.log(a) //报错  a未定义
```

（4）在**不同**作用域内，let可以重复声明一个变量，但是如果在一段作用域内先调用再声明，会有**暂时死区TDZ**问题。因此在作用域内，不要先调用再声明！

```javascript
let num=10       //全局变量
function fn(){
  console.log(num)  //暂时性死区  声明前调用了
  let num=20     //局部变量 因为let没有变量提升
}
```

### 魔法：for + let

 一共有六个i，圆括号里的 i 和花括号里的i0 i1 i2 i3 i4，每一个 i 都是一个独立的变量

```javascript
var arr=[]
for(let i=0;i<5;i++){   // i的作用域只在( )里  
   //块里面的i = 圆括号里面i 的值  js自动加了这句 
  arr.push(function (){
    console.log(i)
  })
    //圆括号里面的i = 块里面的i的值  js自动加了这句 
}
arr[1]()  //1
arr[2]()  //2
```

这样刚才的例子就迎刃而解！只需要把var 改成let，不再需要麻烦的立即执行函数~

***

## const 

（1）（2）（3）同let

（4）const只有一次赋值机会，而且必须在声明时马上赋值

```javascript
{
  const a=1       //const a 报错  //不赋值也得赋值
  console.log(a)  //1
  //a=2           //报错  
}
```

***

## 对比

|          | var                                                          | let        | const      |
| -------- | ------------------------------------------------------------ | ---------- | ---------- |
| 作用域   | （1）在函数外：全局域，挂载到window对象上<br/>（2）在函数内：整个函数域 | 块级作用域 | 块级作用域 |
| 变量提升 | 有，提升到**该作用域**的最开头                               | 无         | 无         |
| 重复声明 | 可以重复声明                                                 | 不可以     | 不可以     |

***

## 面试题

### 题目1.

```javascript
for(var i=0;i<6;i++){
  
}
console.log(i)  //6
```

```javascript
for(var i=0;i<6;i++){
  function fn(){
    console.log(i)
  }
  fn()  //打印出0,1,2,3,4,5
}
```

执行的时机：代码一瞬间执行完了，在点击这个按钮之前，i早就是6了

```html
<button id=x>按钮</button>
```

```javascript
for(var i=0;i<6;i++){
  function fn(){
    console.log(i)
  }
  x.onclick=fn  //6
}
```

### 题目2.

```html
<ul>
  <li>1</li>
  <li>2</li>
  <li>3</li>
  <li>4</li>
</ul>
```

```javascript
var liTags=document.querySelectorAll('li')
for(var i=0;i<liTags.length;i++){
  liTags[i].onclick=function(){  //for循环里形成了闭包
    console.log(i)
  }
}
//鼠标还没动呢 i就已经是4了   var作用域是全局的
```

解决办法：（1）用let：j作用域只在函数块里

for循环每次执行都是一个全新的独立的块作用域

```javascript
var liTags=document.querySelectorAll('li')  //每一个都是<li>2</li>元素
for(var i=0;i<liTags.length;i++){
  let j=i
  liTags[j].onclick=function(){
    console.log(j)
  }
}
相当于重新开辟了四块空间 j0 j1 j2 j3
{
	let j=0
	liTags[0].onclick=function(){
  	console.log(0)
	}
}

{
	let j=1
	liTags[1].onclick=function(){
 	  console.log(1)
	}
}

{
	let j=2
	liTags[2].onclick=function(){
 	  console.log(2)
	}
}

{
	let j=3
	liTags[3].onclick=function(){
 	  console.log(3)
	}
}
最后i是走到了4，但是每个j还保留这个自己的值
```

（2）用立即执行函数

```javascript
var liTags=document.querySelectorAll('li')  
for(var i=0;i<liTags.length;i++){
 (function(j){  //用j来接收
   liTags[j].onclick=function(){
     console.log(j)
  }
 }) (i) //传递一个i
}
```

（3）最简单的一种写法：魔法 一共七个i  圆括号里的i  和{}里的i  i0 i1 i2 i3 i4 i5 

```javascript
var liTags=document.querySelectorAll('li')  
for(let i=0;i<liTags.length;i++){  //i的作用域只在( )里
  //块里面的i = 圆括号里面i 的值
  liTags[i].onclick=function(){   //js自动加了一句let _i=i  创建了一个同名的_i
    console.log(i)
  }
  //圆括号里面的i = 块里面的i 的值
}
```

### 题目3.

```javascript
for (var i = 0; i <10; i++) {  
  setTimeout(function() {  
    console.log(i);        
  }, 0);
}

// 10个10
```

```javascript
for (let i = 0; i < 10; i++) { 
  setTimeout(function() {
    console.log(i);    // i 是循环体内局部作用域，不受外界影响。
  }, 0);
}

//0  1  2  3  4  5  6  7  8 9
```





