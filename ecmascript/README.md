
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
//因此，不能在函数内部重新声明参数。d[s

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

**参数默认值的位置**


非尾部参数设置默认值，这个参数是无法省略的

```javascript
function f(x = 1, y) {
  return [x, y];
}
///////
function f(x, y) {
  if (x === undefined) x = 1;

  return [x, y];
}


f() // [1, undefined]
f(2) // [2, undefined])
f(, 1) // 报错,其实本身这种语法就是错误的
f(undefined, 1) // [1, 1]

function f(x, y = 5, z) {
  return [x, y, z];
}

f() // [undefined, 5, undefined]
f(1) // [1, 5, undefined]
f(1, ,2) // 报错
f(1, undefined, 2) // [1, 5, 2]

```

**函数的length属性**

指定了默认值以后，函数的length属性，将返回没有指定默认值的参数个数

```javascript
(function(a){}).length // 1
(function(a = 5){}).length // 0
(function(a, b, c = 5){}).length // 2

```

**默认值参数作用域**

总结起来就是参数使用的变量已经生成则作用域是函数作用域否则更上层的作用域的变量已经生成则使用的是上层作用域的值，如果全局内都没有此变量存在则会报错

情况1：
```javascript
var x = 1;

function f(x, y = x) {
  console.log(y);
}

f(2) // 2

```
情况2：
```javascript
let x = 1;

function f(y = x) {
  let x = 2;
  console.log(y);
}

f() // 1

```
情况3：
```javascript

function f(y = x) {
  let x = 2;
  console.log(y);
}

f() // ReferenceError: x is not defined

```

## 2. rest参数

就是`...变量名`形式的参数，表示不定数量的参数


```javascript
function add(...values) {
  let sum = 0;

  for (var val of values) {
    sum += val;
  }

  return sum;
}

add(2, 5, 3) // 10

```

**rest参数后不能再有其他参数**

```javascript

// 报错
function f(a, ...b, c) {
  // ...
}
```

**注意函数的length属性不包括rest参数**

```javascript
(function(a) {}).length  // 1
(function(...a) {}).length  // 0
(function(a, ...b) {}).length  // 1

```

## 3. 扩展运算符(spread)

扩展运算符用`...`表示，用于将一个数组转换为逗号分隔的参数序列。

```javascript
console.log(...[1, 2, 3])//console.log(1,2,3)
// 1 2 3

console.log(1, ...[2, 3, 4], 5)
// 1 2 3 4 5

```

**主要用于函数调用**

```javascript
//多么扭曲的例子=_=
function f(v, w, x, y, z) { }
var args = [0, 1];
f(-1, ...args, 2, ...[3]);
```


**替代apply方法**

```javascript
// ES5的写法
function f(x, y, z) {
  // ...
}
var args = [0, 1, 2];
f.apply(null, args);

// ES6的写法
function f(x, y, z) {
  // ...
}
var args = [0, 1, 2];
f(...args);

// ES5的写法
Math.max.apply(null, [14, 3, 77])

// ES6的写法
Math.max(...[14, 3, 77])

// 等同于
Math.max(14, 3, 77);

```

**合并数组**

```javascript
// ES5
[1, 2].concat(more)
// ES6
[1, 2, ...more]

var arr1 = ['a', 'b'];
var arr2 = ['c'];
var arr3 = ['d', 'e'];

// ES5的合并数组
arr1.concat(arr2, arr3));
// [ 'a', 'b', 'c', 'd', 'e' ]

// ES6的合并数组
[...arr1, ...arr2, ...arr3]
// [ 'a', 'b', 'c', 'd', 'e' ]

```

**扩展解构赋值**

```javascript
const [first, ...rest] = [1, 2, 3, 4, 5];
first // 1
rest  // [2, 3, 4, 5]

const [first, ...rest] = [];
first // undefined
rest  // []:

const [first, ...rest] = ["foo"];
first  // "foo"
rest   // []

```
需要注意的是将扩展运算符用于数组赋值则只能放在参数最后一位，否则会报错

```javascript

const [...butLast, last] = [1, 2, 3, 4, 5];
// 报错

const [first, ...middle, last] = [1, 2, 3, 4, 5];
// 报错
```

**map set结构**



```javascript
let map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);

let arr = [...map.keys()]; // [1, 2, 3]

```


## 4. name属性

用于返回该函数的函数名

```javascript
var func1 = function () {};

// ES5
func1.name // ""

// ES6
func1.name // "func1"
```
## 5.箭头函数

es6可以使用`=>`定义函数, `=>` 左边代表参数名

```javascript
var f=v=>v;

///等价于///

var f = function(v) {
  return v;
};

```

如果箭头函数不需要参数或者需要多个参数,使用括号代表参数部分

```javascript
var f = () => 5;
// 等同于
var f = function (){ return 5 };

var sum = (num1, num2) => num1 + num2;
// 等同于
var sum = function(num1, num2) {
  return num1 + num2;
};

```

如果方法体多于一行,则需要用`{}`并使用return 返回

```javascript
var sum = (num1, num2) => { return num1 + num2; }

///////
var sum = function sum(num1, num2) {
  return num1 + num2;
};

```

如果箭头部分直接返回对象,则必须在对象外边加上括号


```javascript
var getTempItem = id => ({id:id,name:"Temp"});
///////
var getTempItem = function getTempItem(id) {
  return { id: id, name: "Temp" };
};

```

## 6. 函数绑定

函数绑定运算符是`::`,双冒号左边是对象,右边是一个函数,运算符会自动将左边的对象作为上下文环境,绑定到右边的函数上
```javascript

foo::bar;
// 等同于
bar.bind(foo);

foo::bar(...arguments);
// 等同于
bar.apply(foo, arguments);

const hasOwnProperty = Object.prototype.hasOwnProperty;
function hasOwn(obj, key) {
  return obj::hasOwnProperty(key);
}

```
如果双冒号左边为空，右边是一个对象的方法，则等于将该方法绑定在该对象上面。

```javascript
var method = obj::obj.foo;
// 等同于
var method = ::obj.foo;

let log = ::console.log;
// 等同于
var log = console.log.bind(console);

```


## 7. 尾调用优化

尾调用（Tail Call）是函数式编程的一个重要概念,就是指某个函数的最后一步是调用另一个函数。


```javascript
function f(x){
  return g(x);
}

```

尾调用优化即只保留内层函数的调用帧.

如下例子,g函数调用后f就结束,则在最后执行g的时候f可以不用保留

```javascript
function f() {
  let m = 1;
  let n = 2;
  return g(m + n);
}
f();

// 等同于
function f() {
  return g(3);
}
f();

// 等同于
g(3);

```

**尾递归**

通过尾调用优化可以节省尾递归产生的栈内存


```javascript

function factorial(n, total) {
  if (n === 1) return total;
  return factorial(n - 1, n * total);
}

factorial(5, 1) // 120
```


**尾调用优化只在严格模式下开启**


# 六. 对象


# 七. Proxy和Reflect

# 八. Set和Map数据结构


## 1. Set

**初始化**

```javascript
var s = new Set();

var set = new Set([1, 2, 3, 4, 4])



```
添加数据

```javascript
var s = new Set();

s.add(1);

```


两个对象总是不相等的。
```javascript
let set = new Set();

set.add({})
set.size // 1

set.add({})
set.size // 2

```

**操作方法:**

* add(value)：添加某个值，返回Set结构本身。
* delete(value)：删除某个值，返回一个布尔值，表示删除是否成功。
* has(value)：返回一个布尔值，表示该值是否为Set的成员。
* clear()：清除所有成员，没有返回值。


```javascript

s.add(1).add(2).add(2);
// 注意2被加入了两次

s.size // 2

s.has(1) // true
s.has(2) // true
s.has(3) // false

s.delete(2);
s.has(2) // false

````


**遍历操作:**

* keys()：返回一个键名的遍历器
* values()：返回一个键值的遍历器
* entries()：返回一个键值对的遍历器
* forEach()：使用回调函数遍历每个成员


```javascript
let set = new Set(['red', 'green', 'blue']);

for ( let item of set.keys() ){
  console.log(item);
}
// red
// green
// blue

for ( let item of set.values() ){
  console.log(item);
}
// red
// green
// blue

for ( let item of set.entries() ){
  console.log(item);
}
// ["red", "red"]
// ["green", "green"]
// ["blue", "blue"]

```



## 2. Map

**初始化**

```javascript

var m = new Map();
var o = {p: "Hello World"};

m.set(o, "content")
m.get(o) // "content"

m.has(o) // true
m.delete(o) // true
m.has(o) // false


var map = new Map([["name", "张三"], ["title", "Author"]]);

map.size // 2
map.has("name") // true
map.get("name") // "张三"
map.has("title") // true
map.get("title") // "Author"

```


**操作方法**

* `size`属性: 返回Map成员总数
* `set(key, value)`:设置key所对应的键值
* `get(key)`:读取key对应的键值如果找不到key，返回undefined
* `has(key)`:返回一个布尔值，表示某个键是否在Map数据结构中
* `delete(key)`:删除某个键，返回true。如果删除失败，返回false
* `clear()`:方法清除所有成员，没有返回值


**遍历方法**

* keys()：返回键名的遍历器。
* values()：返回键值的遍历器。
* entries()：返回所有成员的遍历器。
* forEach()：遍历Map的所有成员。

```javascript
let map = new Map([
  ['F', 'no'],
  ['T',  'yes'],
]);

for (let key of map.keys()) {
  console.log(key);
}
// "F"
// "T"

for (let value of map.values()) {
  console.log(value);
}
// "no"
// "yes"

for (let item of map.entries()) {
  console.log(item[0], item[1]);
}
// "F" "no"
// "T" "yes"

// 或者
for (let [key, value] of map.entries()) {
  console.log(key, value);
}

// 等同于使用map.entries()
for (let [key, value] of map) {
  console.log(key, value);
}

```


**与其他结构互转**(TODO:)


# 九. Class

# 十. 异步

JS 是单线程模型的,在浏览器环境下 JS和其他浏览器任务共享同一个线程,在一个任务执行过程中会阻塞其他任务.

我们不能阻塞主线程太长时间,异步就是用来解决这种情况的.

常见的异步编程有以下四种:
- 回调函数
- 事件监听
- 发布/订阅
- Promise 对象



## 1. 回调函数


## 2. 事件监听

在浏览器下的 js 编程中我们经常会写到这样的事件监听代码


```javascript

var image1=document.querySelector('.img-1');

image1.addEventListener('load',()={
//图片加载完成
});

image1.addEventListener('error',()=>{
//出现错误
})
```


## 3. 发布/订阅


## 4. Promise(重要!!)

Promise 是异步编程的一种解决方案


Promise 与事件监听对比来说有以下特点:

* 一个 Promise 只能成功或者失败一次,他不能从成功到失败的转换,反之亦然
* 如果在一个 Promise 成功或者失败后添加了相应的回调函数,那么该回调还是会被立即执行.事件监听机制则不能实现该效果,事件错过了再去监听是无法得到结果的

**Promise 优点**:
* 可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数
* Promise对象提供统一的接口，使得控制异步操作更加容易

**Promise 缺点**:

* 无法取消Promise，一旦新建它就会立即执行，无法中途取消。
* 如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。
* 当处于Pending状态时，无法得知目前进展到哪一个阶段


**Promise有四种状态**
* fulfilled: 操作成功
* rejected: 操作失败
* pending: 操作进行中还没有得到结果
* settled: Promise已经被 fulfiled 或 rejected, 且并不是 pending.


### 4.1 基本用法

生成 Promise 实例
```javascript
var promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});

```


Promise 构造函数接受一个函数作为参数,该函数有两个参数`resolve`和`reject`.这两个函数由 JS 引擎提供不用自己部署.

`resolve`函数作用是将 Promise 对象的状态从**未完成(Pending)**变成**成功(Reolved)**在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；reject函数的作用是，将Promise对象的状态**未完**变为**失败**，在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

Promise实例生成以后，可以用then方法分别指定Resolved状态和Reject状态的回调函数。


### 4.2 链式调用

`then`方法返回的是一个新的 Promise 实例(**不是原来的 Promise 实例**),所以可以采用链式写法:

```javascript
getJSON("/posts.json").then(function(json) {
  return json.post;
}).then(function(post) {
  // ...
});

```

举个例子:

```javascript
var p=new Promise((resolve,reject)=>{
  resolve(1);
});
p.then(val=>val+2).then(val=>{console.log(val)})
//3

```

### 4.3 异步操作队列




### 4.3 例子

异步加载图片的例子。

```javascript

function loadImageAsync(url) {
  return new Promise(function(resolve, reject) {
    var image = new Image();

    image.onload = function() {
      resolve(image);
    };

    image.onerror = function() {
      reject(new Error('Could not load image at ' + url));
    };

    image.src = url;
  });
}

let asyncLoad=loadImageAsync('http://www.baidu.com/img/270new_2219485be6054791b9649fd0d423545f.png')
asyncLoad.then(image=>{
console.log('success')
},error=>{
console.log(error);
})


```


Ajax 例子


```javascript

var getJSON = function(url) {
  var promise = new Promise(function(resolve, reject){
    var client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();

    function handler() {
      if ( this.readyState !== 4 ) {
        return;
      }
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
  });

  return promise;
};

getJSON("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出错了', error);
});
```
