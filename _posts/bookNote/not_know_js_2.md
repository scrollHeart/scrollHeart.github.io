---
title: 你不知道的JavaScript（下）
date: 2019/4/29 8:27:10 PM
tags: [JavaScript]
categories: [读书笔记,JavaScript]

---

### 第二部分 ES6及更新版本

**Object.is()与比较操作符===、==**

* == (或者 !=) 操作在需要的情况下自动进行了类型转换。

* === (或 !==)操作不会执行任何转换。

* ===在比较值和类型时，可以说比==更快

* Object.is()类似于===，但在三等号判等的基础上特别处理了 NaN 、-0 和 +0 ，保证 -0 和 +0 不再相同，但 Object.is(NaN, NaN) 会返回 true
* ===不能区分两个不同的数字 -0 和 +0，还会把两个 NaN 看成是不相等的

下面这些情况Object.is()会认为两个值是相同的

``` javascript
两个值都是 undefined
两个值都是 null
两个值都是 true 或者都是 false
两个值是由相同个数的字符按照相同的顺序组成的字符串
两个值指向同一个对象
两个值都是数字并且
都是正零 +0
都是负零 -0
都是 NaN
都是除零和 NaN 外的其它同一个数字
```

``` javascript
// 两个特例，=== 也没法判断的情况
Object.is(0, -0);            // false
Object.is(NaN, 0/0);         // true
```

### 第2章 语法

#### 2.1 块作用域声明

JavaScript中变量作用域的基本单元一直是function，若要创建个块作用域，除了普通的函数声明外，就是立即调用函数表达式（IIFE）

``` javascript
var a = 2;
(function IIFE(){
	var a = 3;
    console.log(a);   // 3
})();
console.log(a);       // 2
```

**let 声明**

创建绑定到任意块的声明，其被称为块作用域（block scoping）。

一对{..}可以创建一个作用域，不像使用var声明的变量是归属于包含函数（即全局，若在最顶层的话）作用域

使用一个专门的{..}块的模式是创建块作用域变量最好方法，

把let声明放在块的最前面，建议只用一个let

``` javascript
var a = 2;
{
    let a = 3, b, c;
    console.log(a);   // 3
}
console.log(a);       // 2
```

**let + for**

for循环头部的let i不只为for 循环本身声明了一个i,为循环的每一次迭代重新声明了一个新的i，loop迭代内部创建的闭包封闭是每次迭代中的变量

``` javascript
var funcs = [];
for(let i = 0; i < 5; i++){
    funcs.push(function(){
        console.log(i)
    })
}
funcs[3]();       // 3
```

**const声明**

const,用于创建常量，设定了初始值之后只读的变量

如果需要一个值为undefined的常量，就要声明const a = undefined

``` javascript
{
    const a = 2;
    console.log(a);     // 2
    a = 3;              // TypeError!
}
```

复杂值，比如对象或者数组，其内容仍然是可以修改的

``` javascript
const a = [1, 2, 3];
a.push(4);
console.log(a);       // [1, 2, 3, 4]
a = 42;               // TypeError!
```

const 用在for, for..in 以及for..of循环的变量声明中，若想要重新赋值就会抛出错误，比如for循环中常用的i++

**是否使用const**

只对你有意表明不会改变的变量使用const

**块作用域函数**

``` javascript
{
    foo();        //可以这么做
    function foo(){
        //..
    }
}
foo();            // ReferenceError
```

#### 2.2 spread/rest

…通常称为spread或rest（展开或收集）运算符，取决于它在哪/如何使用

``` javascript
function foo(x,y,z){
    console.log(x, y, z);
}
foo(...[1,2,3]);           //1 2 3
```

当…用在数组之前时（实际上是任何iterable），会把这个变量“展开”为各个独立的值

… 为我们提供了可以替代apply(..)方法的一个简单的语法形式，…可以在上下文用来展开/扩展一个值

``` javascript
foo.apply(null, [1,2,3])     // 1 2 3
```

``` javascript
var a = [2, 3, 4];
var b = [1, ...a, 5];
console.log(b);         // [1,2,3,4,5]
```

...另外一种常见用法基本上可被看作反向的行为，…把一系列值收集到一起成为一个数组

``` javascript
function foo(x, y, ...z){
	console.log(x, y, z);
}
foo(1, 2, 3, 4, 5);         // 1 2 [3, 4, 5]
```

…z把剩下的参数收集到一起组成一个名为z的数组

若没有命名参数，…会收集所有参数

``` javascript
function foo(...args){
    console.log(args)
}
foo(1, 2, 3, 4, 5);         // [1, 2, 3, 4, 5]
```

…args通常称为“rest参数”，在收集其余的参数

#### 2.3 默认参数值

``` javascript
function foo(x, y){
    x = (0 in arguments) ? x : 11;
    y = (1 in arguments) ? y : 31;
    console.log(x + y);
}
foo(5);               // 36
foo(5, undefined);    // NaN
```

JavaScript设计原则：undefined意味着缺失，undefined和缺失是无法区别的，对于函数参数如此

ES6新增的有用的语法来改进为缺失参数赋默认值的流程

``` javascript
function foo(x = 11, y = 31){
	console.log(x + y);
}
foo();              // 42
foo(5, 6);          // 11
foo(0, 42);         // 42
foo(5);             // 36
foo(5, undefined);  // 36
foo(5, null);       // 5
foo(undefined, 6);  // 17
foo(null, 6);       // 6
```

**默认值表达式**

``` javascript
function bar(val){
    console.log('bar called!');
    return y + val;
}
function foo(x = y + 3, z = bar(x)){
	console.log(x, z);
}
var y = 5;
foo();                 // 'bar called'
                       // 8 13
foo(10);               // 'bar called'
				       // 10 15
y = 6;
foo(undefined, 10);    // 9 10
```

在参数的值省略或者为undefined的时候，注意函数声明中形式参数是在它们自己的作用域中，在默认值表达式中的标识符引用首先匹配到形式参数作用域，然后才搜索外层作用域

#### 2.4 解构

把这个功能看作是一个结构化赋值方法，专用于**数组解构**和**对象解构**

``` javascript
function foo(){
    return [1, 2, 3];
}
function bar(){
    return {
        x: 4,
        y: 5,
        z: 6
    }
}
var [a, b, c] = foo();
var {x: x, y: y, z: z} = bar();
console.log(a, b, c);      // 1 2 3
console.log(x, y, z);      // 4 5 6
```



消除了前面代码中对临时变量tmp的需求，赋值符=左侧的[a, b, c]被当作某种“模式”，用来把右侧数组值解构赋值到独立的变量中

**2.4.1 对象属性赋值模式**

``` javascript
var {x, y, z} = bar();
console.log(x, y, z);     // 4 5 6
```

实际上使用缩写语法的时候是略去了x: 部分，更长的形式支持把属性赋给非同名变量

``` javascript
var {x: bam, y: baz, z: bap} = bar();
console.log(bam, baz, bap);           // 4 5 6
console.log(x, y, z);                 // ReferenceError
```

{a: X, b: Y}  X是要赋给它的值，target: source，property-alias: value 

``` javascript
var {x: bam, y: baz, z: bap} = bar();
```

这里的语法模式是souce: target(value: variable-alias)`x: bam`表示x属性是源值，而bam是要赋值的目标变量,

对象解构赋值是source —>target

**2.4.2 不只是声明**

解构是一个通用的赋值操作，不只是声明

``` javascript
var a, b, c, x, y, z;
[a, b, c] = foo();
({x, y, z} = bar());
console.log(a, b, c);       // 1 2 3
console.log(x, y, z);       // 4 5 6
```

> 若省略了var/let/const声明符，就必须把整个赋值表达式用（）括起来

赋值表达式（a, y等）并不是必须是变量标识符，任何合法的赋值表达式都可以

可以在解构中使用计算出的属性表达式

``` javascript
var which = "x", o = {};
({[which]: o[which]} = bar());
console.log(o.x);            // 4
```

可用一般的赋值来创建对象映射/变换

```  javascript
var o1 = {a: 1, b: 2, c: 3}, o2 = {};
({a: o2.x, b: o2.y, c: o2.z} = o1);
console.log(o2.x, o2.y, o2.z);     // 1 2 3
```

可把一个对象映射为一个数组

``` javascript
var o1 = {a: 1, b: 2, c: 3}, a2 = [];
({a: a2[0], b: a2[1], c: a2[2]} = o1);
console.log(a2);          // [1, 2, 3]
```

还可以把一个数组重排序到另一个

``` javascript
var a1 = [1, 2, 3], a2 = [];
[a2[2], a2[0], a2[1]] = a1;
console.log(a2);             // [2, 3, 1]
```

可不用临时变量解决“交换两个变量”这个经典问题

``` javascript
var x = 10, y = 20;
[y, x] = [x, y];
console.log(x, y);          // 20 10
```

> 不应该在赋值中混入声明，这也是前面在[a2[0], ..] = ..解构赋值中把var a2 = []分离出来，语句var [a2[0], ..] = ..是不合法的，a2[0]不是有效声明标识符，也不会隐式创建一个var a2 = []声明

**2.4.3 重复赋值**

对象解构形式允许多次列出同一个源属性

``` javascript
var {a: X, a: Y} = {a: 1};
X;       // 1
Y;       // 1
```

**解构赋值表达式**

对象或者数组解构的赋值表达式的完成值是所有右侧对象/数组的值

``` javascript
var o = {a: 1, b: 2, c: 3}, a, b, c, p;
p = {a, b, c} = o;
console.log(a, b, c);          // 1 2 3
p === o;                       // true
```

通过持有对象/数组的值作为完成值，把解构赋值表达式组成链

``` javascript
var o = {a: 1, b: 2, c: 3}, p = [4, 5, 6], a, b, c, x, y, z;
({a} = {b, c} = o);
[x, y] = [z] = p;
console.log(a, b, c);        // 1 2 3
console.log(x, y, z);        // 4 5 4
```

#### 2.5 太多，太少，刚刚好

不需要把存在的所有值都用来赋值

``` javascript
function foo(){
    return [1, 2, 3];
}
function bar(){
    return {
        x: 4,
        y: 5,
        z: 6
    }
}
var [, b] = foo();
var {x, z} = bar();
console.log(b, x, z);       // 2 4 6
```

多余的值会被赋为undefined

``` javascript
var [,,c,d] = foo();
var {w, z} = bar();
console.log(c, z);      // 3 6
console.log(d, w);      // undefined undefined
```

**默认值赋值**

默认函数参数值类似的=语法，解构的两种形式都可以提供一个用来赋值的默认值

``` javascript
var [a = 3, b = 6, c = 9, d = 12] = foo();
var {x = 5, y = 10, z = 15, w = 20} = bar();
console.log(a, b, c, d);        // 1 2 3 12
console.log(x, y, z, w);        // 4 5 6 20
```

**解构参数**

若实参/形参配对是一个赋值，也是可以解构

``` javascript
function foo([x, y]){
    console.log(x, y);
}
foo([1, 2]);      // 1 2
foo([1]);         // 1 undefined
foo([]);          // undefined undefined

function foo({x, y}){
    console.log(x, y);
}
foo({y: 1, x: 2});     // 2 1
foo({y: 42});          // undefined 42
foo({});               // undefined undefined
```

* 1.解构默认值 + 参数默认值

``` javascript
function f6({x = 10} = {}, {y} = {y: 10}){
    console.log(x, y);
}
f6();        // 10 10
f6({}, {});  // 10 undefined   
```

* 2.嵌套默认： 解构并重组

考虑在一个嵌套对象结构内的一组默认值，

``` javascript
var defaults = {
    options: {
        remove: true,
        enable: false,
        instance: {}
    },
    log: {
        warn: true,
        error: true
    }
};

```

现在想要把所有空槽的位置用默认值设定

```javascript
var config = {
    options: {
        remove: false,
        instance: null
    }
}
```

解决方法：用一个{}把这块包起来成为一个块作用域

``` javascript
// 把defaults合并进config
{
    //(带默认值赋值的)解构
    let {
        options: {
            remove = defaults.options.remove,
            enable = defaults.options.enable,
            instance = defaults.options.instance
        } = {},
        log: {
            warn = defaults.log.warn,
            error = defaults.log.error
		} = {}
    } = config;
    
    // 重组
    config = {
        options: {remove, enable, instance},
        log: {warn, error}
    }
}
```

#### 2.6 对象字面量扩展

**简洁属性**

``` javascript
var x = 2, y = 3, 
    o = {
        x: x,
        y: y
    }
// 可以把x: x简写为x
var x = 2, y  = 3,
    o = {
        x,
        y
    }
```

**简洁方法**

``` javascript
var o = {
    x: function(){
        //..
    },
    y: function(){
        //..
	}
}
// ES6可以这样写
var o = {
    x() {
        //..
    },
    y() {
        //..
	}
}
```

**计算属性名**

一个或多个属性名来自于某个表达式，无法用对象字面量表达

``` javascript
var prefix = "user_";
var o = {
    baz: function(..){..}
};
o[prefix + "foo"] = function(..){..};
o[prefix + "bar"] = function(..){..};
```

ES6对对象字面定义新增了一个语法，用来支持指定一个计算的表达式

``` javascript
var prefix = "user_";
var o = {
  baz: function(..){..},
  [prefix + "foo"]: function(..){..},
  [prefix + "bar"]: function(..){..}
}
```

**设定[[Prototype]]**

在声明对象字面量的时候设定这个对象的[[Prototype]]是有用的

``` javascript
var o1 = {
  //..
};
var o2 = {
  _proto_: o1,
  //..
}
// o2[[Prototype]]连接到了o1
```

已经存在的对象设定[[Prototype]]，可使用ES6工具Object.setPrototypeOf(..)

``` javascript
var o1 = {
  //..
};
var o2 = {
  //..
};
Object.setPrototypeOf(o2, o1);
```

**super对象**

由于JavaScript的原型类而非类对象的本质，super对于普通对象的简洁方法也一样有效，特性也基本相同

``` javascript
var o1 = {
  foo(){
    console.log("o1:foo");
  }
};
var o2 = {
  foo(){
    super.foo();
    console.log("o2:foo");
  }
};
Object.setPrototypeOf(o2, o1);
o2.foo();               // o1:foo
                        // o2:foo
```

先利用`Object.setPrototypeOf()将`o2`的原型加到`o1`上，然后才能够使用`super`调用 `o1`上的`foo()`

> super只允许在简洁方法中出现，不允许在普通函数表达式属性中出现，也只允许以super.XXX的形式(用于属性/方法访问)出现，不能以super()的形式出现

#### 2.7 模板字面量

ES6引入了一个新的字符串字面量，使用`作为界定符

``` javascript
// 前ES6方式
var name = "Kyle";
var greeting = "Hello" + name + "!";
console.log(greeting);            // "Hello Kyle!"
console.log(typeof greeting)      // "string"

// 新ES6方式
var name = "Kyle";
var greeting = `Hello ${name}!`;
console.log(greeting);            // "Hello Kyle!"
console.log(typeof greeting);     // "string"
```

在这组字符外用``来包围。被解释为一个字符串字面量，任何${..}形式的表达式会被立即在线解析求值，这种形式的解析求值形式是**插入**

**插入表达式**

在插入字符串字面量的${..}内可出现任何合法的表达式，包括函数调用、在线函数表达式调用、甚至其他插入字符串字面量

``` javascript
function upper(s){
	return s.toUpperCase();
}
var who = "reader";
var text = `A very ${upper("warm")} welcome to all of you ${upper(`${who}s`)}!`;
console.log(text);
// A very WARM welcome to all of you READERS!
```

**表达式作用域**

``` javascript
function foo(str){
  var name = "foo";
  console.log(str);
}
function bar(){
  var name = "bar";
  foo(`Hello from ${name}!`);
}
var name = "global";
bar();                // "Hello from bar!"
```

插入字符串字面量在它出现的词法作用域内，没有任何形式的动态作用域

**标签模板字面量**

``` javascript
function foo(strings, ...values){
  console.log(strings);
  console.log(values);
}
var desc = "awesome";
foo`Everything is ${desc}!`;
// ["Everything is ", "!"]
// ["awesome"]
```

第一个参数，名为strings，是一个由所有普通字符串组成的数组。…gather/rest运算符把其余所有参数值收集到名为values的数组中，收集到values数组的参数是已经求值的在字符串字面值中插入表达式的结果

**原始(raw)字符串**

标签函数接收第一个名为strings的参数，这是一个数组。.raw属性访问这些原始字符串值

``` javascript
function showraw(strings, ...values){
  console.log(strings);
  console.log(strings.raw);
}
showraw`Hello\nWorld`;
//["Hello
// World"]
// ["Hello\nWorld"]
```

ES6提供一个内建函数可作字符串字面量标签：**String.raw(..)**

#### 2.8 箭头函数

``` javascript
function foo(x, y){
  return x + y;
}
// 对比
var foo = (x, y) => x + y;
```

箭头函数是(x, y) => x + y这一部分，这个函数引用被赋给变量foo

若只有一个表达式，并且省略了包围的{..}的话，则意味着表达式前面有一个隐含return

``` javascript
var f1 = () => 12;
var f2 = x => x * 2;
var f3 = (x, y) =>{
  var z = x * 2 + y;
  y++;
  x *= 3;
  return (x + y + z)/2;
}
```

箭头函数总是函数表达式，并不存在箭头函数声明

**不只是更短的语法，而是this**

在箭头函数内部，this绑定不是动态的，而是词法的

``` javascript
var controller = {
  makeRequest: function(..){
    var self = this;
    bth.addEventListener('click', function(){
      //..
      self.makeRequest(..);
		}, false);
  }
}
```

用=>箭头函数作为回调

``` javascript
var controller = {
  makeRequest: function(..){
    btn.addEventListener('click', () =>{
      //..
      this.makeRequest(..);
    }, false);
  }
}
```

更详细的=>适用时机的规则

* 若一个简短单句在线函数表达式，其中唯一的语句是return某个计算出的值，且这个函数内部没有this引用，且没有自身引用(递归、事件绑定/解绑定)，且不会要求函数执行这些
* 有一个内层函数表达式，依赖于在包含它的函数中调用var self = this hack 或者 .bind(this)来确保适当的this绑定
* 内层函数表达式依赖于封装函数中某种像var args = Array.prototype.slice.call(arguments)来保证arguments的词法复制

#### 2.9 for..of循环

for..of循环的值必须是一个iterable，或者说它必须是可以转换/封箱到一个iterable对象的值。iterable是一个能够产生迭代器供循环使用的对象

``` javascript
var a = ['a', 'b', 'c', 'd', 'e'];
for(var idx in a){
  console.log(idx);
}
// 0 1 2 3 4
for(var val of a){
  console.log(val);
}
// 'a' 'b' 'c' 'd' 'e'
```

**for .. in在数组a的键/索引上循环，而for..of在a的值上循环**

for..of循环向iterable请求一个迭代器(通过内建的Symbol.iterator)，反复调用这个迭代器把它产生的值赋给循环迭代变量

JS中默认为(或提供)iterable的标准内键值包括：

* Arrays
* Strings
* Generators
* Collections/TypedArrays

for..of循环可以通过break、contine、return提前终止，并抛出异常

#### 2.10 正则表达式

**Unicode标识**

JS字符串通常被解释为16位字符序列，这些字符对应**基本多语言平面**

用来正确处理大于`\uFFFF`的 Unicode 字符。也就是说，会正确处理四个字节的 UTF-16 编码

``` javascript
var s = '𠮷';

/^.$/.test(s) // false
/^.$/u.test(s) // true
```

**定点标识**

新增标签模式是**y**，称为"定点(sticky)模式"。定点指在正则表达式的起点有一个虚拟的锚点，只从**lastIndex**属性指定的位置开始匹配

``` javascript
var re1 = /foo/, str = "++foo++";
re1.lastIndex;      // 0
re1.test(str);      // true
re1.lastIndex;      // 0 没有更新

re1.lastIndex = 4;
re1.test(str);      // true 被忽略的lastIndex
re1.lastIndex;      // 4 没有更新
```

可见

* test()并不关心lastIndex的值，总从输入字符串的起始处开始执行匹配
* 没有起始锚点^，对"foo"的搜索从整个字符串向前寻找匹配
* test(..)不更新lastIndex

试验一下定点模式正则表达式

``` javascript
var re2 = /foo/y,    // <-- 注意定点标识y
    str = "++foo++";
re2.lastIndex;       // 0
re2.test(str);       // false 0处没有找到"foo"
re2.lastIndex;       // 0
re2.lastIndex = 2;  
re2.test(str);       // true
re2.lastIndex;       // 5 -- 更新到前次匹配之后位置

re2.test(str);       // false
re2.lastIndex;       // 0--前次匹配失败后重置
```

关于定点模式的新观察结果

* test(..)使用lastIndex作为str中精确而唯一的位置寻找匹配，不会向前移动去寻找匹配，匹配位于lastIndex位置上
* 若匹配成功，test(..)会更新lastIndex指向紧跟匹配内容之后的那个字符，若匹配失败，test(..)会把lastIndex重置回0

**定点定位**

``` javascript
var re = /f../y,
    str = "foo       far       fad";
str.match(re);    // ["foo"]

re.lastIndex = 10;
str.match(re);    // ["far"]

re.lastIndex = 20;
str.match(re);    //["fad"]
```

**定点还是全局**

用g全局匹配标识和exec(..)方法来模拟这种相对于lastIndex的匹配

``` javascript
var re = /o+./g,      // <--- 注意g!
    str = "foot book more";
re.exec(str);         // ["oot"]
re.lastIndex;         // 4

re.exec(str);         // ["ook"]
re.lastIndex;         // 9

re.exec(str);         // ["or"]
re.lastIndex;         // 13

re.exec(str);         // null--没有更多匹配
re.lastIndex;         // 0--现在从头开始
```

> lastIndex: 开始下一次查找的索引位置
>
> 循环使用同一个正则表达式的exec()方法，靠的就是lastIndex，因为带全局标志的正则表达式每次匹配后都会更新lastIndex的值作为下次查找匹配的起点

**锚定**

^锚点是一个总是指向输入起始处的锚点，和lastIndex完全没有任何关系

> 底线：y加上^再加上lastIndex > 0是一个不兼容的组合，总是会导致匹配失败

``` javascript
var re = /^foo/y,
    str = "foo";
re.test(str);      // true
re.test(str);      // false

re.lastIndex;      // 0--失败后重置

re.lastIndex = 1;
re.test(str);      // false--由于定位而失败
re.lastIndex;      // 0--失败后重置
```

**flags**

若想要通过检查一个正则表达式对象来判断它应用了哪些标识

``` javascript
var re = /foo/ig;
re.flags;            // "gi"
```

#### 2.13 符号

symbol没有字面量形式，创建过程：

``` javascript
var sym = Symbol('some optional description');
typeof sym;          // 'symbol'
```

* 不能也不应该对Symbol(..)使用new，它不是构造器，也不会创建一个对象
* 传给Symbol(..)的参数是可选的，若传入，应该是为这个symbol的用途给出用户友好描述的字符串
* typeof的输出是一个新的值("symbol")

symbol也不是Symbol的实例

Symbol.for(..)在全局符号注册表中搜索，来查看是否有描述文字相同的符号已经存在，若有的话就返回它，若没有，会新建一个并将其返回

**作为对象属性的符号**

``` javascript
var o = {
  foo: 42,
  [Symbol('bar')]: 'hello world',
  baz: true
};
Object.getOwnPropertyNames(o);          // ['foo', 'baz']

// 要取得对象的符号属性
Object.getOwnPropertySymbols(o);        // [Symbol(bar)]
```



### 第3章 代码组织

#### 3.1 迭代器

迭代器(iterator)是一个结构化的模式，为迭代器引入一个隐式的标准化接口

**接口**

ES6详细解释了Iterator接口，包括如下要求

Iterator [required]必选参数

​		next() {method}: 取得下一个IteratorResult

有些迭代器还扩展支持两个可选成员：

Iterator [optional]可选参数

​		return() {method}: 停止迭代器并返回IteratorResult

​		throw() {method}: 报错并返回IteratorResult

IteratorResult 接口指定如下：

IteratorResult

​		value {property}: 当前迭代值或最终返回值(若undefined为可选)

​		done {property}: 布尔值，指示完成状态

还有一个Iterable接口，用来表述必需能够提供生成器的对象：

Iterable

​		@@iterator(){method}: 产生一个Iterator

@@iterator是一个特殊的内置符号，表示可为这个对象产生迭代器的方法

IteratorResult接口指定了从任何迭代器操作返回的值必须是下面这种形式的对象：

``` javascript
{value: .., done: true / false}
```

**next() 迭代**

iterable,产生的迭代器可以消耗其自身值：

``` javascript
var arr = [1, 2, 3];
var it = arr[Symbol.iterator]();

it.next();     // {value: 1, done: false}
it.next();     // {value: 2, done: false}
it.next();     // {value: 3, done: false}

it.next();     // {value: undefined, done: true}
```

**可选的return(..)和throw(..)**

多数内置迭代器都没有实现可选的迭代器接口——return(..)和throw(..)，return(..)被定义为向迭代器发送一个信号，表明消费者代码已经完毕，不会再从其中提取任何值

**迭代器循环**

可通过为迭代器提供一个Symbol.iterator方法简单返回这个迭代器本身使它成为iterable

``` javascript
var it = {
  // 使迭代器it成为iterable
  [Symbol.iterator]() {return this;},
  next(){..},
  ..
};
it[Symbol.iterator]() === it;    // true
```

可用for..of循环消耗这个it 迭代器

``` javascript
for(var v of it){
  console.log(v);
}
```

**迭代器消耗**

spread运算符…完全消耗了迭代器

``` javascript
var a = [1, 2, 3, 4, 5];
function foo(x, y, z, w, p){
  console.log(x + y + z + w + p);
}
foo(...a);         // 15
```

…也可以把一个迭代器展开到一个数组中

``` javascript
var b = [0, ...a, 6];
b;                     // [0, 1, 2, 3, 4, 5, 6]
```

#### 3.2 生成器

**语法**

``` javascript
function *foo(){
  //..
}
```

从功能上来说，*的位置无所谓，同样的声明可写作：

``` javascript
function *foo(){..}
function* foo(){..}
function * foo(){..}
function*foo(){..}
```

**1.运行生成器**

尽管生成器用*声明，但执行起来还和普通函数一样

foo();

``` javascript
function *foo(x, y){
  //..
}
foo(5, 10);
```

**2.yield**

用来标示暂停点：yield

``` javascript
function *foo(){
  var x = 10;
  var y = 20;
  
  yield;
  
  var z = x + y;
}
```

**3.yield***

`*`使用一个function声明成了`function*`生成器声明，`*`使得yield成为了yield *，称为**yield委托**

yield *..需要一个iterable

#### 3.3 模块

唯一最重要的代码组织模式是模块

**旧方法**

传统的模块模式基于一个带有内部变量和函数的外层函数，以及一个被返回的"public API"，这个"public API"带有对内部数据和功能拥有闭包的方法

``` javascript
function Hello(name){
	function greeting(){
    console.log("Hello" + name + "!");
  }
  // public API
  return {
    greeting: greeting
  };
}

var me = Hello("Kyle");
me.greeting();            // Hello Kyle!
```

**前进**

对于ES6来说，不需要依赖于封装函数和闭包提供模块支持

* ES6使用基于文件的模块，也就是说一个文件一个模块，需要分别加载
* ES6模块的API是静态的，需要在模块的公开API中静态定义所有最高层导出
* ES6模块是单例，模块只有一个实例，其中维护了它的状态
* 模块的公开API中暴露的属性和方法并不仅仅是普通的值或引用的赋值

ES6模块将会为代码组织提供完整支持，包括封装、控制公开API以及引用依赖导入

**CommonJS**

模块transpiler/ 转换工具是必不可少的

**新方法**

ES6模块的两个主要新关键字是import和export

**1.导出API成员**

``` javascript
export function foo(){
  //..
}
export var awesome = 42;
var bar = [1, 2, 3];
export {bar};

// 同样导出的另外一种表达形式：
function foo(){
  //..
}
var awesome = 42;
var bar = [1, 2, 3];

export{foo, awesome, bar};
```

这些称为**命名导出**，因为导出变量/函数等的名称绑定

没有用export标示的一切都在模块作用域内部保持私有

在命名导出时还可以"重命名"(也即别名)一个模块成员：

``` javascript
function foo(){..}
export {foo as bar};
```

导入这个模块的时候，只有成员名称bar可以导入；foo还是隐藏在模块内部

在模块定义内部多次使用export，ES6倾向于一个模块使用一个export，称之为**默认导出**

``` javascript
function foo(..){
  //..
}
export default foo;
//以及
function foo(..){
  //..
}
export {foo as default};
```

导出的是此时到函数表达式值的绑定，而不是标识符foo，export default..接受的是一个表达式

可以有一个单独的默认导出，同时又有其他命名导出

``` javascript
export default function foo(){..}

export function bar(){..}
export function baz(){..}

// 也可以这样写
function foo(){..}
function bar(){..}
function baz(){..}

export { foo as default, bar, baz, ..};
```

ES6模块机制的设计意图是不鼓励模块大量导出

**2.导入API成员**

使用import语句导入模块

若想导入一个模块API的某个特定命名成员到顶层作用域，可使用下面语法：

``` javascript
import { foo, bar, baz } from "foo";
```

字符串"foo"称为**模块指定符**

import语句有一种语法变体可以支持这种模块导入，称为**命名空间导入**

``` javascript
import  * as foo from "foo";
foo.bar();
foo.x;
foo.baz();
```

> `* as..`语句需要一个*通配符, 不能用 import {bar, x} as foo from "foo"这样的语句只导入API的一部分但仍然绑定到foo命名空间

所有导入的绑定都是不可变和/或只读的，导入之后所有这些试图赋值的动作都会抛出TypeErrors:

``` javascript
import foofn, * as hello from "world";
foofn = 42;          // (运行时) TypeError!
hello.default = 42;  // (运行时) TypeError!
hello.bar = 42;      // (运行时) TypeError!
hello.baz = 42;      // (运行时) TypeError!
```

作为import结果的声明是"提升的"

``` javascript
foo();
import { foo } from "foo";
```

import最基本的形式是这样：

``` javascript
import "foo";
```

这种形式没有实际导入任何一个这个模块的绑定到你的作用域，它加载，编译，并求值"foo"模块

#### 3.4 类

class 表示一个块，其内容定义了一个函数原型的成员

``` javascript
class Foo{
  constructor(a,b) {
    this.x = a;
    this.y = b;
  }
  gimmeXY() {
    return this.x * this.y;
  }
}
```

需要注意以下几点

* class Foo表明创建一个名为Foo的函数
* constructor(..)指定Foo(..)函数的签名以及函数体内容
* 类方法使用对象字面量可用的同样的"简洁方法"语法，类方法是不可枚举的，而对象方法默认是可枚举的
* 和对象字面量不一样，在class定义体内部不用逗号分隔成员

尽管class Foo看起来很像function Foo()

* class Foo 的Foo(..)调用必须通过new来实现
* function Foo是"提升的"，而class Foo并不是，extends..语句指定了一个不能被"提升"的表达式，在实例化一个class之前必须先声明它
* 全局作用域中的class Foo创建了这个作用域的一个词法标识符Foo，并没有创建一个同名的全局对象属性

class只是创建了一个同名的构造器函数，现有的instanceof运算符对ES6类仍然可以工作

可以把类看作一个宏(macro)，自动产生一个prototype对象，若使用extends，也连接起了[[Prototype]]关系

ES6 class本身并不是一个真正的实体，而是一个包裹其他向函数和属性这样的具体实体并把它们组合到一起的元概念

**extends 和 super**

extends 提供了一个语法糖，用来在两个函数原型之间建立[[Protoype]]委托链接

``` javascript
class Bar extends Foo{
  constructor(a,b,c){
    super(a,b);
    this.z = c;
  }
  gimmeXYZ(){
    return super.gimmeXY() * this.z;
  }
}
var b = new Bar(5, 15, 25);
b.x;            // 5
b.y;            // 15
b.z;            // 25
b.gimmeXYZ();   // 1875
```

super自动指向"父构造器"，super会指向"父对象"，可访问其属性/方法，super.gimmeXY()

Bar extends Foo是Bar.prototype的[[Prototype]]连接到Foo.prototype

**1.super**

* super的行为根据其所处的位置不同而有所不同，构造器中不能这样使用super；super.prototype不能工作

* super(..)意味着调用new Foo(..)，但实际上并不是指向Foo自身的一个可用引用。

* 在非构造器方法内部引用Foo(..)函数，super.constructor指向函数Foo(..)，只能通过new调用，new super.constructor是合法的

* super是静态绑定到这个特定的类层次上，不能重载

2.**子类构造器**

子类构造器自动调用父类的构造器并传递所有参数

``` javascript
constructor(...args){
  super(...args);
}
```

``` javascript
class Foo{
  constructor(){this.a = 1;}
}
class Bar extends Foo{
  constructor(){
    super();
    this.b = 2;
  }
}
```

**new.target**

new.target的形式引入了一个新概念，称为**元属性**

new.target总是指向new实际上直接调用的构造器，即使构造器是在父类中且通过子类构造器用super(..)委托调用

``` javascript
class Foo{
  constructor() {
    console.log("Foo", new.target.name);
  }
}
class Bar extends Foo{
	constructor(){
    super();
    console.log("Bar:", new.target.name);
  }
  baz() {
    console.log("baz:", new.target);
  }
}
var a = new Foo();
//Foo: Foo
var b = new Bar();
// Foo: Bar  <--遵循new调用点
// Bar: Bar
b.baz();
// baz: undefined
```

> 若new.target是undefined，可知道这个函数不是通过new调用的

**static**

类（class）通过 **static** 关键字定义静态方法，可实现Bar()[[Prototype]]链接到Foo()，因为这些是直接添加到这个类的函数对象上的

静态方法调用直接在类上进行，不能在类的实例上调用。静态方法通常用于创建实用程序函数。

``` javascript
class Foo{
  static cool(){console.log("cool");}
  wow(){console.log("wow");}
}
class Bar extends Foo{
  static awesome(){
    super.cool();
    console.log('awesome');
  }
  neat(){
    super.wow();
    console.log('neat');
  }
}

Foo.cool();          //'cool'

Bar.cool();          // 'cool'
Bar.awesome();       // 'cool'
                     // 'awesome'
var b = new Bar(); 
b.neat();            // "wow"
      							 // "neat"
b.awesome;           // undefined
b.cool;              // undefined
```

在函数构造器之间的双向/并行链上

**Symbol.species Getter构造器**

``` javascript
class MyCoolArray extends Array{
  //强制species为父构造器
  static get [Symbol.species](){return Array;}
}
var a = new MyCoolArray(1, 2, 3),
    b = a.map(function(v){return v * 2});
b instanceof MyCoolArray;      // false
b instanceof Array;            // true
```

父类方法如何使用子类species声明

``` javascript
class Foo{
  // 推迟species为子构造器
  static get [Symbol.species](){return this;}
  spawn(){
    return new this.constructor[Symbol.species]();
  }
}
class Bar extends Foo{
  //强制species为父构造器
  static get [Symbol.species](){return Foo;}
}
var a = new Foo();
var b = a.spawn();
b instanceof Foo;            // true

var x = new Bar();
var y = x.spawn();
y instanceof Bar;            // false
y instanceof Foo;            // true
```

### 第5章 集合

#### 5.1 TypedArray

带类型的数组更多是为了使用类数组语义(索引访问等)结构化访问二进制数据。

名称中的“type(类型)”指看待一组位序列的"视图"，本质上是一个映射

构建这样的集合，称为一个"buffer"，最直接的方法是通过ArrayBuffer(..)构造器来构造：

``` javascript
var buf = new ArrayBuffer(32);
buf.byteLength;                // 32
```

#### 5.2 Map

对象是创建无序键/值对数据结构[也称为映射(map)]的主要机制。对象作为映射的主要缺点是不能使用非字符串值作为键

``` javascript
var m = {};
var x = {id: 1},
    y = {id: 2};
m[x] = "foo";
m[y] = "bar";
m[x];           //"bar"
m[y];           //"bar"

```

x 和 y两个对象字符串化都是"[object object]"，所以m中只设置了一个键

可用ES6，需要使用Map(..)

``` javascript
var m = new Map();
var x = {id: 1},
    y = {id: 2};
m.set(x, "foo");
m.set(y, "bar");

m.get(x);            // "foo"
m.get(y);            // "bar"
```

唯一的缺点是不能使用方括号[ ]语法设置和获取值，可以使用get(..)和set(..)方法完美代替

要从map中删除一个元素，不要使用delete运算符，而是要使用delete()方法：

``` javascript
m.set(x, "foo");
m.set(y, "bar");

m.delete(y);
```

可通过clear()清除整个map的内容，要得到map的长度，可以使用size属性

``` javascript
m.size;         // 1
m.clear();      // 0
m.size;         // 0
```

Map(..)构造器也可以接受一个iterable，这个迭代器必须产生一列数组，每个数组的第一个元素是键，第二个元素是值。这种迭代形式和entries()方法产生的形式是完全一样的

``` javascript
var m2 = new Map(m.entries());
//等价于
var m2 = new Map(m);
```

> entries() 方法返回一个数组的迭代对象，该对象包含数组的键值对 (key/value)，迭代对象中数组的索引值作为 key， 数组元素作为 value。

也可以在Map(..)构造器中手动指定一个项目（entry）列表(键/值数组的数组)：

``` javascript
var x = {id: 1},
    y = {id: 2};

var m = new Map([
  [x, "foo"],
  [y, "bar"]
]);

m.get(x);            // "foo"
m.get(y);            // "bar"
```

**Map值**

从map中得到一列值，可使用values(..)，会返回一个迭代器

``` javascript
var m = new Map();

var x = {id: 1},
    y = {id: 2};

m.set(x, "foo");
m.set(y, "bar");

var vals = [...m.values()];

vals;                       // ["foo", "bar"]
Array.from(m.values());     // ["foo", "bar"]

```

**Map键**

要得到一列键，可以使用keys()，它会返回map中键上的迭代器

``` javascript
var m = new Map();
var x = {id: 1},
    y = {id: 2};

m.set(x, "foo");
m.set(y, "bar");

var keys = [...m.keys()];

keys[0] === x;            // true
keys[1] === y;            // true
```

要确定一个map中是否有给定的键，可以使用has(..)方法：

``` javascript
var m = new Map();
var x = {id: 1},
    y = {id: 2};

m.set(x, "foo");

m.has(x);          // true
m.has(y);          // false
```

#### 5.3 WeakMap

WeakMap是map的变体，二者多数外部行为特性一样，区别在于内部内存分配(特别是其GC)的工作方式

WeakMap(只)接受对象作为键，这些对象是被弱持有的，若对象本身被垃圾回收的话，在WeakMap中的这个项目也会被移除。

``` javascript
var m = new WeakMap();

var x = {id: 1},
    y = {id: 2};

m.set(x, "foo");

m.has(x);        // true
m.has(y);        // false
```

WeakMap没有size属性或clear()方法，也不会暴露任何键、值或项目上的迭代器。

#### 5.4 Set

set是一个值的集合，其中的值唯一(重复会被忽略)

只是add(..)方法代替了set(..)方法，没有get(..)方法。

``` javascript
var s = new Set();

var x = {id: 1},
    y = {id: 2};

s.add(x);
s.add(y);
s.add(x);

s.size;        // 2

s.delete(y);   
s.size;        // 1

s.clear();
s.size;         // 0
```

和Map(..)接受项目(entry)列表(键/值数组的数组)不同，Set(..)接受的是值(value)列表(值的数组)：

``` javascript
var x = {id: 1},
    y = {id: 2};

var s = new Set([x,y]);
```

set不需要get(…)是因为不会从集合中 取一个值，而是使用has(..)测试一个值是否存在

``` javascript
var s = new Set();

var x = {id: 1},
    y = {id: 2};

s.add(x);

s.has(x);       // true
s.has(y);       // false
```

#### 5.5 WeakSet

WeakSet对其值也是弱持有的

``` javascript
var s = new WeakSet();

var x = {id: 1},
    y = {id: 2};

s.add(x);
s.add(y);

x = null;        // x可GC
y = null;        // y可GC
```

WeakSet的值必须是对象，而并不像set一样可以是原生类型值

### 第6章 新增API

#### 6.1 Array

**静态函数Array.of(..)**

Array(..)构造器有一个众所周知的陷阱，"空槽"行为，即传入一个数字参数，构造一个空数组，其length属性为此数字

``` javascript
var a = Array(3);
a.length;          // 3
a[0];              // undefined
```

Array.of(..)取代了Array(..)成为数组的推荐函数形式构造器，并没有这个特殊的单个数字参数问题

``` javascript
var b = Array.of(3);
b.length;          // 1
b[0];              // 3

var c = Array.of(1, 2, 3);  
c.length;          // 3
c;                 // [1, 2, 3]
```

**静态函数Array.from(..)**

"类(似)数组对象"是指一个有length属性，具体说是大于等于0的整数值的对象

普遍的需求是把它们转换为真正的数组

``` javascript
var arrLike = {
  length: 3,
  0: "foo",
  1: "bar"
};
var arr = Array.prototype.slice.call(arrLike);  // ["foo", "bar", empty]
```

常见任务是使用slice(..)来复制产生一个真正的数组：

``` javascript
var arr2 = arr.slice();
```

新的ES6Array.from(..)方法更简洁

``` javascript
var arr = Array.from(arrLike);     // ["foo", "bar", undefined]
var arrCopy = Array.from(arr);
```

**1.避免空槽位**

Array.from(..)永远不会产生空槽位

``` javascript
var a = Array(4);                            // 4个空槽位
var b = Array.apply(null, {length: 4});      // 4个undefined值
```

在Array.from(..)使其简单很多

``` javascript
var c = Array.from({length: 4});             // 4个undefined值
```

**2.映射**

第二个参数是一个映射回调(和一般的Array#map(…)所期望的几乎一样)，这个函数会被调用，来自于源的每个值映射/转换到返回值

``` javascript
var arrLike = {
  length: 4,
  2: "foo"
};

Array.from(arrLike, function mapper(val, idx){
  if(typeof val == 'string'){
    return val.toUpperCase();
  }
  else{
    return idx;
  }
});
// [0, 1, "FOO", 3]
```

> Array.from(..)接收一个可选的第三个参数，若设置了，这个参数为作为第二个参数传入的回调指定this绑定。
>
> 否则，this会是undefined

**创建数组和子类型**

``` javascript
class MyCoolArray extends Array{
  //强制species为父构造器
  static get [Symbol.species]() {return Array;}
}

var x = new MyCoolArray(1, 2, 3);

x.slice(1) instanceof MyCoolArray;       // false
x.slice(1) instanceof Array;             // true
```

@@species设置只用于像slice(..)这样的原型。of(..)和from(..)不会使用它，它们只使用this绑定(由使用的构造器来构造其引用)

``` javascript
MyCoolArray.from(x) instanceof MyCoolArray;        // true
MyCoolArray.of([2, 3]) instanceof MyCoolArray;     // true
```

**原型方法copyWithin(..)**

Array#copyWithin(..)是一个新的修改器方法，copyWithin(..)从一个数组中复制一部分到同一个数组的另一个位置，覆盖这个位置所有原来的值

参数是target(要复制到的索引)、start(开始复制的源索引，包括在内)以及可选的end(复制结束的不包含索引)，若任何一个参数是负数，就被当作是相对于数组结束的相对值

``` javascript
[1, 2, 3, 4, 5].copyWithin(3, 0);              // [1, 2, 3, 1, 2]
[1, 2, 3, 4, 5].copyWithin(3, 0, 1);           // [1, 2, 3, 1, 5]
[1, 2, 3, 4, 5].copyWithin(0, -2);             // [4, 5, 3, 4, 5]
[1, 2, 3, 4, 5].copyWithin(0, -2, -1);         // [4, 2, 3, 4, 5]
```

**原型方法 fill(..)**

Array#fill(..)用指定值完全(或部分)填充已存在的数组

``` javascript
var a = Array(4).fill(undefined);
a;             // [undefined, undefined, undefined, undefined]
```

fill(..)可选地接收参数start 和 end，它们指定了数组要填充的子集位置

``` javascript
var a = [null, null, null, null].fill(42, 1, 3);
a;             // [null, 42, 42, null]
```

**原型方法find(..)**

基本和some(..)工作方式一样，除了一旦回调返回true/真值，会返回实际的数组值

``` javascript
var a = [1, 2, 3, 4, 5];

a.find(function matcher(v){
  return v%2 == 0;
});                          // 2

a.find(function matcher(v){
  return v == 7;             // undefined
})
```

通过自定义matcher(..)函数也可支持比较像对象这样的复杂值：

``` javascript
var points = [
  {x: 10, y: 20},
  {x: 20, y: 30},
  {x: 30, y: 40},
  {x: 40, y: 50},
  {x: 50, y: 60}
]

points.find(function matcher(point){
    return(
       point.x%3 == 0 && point.y %4 == 0
    );
});                                 // {x: 30, y: 40}
```

**原型方法 findIndex(..)**

indexOf(..)无法控制匹配逻辑，总是使用===严格相等。ES6的findIndex(..)可控制匹配逻辑，返回位置索引

``` javascript
points.findIndex(function matcher(point){
  return (
  	point.x % 3 == 0 && point.y % 4 == 0
  )
})      // 2

points.findIndex(function matcher(point){
  return (
  	point.x % 6 == 0 && point.y % 7 == 0
  )
})      // -1

```

不要使用findIndex(..) != -1(这是indexOf(..)的惯用法)

需要自定义匹配的索引值，使用findIndex(..)

**原型方法entries()、values()、keys()**

Array从传统角度来说，可能不会被看作"集合"，但它提供了同样的迭代器方法entries()、values()、keys()是一个集合

``` javascript
var a = [1, 2, 3];

[...a.values()];            // [1, 2, 3]
[...a.keys()];              // [0, 1, 2]
[...a.entries()];           // [[0,1], [1,2],[2,3]]
[...a[Symbol.iterator]()];  // [1, 2, 3]
```

#### 6.2 Object

**静态函数Object.is(..)**

需要严格识别NaN或者-0值，应该选择Object.is(..)，调用底层SameValue算法，和===严格相等比较算法一样

``` javascript
var x = NaN, y = 0, z = -0;

x === x;                   // false
y === z;                   // true

Object.is(x, x);           // true
Object.is(y, z);           // false
```

**静态函数Object.getOwnPropertySymbols(..)**

可直接从对象上去的所有的符号属性

``` javascript
var o = {
  foo: 42,
  [Symbol("bar")]: "hello world",
  baz: true
};

Object.getOwnPropertySymbols(o); [Symbol(bar)]
```

**静态函数Object.setPrototypeOf(..)**

设置对象的[[Prototype]]用于**行为委托**

``` javascript
var o1 = {
  foo(){console.log("foo");}
}
var o2 = {
  //..o2的定义
};

Object.setPrototypeOf(o2, o1);

// 委托给o1.foo()
o2.foo();                  // foo

// 也可以：

var o1 = {
  foo(){console.log("foo");}
};
var o2 = Object.setPrototypeOf({
  // ..o2的定义
}, o1);
// 委托给o1.foo()
o2.foo();                 // foo
```

**静态函数Object.assign(..)**

用于把一个对象的属性复制/混合到另一个对象中的工具，第一个参数是target,其他传入的参数都是源，它们将按照列出的顺序依次被处理，Object.assign(..)返回目标对象

``` javascript
var target = {},
    o1 = {a: 1}, o2 = {b: 2},
    o3 = {c: 3}, o4 = {d: 4};
// 设定只读属性
Object.defineProperty(o3, "e", {
  value: 5,
  enumerable: true,
  writable: false,
  configurable: false
});

// 设定不可枚举属性
Object.defineProperty(o3, "f", {
  value: 6,
  enumerable: false
});

o3[Symbol('g')] = 7;

// 设定不可枚举符号
Object.defineProperty(o3, Symbol('h'), {
	value: 8,
  enumerable: false
});

Object.setPrototypeOf(o3, o4);

Object.assign(target, o1, o2, o3);

target.a;   // 1
target.b;   // 2
target.c;   // 3

Object.getOwnPropertyDescriptor(target, 'e');
// {value: 5, writable: true, enumerable: true, configurable: true}

```

#### 6.4 Number

向已有的全局函数的引用：Number.parseInt(..)和Number.parseFloat(..)

Number.EPSILON：任意两个值之间的最小差：2^-52

**静态函数 Number.isNaN(..)**

``` javascript
var a = NaN, b = "NaN", c = 42;

isNaN(a);                       // true
isNaN(b);                       // true--oops!
isNaN(c);                       // false

Number.isNaN(a);                // true
Number.isNaN(b);                // false -- 修正了
Number.isNaN(c);                // false
```

**静态函数Number.isFinite(..)**

被认为是"非无限的"，isFinite(..)会对参数进行强制类型转换

``` javascript
var a = NaN, b = Infinity, c = 42;

Number.isFinite(a);     // false
Number.isFinite(b);     // false
Number.isFinite(c);     // true

var d = "42";

isFinite(d);            // true
Number.isFinite(d);     // false
```

#### 6.5 字符串

**静态函数String.raw(..)**

提供内置标签函数，与模板字符串字面值一起使用，用于获得不应用任何转义序列的原始字符串

``` javascript
var str = "bc";

String.raw`\ta${str}d\xE9`;
// "\tabcd\xE9",而不是"abcde"
```

**原型函数repeat(..)**

重复字符串

``` javascript
"foo".repeat(3);         // "foofoofoo"
```

**字符串检查函数**

startsWith(..)、endsWith(..) 和 includes(..)

``` javascript
var palindrome = "step on no pets";

palindrome.startsWith('step on');    // true
palindrome.startsWith('on', 5);      // true

palindrome.endsWith('no pets');      // true
palindrome.endsWith('no', 10);       // true

palindrome.includes('on');           // true
palindrome.includes('on', 6);        // false
```

### 第7章 元编程

元编程是指操作目标是程序本身的行为特性的编程


