---
title: es6学习
date: 2018-10-25 10:46:07
categories: ['前端'] 
tags: 前端
comments: true
---
# es5缺陷与es6解决
## 数组
ES5 内建的forEach方法 缺陷：无法使用break 中断 ，也不能使用return 语句返回到外层函数
### 1. 遍历数组
① for forEach
```javascript
var arr = [1,2,3,4,5,6];
arr.name = 'a';
for (var index = 0; index < arr.length; index++) {
  console.log(arr[index]);
}
arr.forEach(function (value) { //ES5 内建的forEach方法 缺陷：无法使用break 中断 ，也不能使用return 语句返回到外层函数
  console.log(value);
});
// 结果都是：1,2,3,4,5,6
```
② 用 for-in ：作用于数组的 forfor -in 循环体除了遍历数组元素外,还会遍历自定义属性。比如数组有一个可枚举属性arr.a,循环将额外执行一次
```javascript
for (var index in arr) { // 千万别这样做
 console.log(arr[index]);
}
// 结果：1,2,3,4,5,6，a
```
for-in 是为普通对象设计的，赋值给index的值不是实际的数字1、2，而是字符串‘1'，‘2'
```javascript
var b = 0;
for (var index in arr) {
 b = b+ index;
 console.log(b)
}
// 结果：00，001，0012，00123，001234，0012345，0012345name
```
③ 使用 for-of：避开了for-in 的所有缺陷，可以正确响应 break、return 语句\
```javascript
for(var value of arr){
  console.log(value)
}
// 结果：1,2,3,4,5,6
```
### 2.for-of 循环便利其他集合
① 遍历Set
```javascript
var words = 'a';
var s = new Set();
s.add("a");
s.add(1);
for(var word of s){
  console.log(word);
}
// 结果：a，1
```
② 遍历Map 
```javascript
var map = new Map();
map.set('a',1);
map.set('b',2);
map.set('c',3);
map.set('d',4);
for(var [key,value] of map){
  console.log(key+':'+value);
}
// 结果：a：1，b：2，c：3，d：4
```
### 3. Iterator（遍历器）
① 遍历器（Iterator）是一种接口规格，任何对象只要部署这个接口，就可以完成遍历操作。它的作用有两个，一是为各种数据结构，提供一个统一的、简便的接口，二是使得对象的属性能够按某种次序排列。

② 遍历器的原理：遍历器提供了一个指针，指向当前对象的某个属性，使用next方法，就可以将指针移动到下一个属性。next方法返回一个包含value和done两个属性的对象。其中，value属性是当前遍历位置的值，done属性是一个布尔值，表示遍历是否结束。
```javascript
//模拟遍历器原理
function makeIterator(array){
  var nextIndex = 0;
  return {
    next: function(){
      return nextIndex < array.length ?
        {value: array[nextIndex++], done: false} :
        {value: undefined, done: true};
    }
  }
}
var it = makeIterator(['a', 'b']);
console.log(it.next());//{ value: 'a', done: false }
console.log(it.next());//{ value: 'b', done: false }
console.log(it.next());//{ value: undefined, done: true }
```
③ Iterator接口返回的遍历器，原生具备next方法。

>   有三类数据结构原生具备Iterator接口：数组、类似数组的对象、Set和Map结构

能调用遍历器接口的
1. for...of
1. Array.from()
1. Map(), Set(), WeakMap(), WeakSet()（比如new Map([['a',1],['b',2]])）
1. Promise.all()
1. Promise.race()
```javascript
var map = new Map();
console.log(map[Symbol.iterator] === map.entries)//true
var arr = new Array();
console.log(arr[Symbol.iterator] === arr.values)//true
var set = new Set();
console.log(set[Symbol.iterator] === set.values)//true
```
> 其他数据结构（主要是对象）如果需要Iterator接口,都需要自己部署。
```javascript
var students = {}
students[Symbol.iterator] = function() {
 let index = 1;
 return {
  next() {
   return {done: index>10, value: index++}
  }
 }
}
for(var i of students) {
 console.log(i);
}//
```

# es6
## 字符串的扩展

### includes(), startsWith(), endsWith()

JavaScript 只有indexOf方法，可以用来确定一个字符串是否包含在另一个字符串中。ES6 又提供了三种新方法。

includes()：返回布尔值，表示是否找到了参数字符串。             第二参数数字 表示起始位置
startsWith()：返回布尔值，表示参数字符串是否在原字符串的头部。 第二参数数字 表示起始位置
endsWith()：返回布尔值，表示参数字符串是否在原字符串的尾部。   第二参数数字 表示截止位置

```javascript
let s = 'Hello world!';

s.startsWith('world', 6); // true
s.endsWith('Hello', 5); // true
s.includes('Hello', 6); // false

```
### repeat()
repeat方法返回一个新字符串，表示将原字符串重复n次。

```javascript
'x'.repeat(3); // "xxx"
'hello'.repeat(2); // "hellohello"
'na'.repeat(0); // ""

```
正小数,向下取整
0- -1小数 为 -0  视为 0

参数字符串,NaN 会优先转为数字,转不了的话 为0

## padStart()，padEnd()

```javascript
'x'.padStart(5, 'ab') // 'ababx'
'x'.padStart(4, 'ab') // 'abax'

'x'.padEnd(5, 'ab') // 'xabab'
'x'.padEnd(4, 'ab') // 'xaba'

```
如果用来补全的字符串与原字符串，两长度之和超过了指定的最小长度，则会截去超出位数的补全字符串。

如果省略第二个参数，默认使用空格补全长度。
```javascript
'abc'.padStart(10, '0123456789')
// '0123456abc'

'x'.padStart(4) // '   x'
'x'.padEnd(4) // 'x   '
```
padStart的常见用途是为数值补全指定位数。下面代码生成 10 位的数值字符串。

另一个用途是提示字符串格式
```javascript
'1'.padStart(10, '0') // "0000000001"
'12'.padStart(10, '0') // "0000000012"
'123456'.padStart(10, '0') // "0000123456"

'12'.padStart(10, 'YYYY-MM-DD') // "YYYY-MM-12"
'09-12'.padStart(10, 'YYYY-MM-DD') // "YYYY-09-12"
```

## matchAll()
matchAll方法返回一个正则表达式在当前字符串的所有匹配，详见《正则的扩展》的一章

## 模板字符串 `${变量名}`

```javascript
let a = 'xiaojw'
let b =`我是${a}哟!`
console.log(`可以直接换行
哟!`)
```

## 正则的扩展

### 1.RegExp 构造函数 
```javascript
// 允许
var regex = new RegExp('xyz', 'i');
var regex = new RegExp(/xyz/i);
// 等价于
var regex = /xyz/i;
// es5不允许 es6允许
var regex = new RegExp(/xyz/, 'i');

```
### 2.字符串的正则方法
字符串对象共有 4 个方法，可以使用正则表达式：match()、replace()、search()和split()。

### 3.u修饰符
ES6 对正则表达式添加了u修饰符，含义为“Unicode 模式”，用来正确处理大于\uFFFF的 Unicode 字符。也就是说，会正确处理四个字节的 UTF-16 编码


```javascript
// 点字符
var s = '𠮷';

/^.$/.test(s); // false
/^.$/u.test(s); // true
// 如果不添加u修饰符，正则表达式就会认为字符串为两个字符，从而匹配失败。

// 2.Unicode 字符表示法
/\u{61}/.test('a'); // false
/\u{61}/u.test('a'); // true
/\u{20BB7}/u.test('𠮷'); // true

// 3.量词
/a{2}/.test('aa'); // true
/a{2}/u.test('aa'); // true
/𠮷{2}/.test('𠮷𠮷'); // false
/𠮷{2}/u.test('𠮷𠮷'); // true

// 4.预定义模式
/^\S$/.test('𠮷'); // false
/^\S$/u.test('𠮷'); // true

// 返回字符串长度的函数
function codePointLength(text) {
  var result = text.match(/[\s\S]/gu);
  return result ? result.length : 0;
}
var s = '𠮷𠮷';
s.length; // 4
codePointLength(s) // 2

// 5.i 修饰符
/[a-z]/i.test('\u212A'); // false
/[a-z]/iu.test('\u212A'); // true
```
### 4.RegExp.prototype.unicode 属性
```javascript
const r1 = /hello/;
const r2 = /hello/u;

r1.unicode; // false
r2.unicode; // true

```
### 5.y 修饰符
y修饰符与g修饰符类似,全局搜索
但与g不同的地方是需要严格从上一次匹配的位置进行下一次匹配 且y修饰符号隐含了头部匹配的标志^。
```javascript
var s = 'aaa_aa_a';
var r1 = /a+/g;
var r2 = /a+/y;

r1.exec(s) // ["aaa"]
r2.exec(s) // ["aaa"]

r1.exec(s) // ["aa"]  剩余字符 _aa_a
r2.exec(s) // null    剩余字符 _aa_a   _匹配不上a

```
### 6.RegExp.prototype.sticky 属性
与y修饰符相匹配，ES6 的正则实例对象多了sticky属性，表示是否设置了y修饰符。
```javascript
var r = /hello\d/y;
r.sticky // true
```
### 7.RegExp.prototype.flags
ES6 为正则表达式新增了flags属性，会返回正则表达式的修饰符。

### 8.s 修饰符：dotAll 模式
这被称为dotAll模式，即点（dot）代表一切字符。但是有两个例外。一个是四个字节的 UTF-16 字符，这个可以用u修饰符解决；另一个是行终止符（line terminator character）。

1.U+000A 换行符（\n）
2.U+000D 回车符（\r）
3.U+2028 行分隔符（line separator）
4.U+2029 段分隔符（paragraph separator

所以，正则表达式还引入了一个dotAll属性，返回一个布尔值，表示该正则表达式是否处在dotAll模式。

### 9.后行断言

(pattern) ：  匹配 pattern 并获取这一匹配，所获取的匹配可以从产生的 Matches 集合得到。 
(?:pattern) ：匹配 pattern 但不获取匹配结果，也就是说这是一个非获取匹配，不进行存储供以后使用。 
(?!pattern) ：匹配 !pattern 但不获取匹配结果，也就是说这是一个非获取匹配，不进行存储供以后使用。 
(?=pattern) ：正向预查，在任何匹配 pattern 的字符串开始处匹配查找字符串。这是一个非获取匹配，也就是说，该匹配不需要获取供以后使用。预查不消耗字符，也就是说，在一个匹配发生后，在最后一次匹配之后立即开始下一次匹配的搜索，而不是从包含预查的字符之后开始。


(pattern)匹配pattern并捕获该匹配的子表达式.可以使用 $0...$9属性从结果"匹配"集合中检索捕获的匹配.若要匹配括号字符(),请使用""或者""或者"".

(?:pattern)匹配pattern但不捕获该匹配的子表达式,即它是一个非捕获匹配,不存储供以后使用的匹配.这对于用"或"字符(|)组合模式部件的情况很有用.
  例如,与"industry|industries"相比,"industr(?:y|ies)"是一个更加经济的表达式.

let RegExp = /industr(y|ies)/      // 存储
let RegExp1 = /industr(?:y|ies)/   // ?:不存储
let result1 = 'industry'.match(RegExp)
let result2 = 'industries'.match(RegExp)

let result3 = 'industry'.match(RegExp1)
let result4 = 'industries'.match(RegExp1)
	  
(?=pattern)执行正向预测先行搜索的子表达式,该表达式匹配处于匹配 pattern 的字符串的起始点的字符串.它是一个非捕获匹配,即不能捕获供以后使用的匹配.
  例如,"Windows (?=95| 98| NT| 2000)"与"Windows 2000"中的"Windows"匹配,但不与"Windows 3.1"中的"Windows"匹配.
        预测先行不占用字符,即发生匹配后,下一匹配的搜索紧随上一匹配之后,而不是在组成预测先行的字符后.

(?!pattern)执行反向预测先行搜索的子表达式,该表达式匹配不处于匹配 pattern 的字符串的起始点的搜索字符串.它是一个非捕获匹配,即不能捕获供以后使用的匹配.
  例如,"Windows (?!95| 98| NT| 2000)"与"Windows 3.1"中的"Windows"匹配,但不与"Windows 2000"中的"Windows"匹配.
        预测先行不占用字符,即发生匹配后,下一匹配的搜索紧随上一匹配之后,而不是在组成预测先行的字符后.

--------------------- 
作者：warmsmellofcolitas 
来源：CSDN 
原文：https://blog.csdn.net/warmsmellofcolitas/article/details/79403706 
版权声明：本文为博主原创文章，转载请附上博文链接！

### 10.Unicode 属性类
ES2018 引入了一种新的类的写法\p{...}和\P{...}，允许正则表达式匹配符合 Unicode 某种属性的所有字符。
```javascript
const regexGreekSymbol = /\p{Script=Greek}/u;
regexGreekSymbol.test('π') // true
```
上面代码中，\p{Script=Greek}指定匹配一个希腊文字母，所以匹配π成功

Unicode 属性类要指定属性名和属性值
对于某些属性，可以只写属性名，或者只写属性值。
```javascript
/\p{UnicodePropertyName=UnicodePropertyValue}/
/\p{UnicodePropertyName}/
/\p{UnicodePropertyValue}/
```
\P{…}是\p{…}的反向匹配，即匹配不满足条件的字符。
注意，这两种类只对 Unicode 有效，所以使用的时候一定要加上u修饰符。如果不加u修饰符，正则表达式使用\p和\P会报错，ECMAScript 预留了这两个类。
由于 Unicode 的各种属性非常多，所以这种新的类的表达能力非常强
```javascript
const regex = /^\p{Decimal_Number}+$/u;
regex.test('𝟏𝟐𝟑𝟜𝟝𝟞𝟩𝟪𝟫𝟬𝟭𝟮𝟯𝟺𝟻𝟼'); // true
```
上面代码中，属性类指定匹配所有十进制字符，可以看到各种字型的十进制字符都会匹配成功。

\p{Number}甚至能匹配罗马数字。
```javascript
// 匹配所有数字
const regex = /^\p{Number}+$/u;
regex.test('²³¹¼½¾'); // true
regex.test('㉛㉜㉝'); // true
regex.test('ⅠⅡⅢⅣⅤⅥⅦⅧⅨⅩⅪⅫ'); // true
```
例子
```javascript
// 匹配所有空格
/\p{White_Space}/

// 匹配各种文字的所有字母，等同于 Unicode 版的 \w
/[\p{Alphabetic}\p{Mark}\p{Decimal_Number}\p{Connector_Punctuation}\p{Join_Control}]/

// 匹配各种文字的所有非字母的字符，等同于 Unicode 版的 \W
/[^\p{Alphabetic}\p{Mark}\p{Decimal_Number}\p{Connector_Punctuation}\p{Join_Control}]/

// 匹配 Emoji
/\p{Emoji_Modifier_Base}\p{Emoji_Modifier}?|\p{Emoji_Presentation}|\p{Emoji}\uFE0F/gu

// 匹配所有的箭头字符
const regexArrows = /^\p{Block=Arrows}+$/u;
regexArrows.test('←↑→↓↔↕↖↗↘↙⇏⇐⇑⇒⇓⇔⇕⇖⇗⇘⇙⇧⇩') // true
```
### 11.具名组匹配
正则表达式使用圆括号进行组匹配。
```javascript
const RE_DATE = /(\d{4})-(\d{2})-(\d{2})/;
```
上面代码中，正则表达式里面有三组圆括号。使用exec方法，就可以将这三组匹配结果提取出来
```javascript
const RE_DATE = /(\d{4})-(\d{2})-(\d{2})/;

const matchObj = RE_DATE.exec('1999-12-31');
const year = matchObj[1]; // 1999
const month = matchObj[2]; // 12
const day = matchObj[3]; // 31
```
组匹配的一个问题是，每一组的匹配含义不容易看出来，而且只能用数字序号（比如matchObj[1]）引用，要是组的顺序变了，引用的时候就必须修改序号。

ES2018 引入了具名组匹配（Named Capture Groups），允许为每一个组匹配指定一个名字，既便于阅读代码，又便于引用。
```javascript
const RE_DATE = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;

const matchObj = RE_DATE.exec('1999-12-31');
const year = matchObj.groups.year; // 1999
const month = matchObj.groups.month; // 12
const day = matchObj.groups.day; // 31
// 数字序号（matchObj[1]）依然有效
```
没有找到匹配,属性值就是undefined

#### 解构赋值和替换
有了具名组匹配以后，可以使用解构赋值直接从匹配结果上为变量赋值。
```javascript
let {groups: {one, two}} = /^(?<one>.*):(?<two>.*)$/u.exec('foo:bar');
one  // foo
two  // bar
```
字符串替换时，使用$<组名>引用具名组
```javascript
let re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/u;

'2015-01-02'.replace(re, '$<day>/$<month>/$<year>')
// '02/01/2015'
```
上面代码中，replace方法的第二个参数是一个字符串，而不是正则表达式。

replace方法的第二个参数也可以是函数，该函数的参数序列如下。
```javascript
'2015-01-02'.replace(re, (
   matched, // 整个匹配结果 2015-01-02
   capture1, // 第一个组匹配 2015
   capture2, // 第二个组匹配 01
   capture3, // 第三个组匹配 02
   position, // 匹配开始的位置 0
   S, // 原字符串 2015-01-02
   groups // 具名组构成的一个对象 {year, month, day}
 ) => {
 let {day, month, year} = groups;
 return `${day}/${month}/${year}`;
});
```
具名组匹配在原来的基础上，新增了最后一个函数参数：具名组构成的一个对象。函数内部可以直接对这个对象进行解构赋值。

#### 引用
如果要在正则表达式内部引用某个“具名组匹配”，可以使用\k<组名>的写法。

```javascript
const RE_TWICE = /^(?<word>[a-z]+)!\k<word>$/;
RE_TWICE.test('abc!abc'); // true
RE_TWICE.test('abc!ab'); // false
```
数字引用（\1）依然有效
```javascript
const RE_TWICE = /^(?<word>[a-z]+)!\1$/;
RE_TWICE.test('abc!abc'); // true
RE_TWICE.test('abc!ab'); // false
```
这两种引用语法还可以同时使用。
```javascript
const RE_TWICE = /^(?<word>[a-z]+)!\k<word>!\1$/;
RE_TWICE.test('abc!abc!abc') // true
RE_TWICE.test('abc!abc!ab') // false
```

### 12.String.prototype.matchAll
如果一个正则表达式在字符串里面有多个匹配，现在一般使用g修饰符或y修饰符，在循环里面逐一取出。

```javascript
var regex = /t(e)(st(\d?))/g;
var string = 'test1test2test3';

var matches = [];
var match;
while (match = regex.exec(string)) {
  matches.push(match);
}

matches
// [
//   ["test1", "e", "st1", "1", index: 0, input: "test1test2test3"],
//   ["test2", "e", "st2", "2", index: 5, input: "test1test2test3"],
//   ["test3", "e", "st3", "3", index: 10, input: "test1test2test3"]
// ]
```
上面代码中，while循环取出每一轮的正则匹配，一共三轮。

目前有一个提案，增加了String.prototype.matchAll方法，可以一次性取出所有匹配。不过，它返回的是一个遍历器（Iterator），而不是数组
```javascript
const string = 'test1test2test3';

// g 修饰符加不加都可以
const regex = /t(e)(st(\d?))/g;

for (const match of string.matchAll(regex)) {
  console.log(match);
}
// ["test1", "e", "st1", "1", index: 0, input: "test1test2test3"]
// ["test2", "e", "st2", "2", index: 5, input: "test1test2test3"]
// ["test3", "e", "st3", "3", index: 10, input: "test1test2test3"]
```
上面代码中，由于string.matchAll(regex)返回的是遍历器，所以可以用for...of循环取出。相对于返回数组，返回遍历器的好处在于，如果匹配结果是一个很大的数组，那么遍历器比较节省资源。

遍历器转为数组是非常简单的，使用...运算符和Array.from方法就可以了。
```javascript
// 转为数组方法一
[...string.matchAll(regex)]

// 转为数组方法二
Array.from(string.matchAll(regex));
```
###### 注:javascript中for...in和for...of的区别,以下括号中注的解释
{
for...of循环是ES6引入的新的语法。
for...in遍历拿到的x是键（下标）。而for...of遍历拿到的x是值，但在对象中会提示不是一个迭代器报错。
```javascript
let x;
let a = ['A','B','C'];
let b = {name: '刘德华',age: '18'};

console.log(a.length);
for(x of a){
　　console.log(x); //A,B,C
}
for(x in a){
　　console.log(x+':'+a[x]); //0:A,1:B,2:C
}
/*for(x of b){
　　console.log(x); //报错
}*/
for(x in b){
　　console.log(x); //name,age
}
```
}

## 函数的扩展

### 1.函数参数的默认值
```javascript
function log(x, y = 'World') {
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello
```
#### 与解构赋值默认值结合使用
```javascript
function fetch(url, { body = '', method = 'GET', headers = {} }) {
  console.log(method);
}

fetch('http://example.com', {})
// "GET"

fetch('http://example.com')
// 报错
```
两个写法的区别
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

// x 和 y 都有值的情况
m1({x: 3, y: 8}) // [3, 8]
m2({x: 3, y: 8}) // [3, 8]

// x 有值，y 无值的情况
m1({x: 3}) // [3, 0]
m2({x: 3}) // [3, undefined]

// x 和 y 都无值的情况
m1({}) // [0, 0];
m2({}) // [undefined, undefined]

m1({z: 3}) // [0, 0]
m2({z: 3}) // [undefined, undefined]
```
#### 函数的 length 属性
指定了默认值以后，函数的length属性，将返回没有指定默认值的参数个数。也就是说，指定了默认值后，length属性将失真。
```javascript
(function (a) {}).length // 1
(function (a = 5) {}).length // 0
(function (a, b, c = 5) {}).length // 2
```
#### 作用域
```javascript
function f(y = x) {
  let x = 2;
  console.log(y);
}

f() // ReferenceError: x is not defined



var x = 1;

function foo(x = x) {
  // ...
}

foo() // R

```
上面代码中，参数x = x形成一个单独作用域。实际执行的是let x = x，由于暂时性死区的原因，这行代码会报错”x 未定义“。

```javascript
let foo = 'outer';

function bar(func = () => foo) {
  let foo = 'inner';
  console.log(func());
}

bar(); // outer
```
上面代码中，函数bar的参数func的默认值是一个匿名函数，返回值为变量foo。函数参数形成的单独作用域里面，并没有定义变量foo，所以foo指向外层的全局变量foo，因此输出outer。
### 2.rest 参数
ES6 引入 rest 参数（形式为...变量名），用于获取函数的多余参数，这样就不需要使用arguments对象了。rest 参数搭配的变量是一个数组，该变量将多余的参数放入数组中。

```javascript
function add(...values) {
  let sum = 0;

  for (var val of values) {
    sum += val;
  }

  return sum;
}

add(2, 5, 3) // 10
``````

### 3.尾调用优化
#### 什么是尾调用
尾调用（Tail Call）是函数式编程的一个重要概念，本身非常简单，一句话就能说清楚，就是指某个函数的最后一步是调用另一个函数。

```javascript
function f(x){
  return g(x);
}
```
上面代码中，函数f的最后一步是调用函数g，这就叫尾调用。

以下三种情况，都不属于尾调用。
```javascript
// 情况一
function f(x){
  let y = g(x);
  return y;
}

// 情况二
function f(x){
  return g(x) + 1;
}

// 情况三
function f(x){
  g(x);
}
```
上面代码中，情况一是调用函数g之后，还有赋值操作，所以不属于尾调用，即使语义完全一样。情况二也属于调用后还有操作，即使写在一行内。情况三等同于下面的代码。
```javascript
function f(x){
  g(x);
  return undefined;
}
```
尾调用不一定出现在函数尾部，只要是最后一步操作即可。
```javascript
function f(x) {
  if (x > 0) {
    return m(x)
  }
  return n(x);
}
```
上面代码中，函数m和n都属于尾调用，因为它们都是函数f的最后一步操作。
#### 尾调用优化
函数调用会在内存形成一个“调用记录”，又称“调用帧”（call frame），保存调用位置和内部变量等信息。如果在函数A的内部调用函数B，那么在A的调用帧上方，还会形成一个B的调用帧。等到B运行结束，将结果返回到A，B的调用帧才会消失。如果函数B内部还调用函数C，那就还有一个C的调用帧，以此类推。所有的调用帧，就形成一个“调用栈”（call stack）

尾调用由于是函数的最后一步操作，所以不需要保留外层函数的调用帧，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用帧，取代外层函数的调用帧就可以了。
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

这就叫做“尾调用优化”（Tail call optimization），即只保留内层函数的调用帧。如果所有函数都是尾调用，那么完全可以做到每次执行时，调用帧只有一项，这将大大节省内存。这就是“尾调用优化”的意义。

注意，只有不再用到外层函数的内部变量，内层函数的调用帧才会取代外层函数的调用帧，否则就无法进行“尾调用优化”

```javascript
function addOne(a){
  var one = 1;
  function inner(b){
    return b + one;
  }
  return inner(a);
}
```
上面的函数不会进行尾调用优化，因为内层函数inner用到了外层函数addOne的内部变量one。

### 尾递归
函数调用自身，称为递归。如果尾调用自身，就称为尾递归。

递归非常耗费内存，因为需要同时保存成千上百个调用帧，很容易发生“栈溢出”错误（stack overflow）。但对于尾递归来说，由于只存在一个调用帧，所以永远不会发生“栈溢出”错误。
```javascript
function factorial(n) {
  if (n === 1) return 1;
  return n * factorial(n - 1);
}

factorial(5) // 120
```
上面代码是一个阶乘函数，计算n的阶乘，最多需要保存n个调用记录，复杂度 O(n) 。

如果改写成尾递归，只保留一个调用记录，复杂度 O(1) 。
```javascript
function factorial(n, total) {
  if (n === 1) return total;
  return factorial(n - 1, n * total);
}

factorial(5, 1) // 120
```

非尾递归的 Fibonacci 数列实现如下。
```javascript
function Fibonacci (n) {
  if ( n <= 1 ) {return 1};

  return Fibonacci(n - 1) + Fibonacci(n - 2);
}

// f(3) => f(2)+ f(1) => f(1)+ f(0) + f(1) => 3
Fibonacci(3) // 3
Fibonacci(10) // 89
Fibonacci(100) // 堆栈溢出
Fibonacci(500) // 堆栈溢出
```
尾递归优化过的 Fibonacci 数列实现如下。
```javascript
function Fibonacci2 (n , ac1 = 1 , ac2 = 1) {
  if( n <= 1 ) {return ac2};
 //                  (序号,上一次结果,本次结果)  
  return Fibonacci2 (n - 1, ac2, ac1 + ac2);
}

// f(3,1,1)= f(2,1,2)=f(1,2,3) = 3
// f(4,1,1) = f(3,1,2)=f(2,2,3) =f(1,3,5) = 5
Fibonacci2(100) // 573147844013817200000
Fibonacci2(1000) // 7.0330367711422765e+208
Fibonacci2(10000) // Infinity
```
由此可见，“尾调用优化”对递归操作意义重大，所以一些函数式编程语言将其写入了语言规格。ES6 是如此，第一次明确规定，所有 ECMAScript 的实现，都必须部署“尾调用优化”。这就是说，ES6 中只要使用尾递归，就不会发生栈溢出，相对节省内存。

### 严格模式
ES6 的尾调用优化只在严格模式下开启，正常模式是无效的。

这是因为在正常模式下，函数内部有两个变量，可以跟踪函数的调用栈。

func.arguments：返回调用时函数的参数。
func.caller：返回调用当前函数的那个函数。
尾调用优化发生时，函数的调用栈会改写，因此上面两个变量就会失真。严格模式禁用这两个变量，所以尾调用模式仅在严格模式下生效。
```bash
function restricted() {
  'use strict';
  restricted.caller;    // 报错
  restricted.arguments; // 报错
}
restricted();
```

### 尾递归优化的实现
尾递归优化只在严格模式下生效，那么正常模式下，或者那些不支持该功能的环境中，有没有办法也使用尾递归优化呢？回答是可以的，就是自己实现尾递归优化。

它的原理非常简单。尾递归之所以需要优化，原因是调用栈太多，造成溢出，那么只要减少调用栈，就不会溢出。怎么做可以减少调用栈呢？就是采用“循环”换掉“递归”。

下面是一个正常的递归函数。
```javascript
function sum(x, y) {
  if (y > 0) {
    return sum(x + 1, y - 1);
  } else {
    return x;
  }
}

sum(1, 100000)
// Uncaught RangeError: Maximum call stack size exceeded(…)
```
上面代码中，sum是一个递归函数，参数x是需要累加的值，参数y控制递归次数。一旦指定sum递归 100000 次，就会报错，提示超出调用栈的最大次数。

蹦床函数（trampoline）可以将递归执行转为循环执行。
```javascript
function trampoline(f) {
  while (f && f instanceof Function) {
    f = f();
  }
  return f;
}
```
上面就是蹦床函数的一个实现，它接受一个函数f作为参数。只要f执行后返回一个函数，就继续执行。注意，这里是返回一个函数，然后执行该函数，而不是函数里面调用函数，这样就避免了递归执行，从而就消除了调用栈过大的问题。

然后，要做的就是将原来的递归函数，改写为每一步返回另一个函数。
```javascript
function sum(x, y) {
  if (y > 0) {
    return sum.bind(null, x + 1, y - 1);
  } else {
    return x;
  }
}
```
