---
title: 你不知道的JavaScript（中）
date: 2019/4/29 8:13:43 PM
tags: [JavaScript]
categories: [读书笔记,JavaScript]

---

### 第一部分 类型和语法

### 1、值和类型

**检测null 值的类型**：

``` javascript
typeof null === "object"   // true

var a = null;
(!a && typeof a=== "object")  // true
```



JavaScript中的变量是没有类型的，只有值才有。变量可以随时持有任何类型的值。

**undefined 和 undeclared**:

变量在未持有值的时候为undefined, typeof 返回“undefined”

还没有作用域中声明过的变量，是undeclared

``` javascript
var a;
a; // undefined
b; //ReferenceError: b is not defined
```

**字符串**：

JavaScript 中字符串是不可变的，而数组是可变的。不同点在于字符串反转，数组有一个字符串没有的可变更成员函数reverse()

undefined类型只有一个值，即undefined,null类型只有一个值，即null

**特殊数值**：

* null :空值
  * 特殊关键字，不是标识符，不能将其当作变量来使用和赋值

* undefined：没有值
  * 标识符，可被当作变量来使用和赋值
  * void运算符返回undefined

* NaN：它和自身不相等，是唯一一个非自反（自反： 即 x === x 不成立），而NaN != NaN 为true
  * isNaN(…)来判断一个值是否为NaN

* +Infinity
* -Infinity
* -0： +0 === -0

**面点：判断两份值是否绝对相等**

* ===: 效率更高，更通用
* Object.is()：主要用来处理特殊的相等比较

**值和引用**：

* 简单值（标量基本类型值）：通过值复制的方式来赋值/传递，包括null, undefined, 字符串，数字，布尔和symbol
* 复合值：通过引用复制来赋值/传递，js中的引用只能指向值，不能指向别的变量/引用

**Symbol**(…): 命名对象属性不容易导致重名

### 2、强制类型转换

**JSON.stringify(…)** 

* 在对象中遇到undefined、function 和 symbol时会自动将其忽略，在数组中会返回null

* 传递一个可选参数，数组或函数，来指定对象序列化过程中哪些属性应该被处理，哪些应该被排除，和toJSON()很像

  ``` javascript
  var a = {
      b: 42,
      c: '42',
      d: [1,2,3]
  };
  JSON.stringify(a, ['b', 'c']) // "{"b": 42, "c": "42"}"
  JSON.stringify(a, function(k, v){
      if(k !== 'c') return v;
  })
  // "{"b": 42, "d": [1,2,3]}"
  ```

**解析字符串中的数字和将字符串强制类型转换为数字的区别**

* 解析允许字符串中含有非数字字符，解析按从左到右的顺序，遇到非数字字符就停止
* 转换不允许出现非数字字符，会失败并返回NaN

* parseInt()针对的是字符串值

**显示转换为布尔值**：!!，第二个!会将结果反转回原值，建议使用Boolean(a) 和 !!a来进行显示强制类型转换

``` javascript
if(!!a){

}
if(Boolean(a)){
    
}
```

**安全运用隐式强制类型转换**

* 如果两边的值有true或者false，千万不要使用==

* 如果两边的值中有[]、“”或者0，尽量不要使用==

---

 ### 第二部分     异步和性能 

程序中现在运行的部分和将来运行的部分之间的关系是异步编程的核心

#### 1.1 分块的程序

只有一个是现在执行，其余的在将来执行，最常见的块单位是函数

将来执行的部分并不一定在现在运行的部分执行完之后就立即执行

只要把一段代码包装成一个函数，并指定它在响应某个事件（定时器、鼠标点击、Ajax响应等）时执行

#### 1.2 事件循环

提供了一种机制来处理程序中多个块的执行，执行每块时调用JavaScript引擎，这种机制被称为**事件循环**

JavaScript 引擎在需要的时候，在给定的任意时刻执行程序中的单个代码块，不是独立运行的，在宿主环境中，web浏览器，按需执行JavaScript任意代码片段的环境

> js程序发出请求，从服务器获取数据，在回调函数中设置响应代码，引擎会通知宿主环境，然后浏览器会设置侦听网络响应，拿到数据后，把回调函数插入事件循环，实现回调的调度执行



setTimeout(…)并没有把回调函数挂在事件循环队列中，而是设定一个定时器，当定时器到时后，环境会把回调函数放在事件循环中

**setTimeout(…)的精度不高**：事件循环中项目多时，回调会等待，没有抢占式的方式支持将其排到队首

#### 1.3 并行线程

异步是关于现在和将来的时间间隙，并行是关于能够同时发生的事情

并行计算最常见的工具是进程和线程，多个线程能共享单个进程的内存

事件循环把自身的工作分为一个个任务并顺序执行，不允许对共享内存的并行访问和修改，并行和顺序执行可共存

**交互**：并发的“进程”需要互相交流，通过作用域或DOM间接交互

竞态：一种并发交互条件，也叫门闩，特征为“只有第一名取胜”

一旦有事件需要运行，事件循环就会运行，直到队列清空，事件循环的每一轮称为一个tick，用户交互，IO和定时器会向事件队列中加入事件

并发指两个或多个事件链随时发展交替执行，从更高层次看，像同时在运行（尽管在任一时刻只处理一个事件）



### 第2章 回调

理解处理所有事件（异步函数调用）的单线程事件循环队列，事件的运行顺序可以有多种可能

函数作为回调使用的，是事件循环“回头调用”到程序中的目标，队列处理到这个项目的时候运行它

> 大脑在两个或更多任务之间快速连续地来回切换，同时处理每个任务的微小片段

**回调地狱**：多个函数嵌套在一起构成的链，其中每个函数代表异步序列（任务，“进程”）中的一个步骤，这种代码称为回调地狱，或 毁灭金字塔

**回调地狱的真正问题**：

**控制转移**是指我们由于把控制权交给一个我们不能信任的第三方而产生的对我们的程序失去控制的现象

**尝试挽救回调**

* 分离回调设计：ES6 Promise API使用此回调设计
* error-first回调模式：回调的第一个参数保留用作错误对象，node.jsAPI都采用此风格



### 第3章 Promise

通过回调表达程序异步和管理并发的两个主要缺陷：缺乏顺序性和可信任性

> 控制反转（Inversion of Control，缩写为IoC），是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。最常见的方式有依赖注入（Dependency Injection，简称DI）和“依赖查找”。依赖注入即组件之间的依赖关系由容器在运行期决定，形象的来说，即由容器动态的将某种依赖关系注入到组件之中

控制反转原则：

* 高层模块不应该依赖低层模块。两个都应该依赖抽象

* 抽象不应该依赖具体实现

* 面向接口编程，而非面向实现编程

#### 3.1 什么是Promise

**未来值**的一个重要特性：它可能成功，也可能失败

**Promise值**

Promise封装了依赖时间的状态——等待底层值的完成或拒绝，Promise本身与时间无关

一旦Promise决议，它就永远保持在这个状态，成为不变值

Promise是一种封装和组合未来值的易于复用的机制

**完成事件**

Promise的决议：一种在异步任务中作为两个或更多步骤的流程控制机制，时序上的this-then-that

Promise"事件"：像是一个函数在说“我这有一个事件监听器，当我完成或者失败的时候会被通知到。”，只需要监听then事件，然后通过知道哪个回调函数被调用就可以知道是成功还是失败

``` javascript
function someAsyncThing(){
    var p = new Promise(function(resolve,reject){
        //at some later time,call 'resolve()' or 'reject()'
    }) ;
    return p ;
}
var p = someAsyncThing() ;
p.then(
    function(){
        //success happened    
    },
    function(){
        //failure happened
    }
) ;
```



**标准的promises机制有以下这些保证**

1. 如果promise被resolve，它要不是success就是failure，不可能同时存在。
2. 一旦promise被resolve，它就再也不会被resolve(不会出现重复调用)。
3. 如果promise返回了成功的信息，那么你绑定在成功事件上的回调会得到这个消息。
4. 如果发生了错误，promise会收到一个带有错误信息的错误通知。
5. 无论promise最后的结果是什么(success或者failure)，他就不会改变了，你总是可以获得这个消息只要你不销毁promise

#### 3.2 具有then方法的鸭子类型

**如何确定某个值是不是真正的Promise?**

由于Promise值可能是从其他浏览器窗口接收到的，或框架实现的，所以用 instanceof **不足以作为检查方法**

识别Promise是定义某种称为thenable的东西，将其定义为任何具有then(…)方法的对象和函数，任何这样的值是Promise一致的thenable

根据一个值的形态（具有哪些属性）对这个值的类型做出一些假定，这种类型检查称为**“鸭子类型”**（如果它看起来像鸭子，叫起来像鸭子，那它一定是鸭子）

对thenable值的鸭子类型检查类似于：

``` javascript
if(p !== null && (typeof p === "object" || typeof p === "function") && typeof p.then === "function"){
    //假定这是一个thenable
}else{
    // 不是thenable
}
```



#### 3.3 Promise 信任问题

**promise的特性是专门用来为这些问题提供一个有效的可复用答案**

* 调用回调过早

  对一个Promise调用then(…)时，即使Promise已经决议，提供给then(…)的回调总会被异步调用

* 调用回调过晚

  一个promise决议后，这个Promise所有的通过then(…)注册的回调会在下一个异步时机点上依次被立即调用

  两个独立Promise上链接的回调的相对顺序无法可靠预测

* 回调未调用

  没有任何东西（甚至JavaScript错误）能阻止Promise向你通知它的决议，对一个Promise注册了一个完成回调和一个拒绝回调，Promise在决议时总是会调用其中的一个

  若本身永远不被决议，Promise提供一种竞态的高级抽象机制

  ``` javascript
  function timeoutPromise(delay) {
    return new Promise(function(resolve, reject){
        setTimeout(function(){
            reject("Timeout!")
        }, delay)
    })
  }
  //设置foo()超时
  Promise.race([
      foo(),
      timeoutPromise(3000)
  ]).then(function(){
  	//foo(...)及时完成
  }, function(err){
      // 或者foo()被拒绝，只是没能按时完成
      //查看err来了解是哪种情况
  })
  ```

  保证一个foo()有一个输出信号，防止其永久挂住程序

* 调用回调次数过少或过多

  Promise定义方式使得它只能被决议一次，任何通过then(…)注册的回调只会被调用一次

* 未能传递所需的环境和参数

  没有用任何值显式协议，这个值就是undefined,使用多个参数调用resovle(…)或reject(…),第一个参数之后的所有参数都会被默默忽略

* 吞掉可能出现的错误和异常

  拒绝一个Promise并给出理由（出错消息），这个值会被传给拒绝回调，Promise把Javascript异常也变成异步行为，极大降低了竞态条件出现的可能

  ``` javascript
  var p = new Promise(function(resolve, reject){
  	resolve(42);
  })
  p.then(
      function fulfilled(msg){
          foo.bar();
          console.log(msg); // 永远不会到达这里
      },
      function rejected(err){
          // 永远也不会到达这里
      }
  )
  ```

  p.then(…)本身返回另外一个Promise，此promise会因TypeError异常而被拒绝

**是可信任的Promise吗**

* Promise.resolve(…)传递一个非Promise、非thenable的立即值，会得到一个用这个值填充的promise

* Promise.resolve(…)传递一个真正的Promise，只会返回同一个Promise

* 向Promise.resolve(…)传递一个非Promise的thenable值，前者会试图展开这个值，展开过程会持续到提取出一个具体的非类Promise的最终值

  ``` javascript
  var p = {
      then: function(cb, errcb){
          cb(42);
          errcb('evil laugh')
      }
  }
  Promise.resolve(p).then(
      function fulfilled(val){
          console.log(val); // 42
      },
      function rejected(err){
          // 永远不会到这里
      })
  ```

#### 3.4 链式流

链式流：把多个Promise连接到一起以表示一系列异步步骤

链式流实现的关键在于Promise固有行为特性：

* 每次对Promise调用then(…)，都会创建并返回一个新的Promise，可以将其链接起来
* 从then(…)调用的完成回调（第一个参数）返回的值是什么，都会被自动设置为被链接Promise的完成

``` javascript
var p = Promise.resolve(21);
p.then(function(v){
    console.log(v);  //21
    return v * 2;    //用值42完成链接的promise
}).then(function(v){ //这里是链接的promise
    console.log(v);  // 42
})
```

使Promise真正能够在每一步有异步能力的关键是，回忆一下当传递给Promise.resolve(…)的是一个Promise或thenable而不是最终值时的运作方式

Promise链不仅是一个表达多步异步序列的流程控制，还是一个从一个步骤到下一个步骤传递消息的消息通道

``` javascript
//步骤1 
request('http://some.url.1/')
//步骤2
.then(function(response1){
    foo.bar(); // undefined,出错
    //永远不会到达这里
    return request('http://some.url.2/?v=' + response1);
})
//步骤3
.then(
	function fulfilled(response2){
        //永远达不到这里
	},
    //捕捉错误的拒绝处理函数
    function rejected(err){
        console.log(err);
        // 来自foo.bar()的错误TypeError
        return 42;
	}
)
//步骤4
.then(function(msg){
	console.log(msg);     //42
})
```

第3步的拒绝处理函数会捕捉到这个错误，拒绝处理函数的返回值

``` javascript
var p = new Promise(function(resolve, reject){
    reject('Oops');
})
var p2 = p.then(
	function fulfilled(){
        //永远不会到达这里
	}
    //假定的拒绝处理函数，如果省略或者传入任何非函数值
    // function(err){
    // 		throw err;
	// }
)
```

promise的then(…)，并且只传入一个完成处理函数，一个默认拒绝处理函数会顶替上来，默认拒绝处理函数只是把错误重新抛出，使得p2用同样的错误理由拒绝，使得错误可以继续沿着Promise链传播下去，直到遇到显式定义的拒绝处理函数

> then(null, function(err){…})这个模式——只处理拒绝（若有的话），又把完成值传递下去，有个缩写API:catch(function(err){…})

链式流程控制可行的Promise固有特性

* 调用Promise的then(…)会自动创建一个新的Promise从调用返回
* 在完成或拒绝处理函数内部，若返回一个值或抛出一个异常，新返回的Promise就相应地决议
* 若完成或拒绝处理函数返回一个Promise，将会被展开，不管它的决议值是什么，都会成为当前then(…)返回的链接Promise的决议值

**术语： 决议、完成以及拒绝**

决议（resolve）、完成（fulfill）和拒绝（reject）

先来研究一下构造器Promise(...)

``` javascript
var p = new Promise(function(X,Y){
    // X()用于完成
    // Y()用于拒绝
})
```

第一个通常用于标识Promise已经完成，第二个总是用于标识Promise被拒绝

Promise.resolve(…)是一个精确好名字，它实际上的结果可能是完成或拒绝，Promise(…)构造器的第一个回调参数的恰当称谓是resolve(…)

对then(…)的第一个参数来说，总是处理完成的情况。ES6规范将这两个回调命名为onFulfilled(…)和onRejected(…)

``` javascript
function fulfilled(msg){
    console.log(msg)
}
function rejected(err){
    console.error(err)
}
p.then(
    fulfilled,
    rejected
)
```

#### 3.5 错误处理

错误处理最自然的形式是同步的try…catch结构，只用于同步，无法用于异步代码模式

Promise错误处理是一个“绝望的陷阱”设计，默认情况下，它假定你想要Promise状态吞掉所有的错误，若忘记查看这个状态，这个错误会默默地在暗处凋零死掉

为避免丢失被忽略和抛弃的Promise错误，可采用

``` javascript
var p = Promise.resolve(42);
p.then(
	function fulfilled(msg){
		//数字没有string函数，会抛出错误
        console.log(msg.toLowerCase())
    }
)
.catch(handleErrors);
```

**处理未捕获的情况**

done(...)拒绝处理函数内部的任何异常都会被作为一个全局未处理错误抛出

``` javascript
var p = Promise.resolve(42);
p.then(
	function fulfilled(msg){
        //数字没有string函数，会抛出错误
        console.log(msg.toLowerCase())
	}
)
.done(null, handleErrors);  // 如果handleErrors(...)引发了自身的异常，会被全局抛出到这里
```

浏览器可以追踪Promise对象，若在它被垃圾回收时候其中有拒绝，浏览器能确保是一个真正的未捕获错误，可以确定应该将其报告到开发者终端

**成功的坑**

如果想要一个拒绝的Promise在查看之前的某个时间段内保持被拒绝状态，可调用defer(),这个函数优先级高于该Promise的自动错误报告

``` javascript
function foo(cb){
	setTimeout(function(){
        try{
            var x = baz.bar();
            cb(null, x); // 成功
		}
        catch(err){
            cb(err)
		}
	}, 100)
}
var p = Promise.reject('Oop').defer();
foo(42)
    .then(
    	function fulfilled(){
            return p;
        },
    	function rejected(err){
            //处理foo(...)错误
		}
})
```

#### 3.6 Promise 模式

门（gate）是一种机制要等待两个或更多并行/并发的任务都完成才能继续，完成顺序不重要，但必须要完成，门才能打开并让流程控制继续，在PromiseAPI中，这种模式称为all([…])

Promise.all([..])需要一个参数，是一个数组，由Promise实例组成。从Promise.all([..])调用返回的promise会收到一个完成消息，由所有传入promise的完成消息组成的数组

Promise.all([..])的数组中的值可以是Promise、thenable，甚至是立即值

确保要等待的是一个真正的Promise,立即值会被规范化为这个值构建的Promise

如果数组是空的，主Promise会立即完成

Promise.all([..])返回的主promise在且仅在所有成员promise都完成后才完成

若这些promise中有任何一个被拒绝的话，主Promise.all([..])promise会立即被拒绝，并丢弃来自其他所有promise的全部结果

为每个promise关联一个拒绝/错误处理函数，特别是从Promise.all([..])返回的那一个

``` javascript
// request(..)是一个Promise-aware Ajax工具
var p1 = request('http://some.url.1/');
var p2 = request('http://some.url.2/');

Promise.all([p1, p2])
    .then(function(msgs){
    // 这里p1和p2完成并把它们的消息传入
    return request(
    	"http://some.url.3/?v=" + msgs.join(',')
    )
})
.then(function(msg){
    console.log(msg)
})
```

**Promise.race([..])**

有时只响应“第一个跨过终点线的Promise”，而抛弃其他Promise，这种模式称为门闩，在Promise中称为竞态

Promise.race([..])接受单个数组参数，传入一个空数组，主race([..])Promise永远不会决议

一旦有任何一个Promise决议为完成，Promise.race([..])就会完成；一旦有任何一个Promise决议为拒绝，它就会拒绝

``` javascript
//request(..)是一个支持Promise的Ajax工具
var p1 = request('http://some.url.1/')
var p2 = request('http://some.url.2/')
Promise.race([p1, p2])
.then(function(msg){
    return request('http://some.url.3/?v=' + msg)    
})
.then(function(msg){
    console.log(msg)
})
```

只有一个Promise能够取胜，完成值是单个消息

**Promise.prototype.finally() ** ( ES2018 引入标准的)

不管`promise`最后的状态，在执行完`then`或`catch`指定的回调函数以后，都会执行`finally`方法指定的回调函数

**all([..]) 和 race([..])的变体**

none([..])：所有的Promise都要被拒绝，即拒绝转化为完成值

any([..])：这个模式与all([..])类似，但会忽略拒绝，只需要完成一个而不是全部

first([..])：这个模式类似于与any([..])的竞争，只要第一个Promise完成，它会忽略后续的任何拒绝和完成

last([..])：这个模式类似于first([..])，但却是只有最后一个完成胜出

**并发迭代**

异步的map(..)工具，接收一个数组的值（可以是Promise或其他任何值），外加在每个值上运行一个函数（任务）作为参数。map(..)本身返回一个promise，其完成值是一个数组，该数组保存在任务执行之后的异步完成值

``` javascript
if(!Promise.map){
    Promise.map = function(vals, cb){
        // 一个等待所有map的promise的新promise
        return Promise.all(
        	//注： 一般数组map(..)把值数组转换为promise数组
            vals.map(function(val){
                //用val异步map之后决议的新promise替换val
                return new Promise(function(resolve){
                    cb(val, resolve)
                })
            })
        )
    }
}
```

下面展示如何在一组Promise上使用map(..)

``` javascript
var p1 = Promise.resolve(21);
var p2 = Promise.resolve(42);
var p3 = Promise.reject('Oops');
//把列表中的值加倍，即使在Promise中
Promise.map([p1,p2,p3], function(pr, done){
	//保证这一条本身是一个Promise
    Promise.resolve(pr)
    .then(
    	// 提取值作为v
        function(v){
            //map完成的V到新值
            done(v * 2);
        },
        // 或者map到promise拒绝消息
        done
    )
})
.then(function(vals){
	console.log(vals)   // [42, 84, "Oops"]
})
```

#### 3.7 Promise API概述

**new Promise(..)构造器**

构造器Promise(..)必须和new 一起使用，并且必须提供一个函数回调。这个回调是同步的或立即调用的。

此函数接受两个函数回调，用以支持promise的决议，把这两个函数称为resolve(..)和reject(..)

``` javascript
var p = new Promise(function(resolve, reject){
	// resolve(...)用于决议/完成这个promise
    // reject(...)用于拒绝这个promise
})
```

reject(…)是拒绝这个promise; resolve(..)可能完成promise，也可能拒绝

**Promise.resolve(..)和Promise.reject(..)**

* **Promise.reject(..)**: 创建一个已被拒绝的Promise的快捷方式是使用Promise.reject(..)

  ``` javascript
  var p1 = new Promise(function(resolve, reject){
      reject('Oop');
  })
  var p2 = Promise.reject('Oop')
  ```

* **Promise.resolve(..)**方法的参数分成四种情况

  * **参数是一个 Promise 实例**

    Promise.resolve将不做任何修改、原封不动地返回这个实例

  * **参数是一个thenable对象**

    `Promise.resolve`方法会将这个对象转为 Promise 对象，然后就立即执行`thenable`对象的`then`方法

    ```javascript
    let thenable = {
      then: function(resolve, reject) {
        resolve(42);
      }
    };
    
    let p1 = Promise.resolve(thenable);
    p1.then(function(value) {
      console.log(value);  // 42
    });
    ```

    `thenable`对象的`then`方法执行后，对象`p1`的状态就变为`resolved`

  * **参数不是具有then方法的对象，或根本就不是对象**

    ```javascript
    const p = Promise.resolve('Hello');
    
    p.then(function (s){
      console.log(s)
    });
    // Hello
    ```

    返回 Promise 实例的状态从一生成就是`resolved`，所以回调函数会立即执行。`Promise.resolve`方法的参数，会同时传给回调函数

  * **不带有任何参数**

    `Promise.resolve()`方法允许调用时不带参数，直接返回一个`resolved`状态的 Promise 对象

    如果希望得到一个 Promise 对象，比较方便的方法就是直接调用`Promise.resolve()`方法

    ```javascript
    const p = Promise.resolve();
    
    p.then(function () {
      // ...
    });
    ```

    > 立即`resolve()`的 Promise 对象，是在本轮“事件循环”（event loop）的结束时执行，而不是在下一轮“事件循环”的开始时

    ``` javascript
    setTimeout(function () {
      console.log('three');
    }, 0); //在下一轮“事件循环”开始时执行
    
    Promise.resolve().then(function () { //在本轮“事件循环”结束时执行
      console.log('two');
    });
    
    console.log('one');   //立即执行
    
    // one
    // two
    // three
    ```

**then(..)和catch(..)**

每个Promise实例都有then(…)和catch(…)方法，通过这两个方法可以为这个Promise注册完成和拒绝处理函数

Promise决议后，立即会调用这两个处理函数之一，总是异步调用

* then(…)：
  * 接受一个或两个参数，第一个用于完成回调，第二个用于拒绝回调
  * 若两者中任何一个被省略或作为非函数值传入的话，会替换为相应的默认回调，默认完成回调只是把消息传递下去，而默认拒绝回调只是重新抛出其接收到的出错原因

* catch(…)只接受一个拒绝回调作为参数，并自动替换默认完成回调，等价于then(null, ..)

  ``` javascript
  p.then(fulfilled)
  
  p.then(fulfilled, rejected)
  
  p.catch(rejected)   // 或者p.then(null, rejected)
  
  ```

**Promise.all([..]) 和 Promise.race([..])**

* Promise.all([..])
  * 只有传入的所有promise都完成，返回promise才能完成，完成会得到一个数组，包含传入的所有promise的完成值
  * 若有任何promise被拒绝，返回的主promise立即会被拒绝，只会得到第一个拒绝promise的拒绝理由值

* Promise.race([..])
  * 只有第一个决议的promise(完成或拒绝)取胜，并且其决议结果成为返回promise决议，这种模式称为门闩

> 当心！若向Promise.all([..])传入空数组，它会立即完成，但Promise.race([..])会挂住，且永远不会决议

``` javascript
var p1 = Promise.resolve(42);
var p2 = Promise.resolve('Hello World');
var p3 = Promise.reject('Oops');
Promise.race([p1, p2, p3])
.then(function(msg){
    console.log(msg);   // 42
})

Promise.all([p1, p2, p3])
.catch(function(err){
    console.error(err);  //'Oops'
})

Promise.all([p1, p2])
.then(function(msgs){
    console.log(msgs);    // [42, 'Hello World']
})
```

#### 3.8 Promise 局限性

**顺序错误处理**

Promise的设计局限性（具体来说，就是它们链接的方式），Promise链中的错误很容易被无意中默默忽略掉

若构建了一个没有错误处理函数的Promise链，链中任何地方的任何错误都会在链中一直传播下去，直到被查看，只要有一个指向链中最后一个promise的引用就够了，可以在那里注册拒绝处理函数，能够得到所有传播过来的错误的通知

``` javascript
// foo(..), STEP2(..)以及STEP3(..)都是支持promise的工具
var p = foo(42)
.then(STEP2)
.then(STEP3);
p.catch(handleErrors)
```

**单一值**

Promise只能有一个完成值或一个拒绝理由

**单决议**

Promise只能被决议一次（完成或拒绝）

**惯性**

需要一个支持Promise而不是基于回调的Ajax工具，可称为request(..)

``` javascript
if(!Promise.wrap){
    Promise.wrap = function(fn){
        return function(){
            var args = [].slice.call(arguments);
            return new Promise(function(resolve, reject){
                fn.apply(
                	null,
                    args.concat(function(err, v){
                        if(err){
                            reject(err);
                        }
                        else{
                            resolve(v);
                        }
					})
                )
            })
		}
    }
}

//使用方式
var request = Promise.wrap(ajax);
request('http://some.url.1/')
.then(..)
```

Promise.wrap(..)产出的是一个将产生Promise的函数，产生Promise的函数可以看作是一个Promise工厂，将其命名为“promisory”("Promise" + "factory")

Promise.wrap(ajax)产生了一个ajax(..)promisory,称之为request(..)

**无法取消的Promise**

一旦创建了一个Promise并为其注册了完成和/或拒绝处理函数，若出现某种情况使得这个任务悬而未决的话，你也没有办法从外部停止它的进程

单独的一个Promise并不是一个真正的流程控制机制，这正是取消所涉及的层次（流程控制）。

**Promise性能**

Promise使所有一切都成为异步的了，即有一些立即（同步）完成的步骤仍然会延迟到任务的下一步。Promise任务序列可能比完全通过回调连接的同样的任务序列运行得稍慢一点，

> Promise只用回调的代码而备受困扰的控制反转问题，把回调的安排转交给一个位于我们和其他工具之间的可信任的中介机制

### 第4章 生成器

之前用回调表达异步控制流程的两个关键缺陷：

* 基于回调的异步不符合大脑对任务步骤的规划方式
* 由于控制反转，回调并不是可信任或可组合的

看似同步的异步流程控制表达风格，使这种风格成为可能的“魔法”是ES6生成器

#### 4.1 打破完整运行

实现合作式并发的ES6代码

``` javascript
var x = 1;
function *foo(){
    x++;
    yield; // 暂停
    console.log("x:", x);
}
function bar(){
    x++;
}
```

> 很多JavaScript文档中的生成器声明格式都是`function* foo(){..}`，这里使用`function *foo(){..}`，唯一的区别是`*`位置风格不同，后者在使用`*foo()`来引用生成器的时候比较一致

``` javascript
//构造一个迭代器it来控制这个生成器
var it = foo();
//这里启动foo()
it.next();
x;             // 2
bar();
x;             // 3
it.next();     // x: 3
```

以下是运行过程：

（1）it = foo()运算并没有执行生成器`*foo()`，只是构造了一个迭代器（iterator）,迭代器会控制它的执行

（2）第一个it.next()启动了生成器`*foo()`，并运行了`*foo()`第一行的x++

（3）`*foo()`在yield语句处暂停，在这一点上第一个it.next()调用结束，此时`*foo()`仍在运行且活跃，但处于暂停状态

（4）查看x的值，此时为2

（5）调用bar()，通过x++再次递增x

（6）再次查看x的值，此时为3

（7）最后的it.next()调用从暂停恢复了生成器`*foo()`的执行，并运行console.log(..)，当前x的值为3

**生成器是一类特殊的函数，可以一次或多次启动和停止，并不一定非得要完成**



**4.1.1 输入和输出**

``` javascript
function *foo(x, y){
    return x * y;
}
var it = foo(6, 7);
var res = it.next(); //指示生成器*foo(..)从当前位置开始继续运行，停在下一个yield处或直到生成器结束
res.value;     // 42
```

next(..)调用的结果是一个对象，有一个value属性，持有从`*foo()`返回的值，yield会导致生成器在执行过程中发送出一个值，类似return

**1. 迭代消息传递**

通过yield 和next(..)实现内建消息输入输出能力

yield和next(..)调用有一个不匹配，一般来说，需要的next(..)调用要比yield语句多一个，为什么呢？

第一个next(..)总是启动一个生成器，并运行到第一个yield处，第二个next(..)调用完成第一个被暂停的yield表达式，第三个next(..)调用完成第二个yield，以此类推

**2. 两个问题的故事**

* 第一个yield基本上提出了一个问题，“这里我应该插入什么值？”

谁来回答这个问题呢？

第一个next()已经运行，使得生成器启动并运行到此处，它无法回答这个问题；

由第二个next(..)调用回答第一个yield提出的这个问题

> 消息是双向传递的——yield..作为一个表达式可以发出消息响应next(..)调用，next(..)也可以向暂停的yield表达式发送值，yield..和next(..)组合起来，在生成器的执行过程中构成了双向消息传递系统

**启动生成器时一定要用不带参数的next()**

因为只有暂停的yield才能接受这样一个通过next(..)传递的值，而在生成器的起始处我们调用第一个next()时，还没有暂停的yield来接受这样一个值

* 生成器`*foo(..)`要给我的下一个值是什么？

谁来回答这个问题呢？

第一个yield "hello" 表达式

最后一个it.next(7)调用再次提出了这样的问题：生成器将要产生的下一个值是什么，再没有yield语句来回答这个问题，由return语句回答这个问题

若生成器中没有return——在生成器和在普通函数中一样，return当然不是必需的——总有一个假定的/隐式的return(return undefined);

**4.1.2 多个迭代器**

通过一个迭代器控制生成器的时候，是在控制声明的生成器函数本身。

每次构建一个迭代器，实际上就隐式构建了生成器的一个实例，通过这个迭代器来控制的是这个生成器的实例

#### 4.2 生成器产生值

**生产者与迭代器**

JS迭代器的接口，是每次想要从生产者得到下一个值的时候调用next()

``` javascript
var something = (function(){
    var nextVal;
    return {
        //for..of循环需要
        [Symbol.iterator]: function(){return this;},
        //标准迭代器接口方法
        next: function(){
            if(nextVal === undefined){
                nextVal = 1;
            }
            else{
                nextVal = (3 * nextVal) + 6;
            }
            return {done: false, value: nextVal}
        }
    }
})();
something.next().value; //1
something.next().value; //9
something.next().value; //33
something.next().value; //105
```

next()调用返回一个对象，这个对象有两个属性： done是一个boolean值，标识迭代器的完成状态；value中放置迭代值

for ..of循环，在每次迭代中自动调用next(),不会向next()传入任何值，且会在接收到done: true之后自动停止

``` javascript
for(var v of something){
    console.log(v);
    //不要死循环
    if(v > 500){
        break;
    }
}
// 1 9 33 105 321 969
```

可以通过Object.keys(..)返回一个array，类似于for(var k of Object.keys(obj)){..}这样使用

**4.2.2 iterable**

前面例子中something对象叫作迭代器，因为它的接口中有一个next()方法

iterable(可迭代)指一个包含可以在其值上迭代的迭代器的对象

从ES6开始，从一个iterable中提取迭代器的方法是：iterable必须支持一个函数，其名称是专门的ES6符号值Symbol.iterator.调用这个函数时，会返回一个迭代器

for..of循环自动调用它的Symbol.iterator函数来构建一个迭代器

**4.2.3 生成器迭代器**

生成器看作一个值的生成者，通过迭代器接口的next()调用一次提取出一个值

生成器本身不是iterable，当你执行一个生成器，就得到了一个迭代器

``` javascript
function *something(){
    var nextVal;
    while(true){
        if(nextVal == undefined){
            nextVal = 1;
        }
        else{
            nextVal = (3* nextVal) + 6;
        }
        yield nextVal
    }
}
```

生成器会在每个yield处暂停，函数*something()的状态（作用域）会被保持，不需要闭包在调用间保持变量状态

生成器的迭代器也有一个Symbol.iterator函数，这个函数做的是return this，生成器的迭代器是一个iterable

**停止生成器**

for ..of循环的“异常结束”（“提前终止”），通常由break、return或者未捕获异常引起，会向生成器的迭代器发送一个信号使其终止

在外部通过return(..)手工终止生成器的迭代器实例

``` javascript
var it = something();
for(var v of it){
    console.log(v);
    //不要死循环
    if(v > 500){
        console.log(
        	//完成生成器的迭代器
            it.return('Hello World').value
        )
        //这里不需要break
    }
}
// 1 9 33 105 321 969
// cleaning up!
// Hello World
```

#### 4.3 异步迭代生成器

``` javascript
function foo(x, y){
    ajax(
    	"http://some.url.1/?x=" + x + "&y=" + y,
        function(err, data){
            if(err){
                //向*main()抛出一个错误
                it.throw(err);
            }
            else{
                //用收到的data恢复*main()
                it.next(data);
            }
		}
    )
}

function *main() {
    try{
        var text = yield foo(11, 31);
        console.log(text);
    }
    catch(err){
        console.error(err);
    }
}

var it = main();
//这里启动
it.next();
```

并不是在消息传递的意义上使用yield，而只是将其用于流程控制实现暂停/阻塞。它还是会有消息传递，只是生成器恢复运行之后的单向消息传递

在foo(..)内的运行可以完全异步，异步作为实现细节抽象了出去，可以以同步顺序的形式追踪流程控制：

“发出一个Ajax请求，等它完成之后打印出响应结果”

**同步错误处理**

try..catch

yield是如何让赋值语句暂停来等待foo(..)完成，使得响应完成后可以被赋给text，yield暂停也使得生成器能够捕获错误

``` javascript
if(err){
    //向*main()抛出一个错误
    it.throw(err);
}
```

生成器yield暂停的特性意味着我们不仅能够从异步函数调用得到看似同步的返回值，还可以同步捕获来自这些异步函数调用的错误

``` javascript
function *main(){
    var x = yield "Hello World";
    yield x.toLowerCase();   //引发一个异常
}
var it = main();
it.next().value;            // Hello World
try{
    it.next(42);
}
catch(err){
	console.error(err);     // TypeError
}
```

可以捕获通过throw(..)抛入生成器的同一个错误，基本上也是给生成器一个处理它的机会，若没有处理的话，迭代器代码必须处理

``` javascript
function *main(){
    var x = yield "Hello World";
    //永远不会到这里
    console.log(x);
}
var it = main();
it.next();
try{
    // *main()会处理这个错误吗？看看吧！
    it.throw('Oops');
}
catch(err){
    //不行，没有处理
    console.error(err);   //Oops
}
```

在异步代码中实现看似同步的错误处理（通过try..catch)在可读性和合理性方面都是一个巨大进步

#### 4.4 生成器 +Promise

ES6中最完美的世界是生成器（看似同步的异步代码）和Promise（可信任可组合）的结合。

获得Promise和生成器最大效用的最自然的方法是yield出一个Promise，通过这个Promise来控制生成器的迭代器

``` javascript
function foo(x,y){
    return request("http://some.url.1/?x=" + x + "&y=" + y);
}
function *main(){
    try{
        var text  = yield foo(11, 31);
        console.log(text);
    }
    catch(err){
        console.error(err);
    }
}
```

**async 与 await**



``` javascript
function foo(x, y){
    return request('http://some.url.1/?x=' + x + '&y=' + y)
}
async function main(){
    try{
        var text = await foo(11, 31);
        console.log(text);
    }
    catch(err){
        console.error(err);
    }
}
main();
```

async函数，不再yield出Promise，而是用await等待它决议，若await一个Promise， async函数会自动获知要做什么，它会暂停这个函数，直到Promise决议