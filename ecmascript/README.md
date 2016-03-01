
# 一. let 和 const

## 1 let
### 基本用法
ES6新增`let`命令,与`var`用法类似,但所声明的变量只在声明的代码块中使用
```javascript
{
    let a=10;
    var b=1;
}
a//a is not defined

b//1

```

var 声明的变量像是 static 的

```javascript
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // a[1]->a[10]的值都是10
```

```javascript
var a = [];
for (let i = 0; i < 10; i++) {//变量 i 使用 let 声明只在本轮循环有效,每次循环 值都是一个新的变量
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
a[5]();//5
```

### 不存在变量提升

let 声明的变量一定要在声明后使用,否则报错
```javascript
console.log(foo); // 输出undefined,foo 在此时已经存在
console.log(bar); // 报错ReferenceError

var foo = 2;
let bar = 2;
```

### temporal dead zone

只要块级作用于存在`let`和`const`命令, 凡是在声明之前使用这些变量就会报错

```javascript
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError: tmp is not defined
  let tmp;
}
```
### 不允许重复声明
```javascript
//let不允许在相同作用域内，重复声明同一个变量。

// 报错
function () {
  let a = 10;
  var a = 1;
}

// 报错
function () {
  let a = 10;
  let a = 1;
}
//因此，不能在函数内部重新声明参数。

function func(arg) {
  let arg; // 报错
}

function func(arg) {
  {
    let arg; // 不报错
  }
}

```


## 2 块级作用域
### 为什么需要块级作用域？
第一种场景，内层变量可能会覆盖外层变量。
```javascript
var tmp = new Date();

function f(){
  console.log(tmp);
  if (false){
    var tmp = "hello world";
  }
}

f() // undefined  内层的tmp变量覆盖了外层的tmp变量。
```

第二种场景，用来计数的循环变量泄露为全局变量。
```javascript
var s = 'hello';

for (var i = 0; i < s.length; i++){
  console.log(s[i]);
}

console.log(i); // 5

```

### ES6的块级作用域
let实际上为JavaScript新增了块级作用域。

```javascript
function f1() {
  let n = 5;
  if (true) {
    let n = 10;
      console.log(n); // 10

  }
  console.log(n); // 5
}


{
  let a = 'secret';
  function f() {
    return a;
  }
}
f() // 报错
//上面代码中，块级作用域外部，无法调用块级作用域内部定义的函数。如果确实需要调用，就要像下面这样处理。

let f;
{
  let a = 'secret';
  f = function () {
    return a;
  }
}
f() // "secret"

```



## 3 const 命令

const 声明常量,类似于 java 中的 final,声明后无法修改其值

```javascript
'use strict';
const PI = 3.1415;
PI // 3.1415

PI = 3;
// TypeError: "PI" is read-only

//常规模式下不会报错,但是修改无效
const PI = 3.1415;
PI = 3; // 常规模式时，重新赋值无效，但不报错
PI // 3.1415

```

const 声明的变量不得改变值, const 一旦声明变量则需要给其赋值,不能在后面赋值,这点与 java 不同.

```javascript
'use strict';
const foo;
// SyntaxError: missing = in const declaration
```
const的作用域与let命令相同：只在声明所在的块级作用域内有效。
```javascript
if (true) {
  const MAX = 5;
}

MAX // Uncaught ReferenceError: MAX is not defined
```

不可重复声明

```javascript
var message = "Hello!";
let age = 25;

// 以下两行都会报错
const message = "Goodbye!";
const age = 30;
```
将对象声明为常量,特性与 java 一样

```javascript
const foo = {};
foo.prop = 123;

foo.prop
// 123

foo = {} // TypeError: "foo" is read-only

const a = [];
a.push("Hello"); // 可执行
a.length = 0;    // 可执行
a = ["Dave"];    // 报错

```

如果想将对象冻结,使用`Object.freeze`方法

```javascript
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;
```
## 4 跨模块常量

```javascript
// constants.js 模块
export const A = 1;
export const B = 3;
export const C = 4;

// test1.js 模块
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3

// test2.js 模块
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3
```

## 5 全局对象属性

全局对象是最顶层的对象，在浏览器环境指的是window对象，在Node.js指的是global对象。ES5之中，全局对象的属性与全局变量是等价的。



# 二. 变量的解构赋值

ES6允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。

## 1 数组的解构赋值

```javascript
//以前，为变量赋值，只能直接指定值。

var a = 1;
var b = 2;
var c = 3;

//ES6允许写成下面这样。
var [a, b, c] = [1, 2, 3];

```

只要等号两边的模式相同，左边的变量就会被赋予对应的值

```javascript
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

let [x, , y] = [1, 2, 3];
x // 1
y // 3

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```

**不完全解构**，即等号左边的模式，只匹配一部分的等号右边的数组


```javascript
let [x, y] = [1, 2, 3];
x // 1
y // 2

let [a, [b], d] = [1, [2, 3], 4];
a // 1
b // 2
d // 4

```

对于`Set`结构，也可以使用数组的解构赋值。
```javascript
let [x, y, z] = new Set(["a", "b", "c"])
x // "a"

```

结构赋值允许指定默认值

```javascript
var [foo = true] = [];
foo // true

[x, y = 'b'] = ['a'] // x='a', y='b'
[x, y = 'b'] = ['a', undefined] // x='a', y='b'
```
惰性赋值，在用到的时候才会求值

```javascript
function f(){
  console.log('aaa');
}

let [x = f()] = [1];

//经过babel翻译后的代码

var _ = 1;
var x = _ === undefined ? f() : _;
```

默认值可以引用解构赋值的其他变量，但该变量必须已经声明。
```javascript
let [x = 1, y = x] = [];     // x=1; y=1
let [x = 1, y = x] = [2];    // x=2; y=2
let [x = 1, y = x] = [1, 2]; // x=1; y=2
let [x = y, y = 1] = [];     // ReferenceError

```

## 2. 对象的解构赋值（重要！！）

与数组不同的是，对象属性没有次序，变量必须与属性同名才能取到正确值

```javascript

var { bar, foo } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"

///相对应下面代码

var _foo$bar = { foo: "aaa", bar: "bbb" };
var bar = _foo$bar.bar;
var foo = _foo$bar.foo;


var { baz } = { foo: "aaa", bar: "bbb" };
baz // undefined

let { log, sin, cos } = Math;
/////////
var log = Math.log;
var sin = Math.sin;
var cos = Math.cos;
```

变量可以重命名
```javascript

var { foo: baz } = { foo: "aaa", bar: "bbb" };
baz // "aaa"
////
var _foo$bar = { foo: "aaa", bar: "bbb" };
var baz = _foo$bar.foo;


let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f // 'hello'
l // 'world'
//////////
var obj = { first: 'hello', last: 'world' };
var f = obj.first;
var l = obj.last;

```

## 3. 函数参数的解构赋值


```javascript

function move({x = 0, y = 0} = {}) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]


////--转义后--

function move() {
  var _ref = arguments.length <= 0 || arguments[0] === undefined ? {} : arguments[0];

  var _ref$x = _ref.x;
  var x = _ref$x === undefined ? 0 : _ref$x;
  var _ref$y = _ref.y;
  var y = _ref$y === undefined ? 0 : _ref$y;

  return [x, y];
}

```

另一种写法

```javascript
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
////////
function move() {
  var _ref = arguments.length <= 0 || arguments[0] === undefined ? { x: 0, y: 0 } : arguments[0];

  var x = _ref.x;
  var y = _ref.y;

  return [x, y];
}

```
## 6.其他解构

字符串解构

```javascript
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"

let {length : len} = 'hello';
len // 5
```

数值解构
```javascript
let {toString: s} = 123;
//////

var _ = 123;
var s = _.toString;
```
## 7 用途

交换变量值

```javascript
[x, y] = [y, x];
/////
var _ref2 = [y, x];
x = _ref2[0];
y = _ref2[1];

```

函数返回多个值

```javascript
// 返回一个数组

function example() {
  return [1, 2, 3];
}
var [a, b, c] = example();

// 返回一个对象

function example() {
  return {
    foo: 1,
    bar: 2
  };
}
var { foo, bar } = example();

```

函数参数定义

```javascript

// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3])

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1})

```
提取Json数据

```javascript
var jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
}

let { id, status, data: number } = jsonData;

console.log(id, status, number)
// 42, "OK", [867, 5309]

//////
var id = jsonData.id;
var status = jsonData.status;
var number = jsonData.data;

```
遍历MAP结构
```javascript

var map = new Map();
map.set('first', 'hello');
map.set('second', 'world');

for (let [key, value] of map) {
  console.log(key + " is " + value);
}
// first is hello
// second is world

```


**模块导入**

```javascript
const { SourceMapConsumer, SourceNode } = require("source-map");
```

# 三. 类型
ECMAScript 有 5 种原始类型，即 Undefined、Null、Boolean、Number 和 String。

## 1. Undefined
如前所述，Undefined 类型只有一个值，即 `undefined`。当声明的变量**未初始化**时，该变量的默认值是` undefined`。


## 2. Null

Null 类型只有一个值`null` ，值 undefined 实际上是从值 null 派生来的

```javascript
null == undefined;//true
null === undefined;//false
```

## 3. Number

八进制数和十六进制数,ES5 之后八进制用`0o`表示
```javascript
var iNum = 0o70;  //070 等于十进制的 56
var iNum = 0x1f;  //0x1f 等于十进制的 31

```
如果要将0b和0x前缀的字符串数值转为十进制，要使用Number方法。
```javascript
Number('0b111')  // 7
Number('0o10')  // 8
```


## 4. String

## 5. 类型转换


转换成字符串 `toString()` 方法
```javascript
arrayObject.toString()
booleanObject.toString()
dateObject.toString()
NumberObject.toString()
stringObject.toString()
```
转换成数字parseInt() 、 parseFloat()


```javascript

var iNum1 = parseInt("12345red");	//返回 12345
var iNum1 = parseInt("0xA");	//返回 10
var iNum1 = parseInt("56.9");	//返回 56
var iNum1 = parseInt("red");	//返回 NaN

var fNum1 = parseFloat("12345red");	//返回 12345
var fNum2 = parseFloat("0xA");	//返回 NaN
var fNum3 = parseFloat("11.2");	//返回 11.2
var fNum4 = parseFloat("11.22.33");	//返回 11.22
var fNum5 = parseFloat("0102");	//返回 102
var fNum1 = parseFloat("red");	//返回 NaN
````


## 6.强制类型转换

ECMAScript 中可用的 3 种强制类型转换如下：
* Boolean(value) - 把给定的值转换成 Boolean 型；
* Number(value) - 把给定的值转换成数字（可以是整数或浮点数）；
* String(value) - 把给定的值转换成字符串；

# 四. 数组

## 1. Array.from()

Array.from方法用于将两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括ES6新增的数据结构Set和Map）所谓类似数组的对象，本质特征只有一点，即必须有length属性,因此，任何有length属性的对象，都可以通过Array.from方法转为数组。

```javascript
Array.from('hello')
// ['h', 'e', 'l', 'l', 'o']

let namesSet = new Set(['a', 'b'])
Array.from(namesSet) // ['a', 'b']


```

`...`运算符也可以将某些结构转换成数组

```javascript
function foo() {
  var args = [...arguments];
}
////////

function foo() {
    var args = [].concat(_slice.call(arguments));
}


var t=[..."helloworld"] //["h","e","l","l","o","w","o","r","l","d"]

```

Array.from还可以接受第二个参数，用来对每个元素进行处理，将处理后的值放入返回的数组。

```javascript
Array.from(arrayLike, x => x * x);
///////
Array.from(arrayLike, function (x) {
    return x * x;
});


Array.from([1, 2, 3], (x) => x * x)// [1, 4, 9]

////////
Array.from([1, 2, 3], function (x) {
    return x * x;
});
```


## 2. Array.of()

Array.of方法用于将一组值，转换为数组。

```javascript

Array.of(3, 11, 8) // [3,11,8]
Array.of(3) // [3]
Array.of(3).length // 1

//Array.of方法可以用下面的代码模拟实现。

function ArrayOf(){
  return [].slice.call(arguments);
}

```
 ## 3. find() findIndex()
 
 find方法用于找出第一个符合条件的数组成员，参数是一个回调函数
 
 ```javascript
 [1, 4, -5, 10].find((n) => n < 0)//-5
/////

[1, 4, -5, 10].find(function (n) {
  return n < 0;
});


[1, 5, 10, 15].find(function(value, index, arr) {//依次为当前的值、当前的位置和原数组。
  return value > 9;
}) // 10

 ```

findIndex方法用于返回符合条件值的位置，如果没有符合的返回-1

```javascript

[1, 5, 10, 15].findIndex(function(value, index, arr) {
  return value > 9;
}) // 2
```

## 5. fill()

填充数组

```javascript
['a', 'b', 'c'].fill(7)
// [7, 7, 7]

new Array(3).fill(7)
// [7, 7, 7]

```
## 6. includes()

用来检测数组里是否包含某个值

```javascript
[1, 2, 3].includes(2);     // true
[1, 2, 3].includes(4);     // false
[1, 2, NaN].includes(NaN); // true
```


## 7. 数组推导

使用现有数组生成新数组

```javascript

var a1 = [1, 2, 3, 4];
var a2 = [for (i of a1) i * 2];

a2 // [2, 4, 6, 8]
 
var years = [ 1954, 1974, 1990, 2006, 2010, 2014 ];

[for (year of years) if (year > 2000) year];
// [ 2006, 2010, 2014 ]

[for (year of years) if (year > 2000) if(year < 2010) year];
// [ 2006]

[for (year of years) if (year > 2000 && year < 2010) year];
// [ 2006]


[for (i of [1, 2, 3]) i * i];
// 等价于
[1, 2, 3].map(function (i) { return i * i });

[for (i of [1,4,2,3,-8]) if (i < 3) i];
// 等价于
[1,4,2,3,-8].filter(function(i) { return i < 3 });

```


# 五. 函数

## 1. 参数默认值

ES6可以为函数参数设置默认值

```javascript
function log(x, y = 'World') {
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello


function Point(x = 0, y = 0) {
  this.x = x;
  this.y = y;
}

var p = new Point();
p // { x: 0, y: 0 }
```

可以结合解构赋值使用


```javascript

function foo({x, y = 5}) {
  console.log(x, y);
}

foo({}) // undefined, 5
foo({x: 1}) // 1, 5
foo({x: 1, y: 2}) // 1, 2
foo() // TypeError: Cannot read property 'x' of undefined

```

```javascript
function fetch(url, { body = '', method = 'GET', headers = {} }){
  console.log(method);
}

fetch('http://example.com', {})
// "GET"

fetch('http://example.com')
// 报错

```

上面这种写法不能省略第二个参数，如果给第二个参数默认值则可以省略

```javascript
function fetch(url, { method = 'GET' } = {}){
  console.log(method);
}

fetch('http://example.com')
// "GET"

```
**需注意的情况**

```javascript
// 写法一
function m1({x = 0, y = 0} = {}) {
  return [x, y];
}

// 写法二
function m2({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}




// 函数没有参数的情况
m1() // [0, 0]
m2() // [0, 0]

// x和y都有值的情况
m1({x: 3, y: 8}) // [3, 8]
m2({x: 3, y: 8}) // [3, 8]

// x有值，y无值的情况
m1({x: 3}) // [3, 0]
m2({x: 3}) // [3, undefined]

// x和y都无值的情况
m1({}) // [0, 0];
m2({}) // [undefined, undefined]

m1({z: 3}) // [0, 0]
m2({z: 3}) // [undefined, undefined]


```

