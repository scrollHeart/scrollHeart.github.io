---
title: JavaScript面向对象精要（上篇）
date: 2019/4/29 8:38:44 PM
tags: [JavaScript]
categories: [读书笔记,JavaScript]

---
面向对象的语言有如下几种特性：

* 封装： 数据可以和操作数据的功能组织在一起，这就是对象的定义
* 聚合： 一个对象能够引用另一个对象
* 继承：一个新创建的对象和另一个对象拥有同样的特性，而无需显示复制其功能
* 多态：一个接口可被多个对象实现

### 第1章 原始类型和引用类型

#### 1. 原始类型

* 照原样保存简单数据

* JavaScript有5种原始类型：boolean,  number,  string, null, undefined

* 每个原始值的变量使用自己的存储空间，一个变量的改变不影响其他变量

  ``` javascript
  var color1 = "red";
  var color2 = color1;
  console.log(color1);    // "red"
  console.log(color2);    // "red"
  
  color1 = "blue";
  console.log(color1);     // "blue"
  console.log(color2);     // "red"
  ```

* 鉴别原始类型  — typeof

  ``` javascript
  console.log(typeof "Nicholas")    // "string"
  console.log(typeof 10)            // "number"
  console.log(typeof true)          // "boolean"
  console.log(typeof undefined)     // "undefined"
  console.log(typeof null)          // "object" null为空对象指针，返回"object"
  ```

  > 判断一个值是否为空类型的最佳方法是直接和null比较

    ``` javascript
  console.log(value === null)      // true or false
    ```

* 非强制转换比较

  * 三等号比较：不会将变量强制转换为另一种类型

  * 双等号比较：将变量强制转换成另一种类型

  ``` javascript
  console.log(undefined == null);   //true
  console.log(undefined === null);  //false
  ```

*  原始方法

  ```javascript
  var count = 10;
  count.toFixed(2);                 // 10.00
  var flag = true;
  flag.toString();                  // true  
  var name = "Nicholas"
  name.toLowerCase();               // nicholas
  ```

  > 原始类型有方法，但它们不是对象，JavaScript使它们看上去像对象，提供语言上的一致性体验



#### 2.引用类型

* 概述：指JavaScript中的对象，最接近类的东西

  * 引用值： 引用类型的实例，是对象的同义词
  * 属性：包含键（始终是字符串）和值，对象是属性的无序列表
  * 方法：属性的值为函数，该属性被称为对象的方法

* 创建对象：

  * new操作符和构造函数

    ``` javascript
    var object = new Object();  // object 是指向内存中实际对象所在位置的指针(或引用)
    ```

  * 对象字面形式: {key: value, key: value}

    > JavaScript引擎背后做的工作和new Object()一样，其他引用类型字面形式也是如此

* 对象引用解除：将对象变量置为null，让垃圾收集器释放内存

* 函数字面形式与正则表达式字面形式

  ``` javascript
  function reflect(value){
      return value;
  }
  // is the same as
  var reflect = new Function('value','return value;');
  ```

  

  ``` javascript
  var numbers = /\d+/g;
  // is the same as
  var numbers = new RegExp('\\d+','g');   // 传入模式参数是字符串，对反斜杠进行转义
  ```

* 访问属性 ：点号和中括号（允许在属性名字上使用特殊字符）

* 鉴别引用类型：instanceof操作符，若对象是构造函数所指定的类型的一个实例，返回true

  > instanceof 操作符可鉴别继承类型，所有对象都是Object的实例，所有引用类型都继承自Object

  ``` javascript
  var items = [];
  var obj = {};
  function reflect(value){
    return value;
  }
  console.log(items instanceof Array);  // true
  console.log(items instanceof Object); // true
  console.log(obj instanceof Object);   // true
  console.log(obj instanceof Array);    // false
  console.log(reflect instanceof Function); // true
  console.log(reflect instanceof Object);   // true
  ```

* 鉴别数组： Array.isArray()，鉴别一个值是否是数组的实例

  ``` javascript
  var items = [];
  console.log(Array.isArray(items))  // true 
  ```

  > ie8或更早版别不支持该方法

* 原始封装类型： String 、Number和Boolean

  * 读取字符串、数字或布尔值，原始封装类型将被自动创建
  * 自动创建的实体仅用于该语句，并随后被销毁

  ``` javascript
  var name = "Nike";
  var firstChar = name.charAt(0);
  console.log(firstChar);
  
  name.last = "zalas";
  console.log(name.last)        // undefined
  //这是在背后发生的事情
  var name = "Nike";
  var temp = new String(name);
  var firstChar = temp.charAt(0);
  temp = null;
  console.log(firstChar);
  
  var temp = new String(name);
  console.log(temp.last);       // undefined
  temp = null;                 
  ```
  * instanceof 检查对应类型返回false(未读取任何东西，未创建临时对象)
  ``` javascript
  var name = 'nike';
  var count = 10;
  var found = false;
  console.log(name instanceof String);    // false
  console.log(count instanceof Number);   // false
  console.log(found instanceof Boolean);  // false
  ```
  * 手动创建原始封装类型会创建出object，typeof无法鉴别出实际保存的数据类型
  ``` javascript
  var name = new String('nike');
  var count = new Number(10);
  var found = new Boolean(false);
  console.log(typeof name);         // "object"
  console.log(typeof count);        // "object"
  console.log(typeof found);        // "object"
  ```

  > 使用String,Number和Boolean对象和使用原始值有一定区别

  ``` javascript
  var found = new Boolean(false);
  if(found){
      console.log("Found")
  }
  ```

### 第二章 函数

 定义：函数是对象，不同于其他对象的决定性特点是存在[[Call]]内部属性，内部属性无法通过代码访问，而是定义了代码执行的行为。

#### 1.函数字面形式： 声明或表达式

> 函数声明会被提升至上下文（所在函数的范围或全局范围）的顶部

``` javascript
//1.函数声明
function add(num1 + num2){
    return num1 + num2;
}
//2.函数表达式：function关键字后不需要加函数名字，通常会被一个变量或属性引用
var add = function(num1, num2){
    return num1 + num2;
}
```

#### 2. 函数就是值

只要可以使用其他引用值的地方，就可以使用函数

``` javascript
var numbers = [1,2,5,8,10,6];
numbers.sort(function(first, second){
    return first - second;
});
```

#### 3.参数

* JS函数独特之处：给函数传递任意数量参数不造成错误，

  因为参数实际被保存在一个称为arguments的类似数组的对象中。

  arguments的length可得知当前参数个数

> 函数期望参数个数保存在函数的length中，实际传入数量不影响期望参数个数

> arguments 对象不是数组的实例，Array.isArray(arguments)永远返回false

``` javascript
function reflect(value){          // value 为命名参数
    console.log(arguments.length) // 2
    return value;
}
console.log(reflect.length)    // 1 期望参数

reflect = function(){
    return arguments[0];
};
console.log(reflect('hi',25));    // "hi"
console.log(reflect.length);      // 0
```

在不知道有多少参数情况下，使用arguments比命名参数有效

``` javascript
function sum(){
	var result = 0,
    	i = 0,
    	len = arguments.length;
    while (i < len){
        result += arguments[i];
        i++;
    }
    return result;
}

console.log(sum(1,2));       // 3
console.log(sum(50));       // 50
console.log(sum(0));       // 0
```

#### 4. 重载

js 不存在函数签名，也不存在重载，可模仿函数重载

``` javascript
function sayMessage(message){
  if(arguments.length === 0){    //  message == undefined更常见
      message = "Default message";
  }
    console.log(message);
}
sayMessage("hello" )
```

#### 5. this 对象

* 解决方法和对象间的紧耦合

* JS所有函数作用域内都有一个this对象代表调用该函数的对象

  ``` javascript
  var person = {
      name: 'nike',
      sayName: function(){
        console.log(person.name); 
          //若person变量名改变，方法中引用名也要改，换成this解决紧耦合
      }
  };
  person.sayName();              
  ```

* 改变this方法

   *  call(): 多个参数，this值和所有参数
   *  apply()：2个参数，this值和一个数组或类似数组的对象
   *  bind()：传给新函数的this的值，其他所有参数代表需要被永久设置在新函数中的命名参数

  ``` javascript
  function sayNameForAll(label){
    console.log(label + ":" + this.name);
  }
  var person1 = {
      name: 'nike'
  };
  var name = 'michael';
  sayNameForAll.call(this, 'global');
  //sayNameForAll.apply(this, ['global']); 其他与call一样
  sayNameForAll.call(person1, 'person1');
  //sayNameForAll.apply(person1, ['person1']);
  ```

  ``` javascript
  function sayNameForAll(label){
    console.log(label + ":" + this.name);
  }
  var person1 = {
      name: 'nike'
  };
  
  var sayNamePerson1 = sayNameForAll.bind(person1);
  sayNamePerson1('person1');            //  "person1:nike"
  
  var sayNamePerson2 = sayNameForAll.bind(person2, 'person2');
  sayNamePerson2();                     //   "person2:nike"
  ```

  

### 第3章 理解对象

#### 1.属性探测

*  in 操作符：检查自有属性和原型属性

* hasOwnProperty()方法：检查自有属性存在

  ``` javascript
  var person1 = {
      name: 'nike',
      sayName: function(){
          console.log(this.name);
      }
  };
  
  console.log("name" in person1);          // true
  // 同样可以检查一个方法是否存在
  console.log("sayName" in person1);       // true
  console.log("toString" in person1);      // true
  
  console.log(person1.hasOwnProperty("name"));      // true
  console.log(person1.hasOwnProperty("toString"));  // false
  ```

#### 2. 删除属性 ——  delete

设置一个属性的值为null不能将它从对象中彻底移除，需要用delete操作符，成功时返回true。可理解为在哈希表中移除了一个键值对。

``` javascript
var person1 = {
    name: 'nike'
};
delete person1.name;                         // true - not output
console.log(person1.name);                   // undefined
```

#### 3. 属性枚举

可枚举属性的内部特征[[Enumerable]]被设置为true，可用for-in循环遍历它们。

  以下两个方法，可获取对象的可枚举属性

* for-in循环

  * 遍历对象的所有可枚举属性并将属性名赋给一个变量

  * 会遍历原型属性

  ``` javascript
    var property;
    for(property in object){
       console.log("name:" + property);
       console.log("value:" + object[property]);
    }
  ```

* Object.keys()

  * 获取可枚举属性名字的数组，再用for循环遍历属性并输出他们的名字和值
  * 只返回自有（实例）属性

  ``` javascript
  var properties = Object.keys(object);
  var i,len;
  for(i = 0,len = properties.length; i < len; i++){
      console.log("name:" + properties[i]);
      console.log("value:" + object[properties[i]]);
  }
  ```

  

propertyIsEnumerable()方法检查一个属性是否为可枚举的

``` javascript
var person1 = {
    name: 'nike'
};
console.log(person1.propertyIsEnumerable("name"));    // true

var properties = Object.keys(person1);
console.log(properties.propertyIsEnumerable("length")); // false
```

> 所有添加的属性默认是可枚举的，对象的大部分原生属性默认是不可枚举的

#### 4.属性类型和特征

属性有两种类型： 数据属性和访问器属性

* 数据属性：包含一个值

* 访问器属性：不包含值，定义了一个当属性读取时调用的函数(getter)和当属性被写入时调用的函数(setter)，两者其一便可

  ``` javascript
  var person1 = {
      _name: 'nike',
      get name() {
          console.log('reading');
          return this._name;
      },
      set name(value) {
          console.log('setting');
          this._name = value;
      }
  };
  console.log(person1.name);            // "nike"       
  person1.name = "greg";
  console.log(person1.name);            // "greg"
  ```

  > 仅定义getter,该属性就变成只读，仅定义setter,该属性就变成只写

* 通用特征：数据和访问器属性都有的特征

  * [[Enumerable]]: 决定是否可以遍历该属性
  * [[Configurable]]：决定该属性是否可配置

* 改变属性特征：Object.defineProperty()，该方法有3个参数

  * 拥有该属性的对象

  * 属性名

  * 包含需要设置的特征的属性描述对象

  ``` javascript
  var person1 = {
    name: 'nike'
  };
  Object.defineProperty(person1, 'name', {
      enumerable: false
  });
  
  console.log('name' in person1);                      // true
  console.log(person1.propertyIsEnumerable('name'));   // false
  ```

* 数据属性特征

  * [[Value]]:  包含属性的值
  * [[Writable]]: 该属性是否可以写入,所有属性默认都是可写的

  ``` javascript
  var person1 = {
      name: 'nike'
  };
  //is the same as
  var person1 = {};
  Object.defineProperty(person1, 'name',{
      value: 'nike',
      enumerable: true,
      configurable: true,
      writable: true
  });
  ```

  > 当用Object.defineProperty()定义新属性时一定要指定所有特征的值，否则布尔型的特征会默认设置为false

* 访问器属性特征: 不需要存储值

  * [[Get]]
  * [[Set]]

  ``` javascript
  var person1 = {
      _name: 'nike',
      get name(){
          console.log('reading');
          return this._name;
      },
      set name(value){
         console.log('setting');
         this._name = value;
      }
  };
  // is the same as
  var person1 = {
      _name: 'nike'
  };
  Object.defineProperty(person1, 'name', {
      get: function() {
          console.log('reading');
          return this._name;
      },
      set: function(value) {
          console.log('setting');
          this._name = value;
      },
      enumerable: true,
      configurable: true
  })
  ```

* 定义多重属性：defineProperties(),接受两个参数：

  * 需要改变的对象
  * 一个包含所有属性信息的对象

  ``` javascript
  var person1 = {};
  Object.defineProperties(person1, {
      _name: {
          value: 'nike',
          enumerable: true,
          configurable: true,
          writable: true
      },
      name: {
          get: function(){
              console.log('reading');
              return this._name;
          },
          set: function(value){
              console.log('setting');
              this._name = value;
          },
          enumerable: true,
          configurable: true
      }
  });
  ```

* 获取属性特征—— Object.getOwnPropertyDescriptor()

  * 只可用于自有属性

  * 接受两个参数：对象和属性名

  * 若属性存在，返回一个属性描述对象，内含4个属性：configurable和

    enumerable，另两个属性根据属性类型决定

  ``` javascript
  var person1 = {
      name: 'nike'
  };
  
  var descriptor = Object.getOwnPropertyDescriptor(person1, 'name');
  console.log(descriptor.enumerable);                 // true
  console.log(descriptor.configurable);               // true
  console.log(descriptor.writable);                   // true
  console.log(descriptor.value);                      // 'nike'
  ```

#### 5. 禁止修改对象

  [[Extensible]] 指导对象行为的内部特征，指明该对象本身是否可以被修改

  设置[[Extensible]]为false,就能禁止新属性的添加 



* 禁止扩展：用Object.preventExtensions()创建一个不可扩展的对象。
  * 该方法接受一个参数，是希望使其不可扩展的对象
  * 一旦使用此方法，永远不能再添加新属性
  * 可用Object.isExtensible()来检查[[Extensible]]的值

``` javascript
var person1 = {
    name: 'nike'
};
console.log(Object.isExtensible(person1));         // true

Object.preventExtensions(person1);
console.log(Object.isExtensible(person1));         // false

person1.sayName = function(){
    console.log(this.name);
};
console.log("sayName" in person1);                 // false
```

* 对象封印: 

  * 一个被封印的对象是不可扩展的且其所有属性都不可配置

  * 如果一个对象被封印，只能读写它的属性

  * 可用Object.seal()来封印对象，调用后[[Extensible]]特性被置为false，

    其所有属性的[[Configurable]]特性被置为false

  * 用Object.isSealed()判断一个对象是否被封印

``` javascript
var person1 = {
    name: 'nike'
};
console.log(Object.isExtensible(person1));           // true
console.log(Object.isSealed(person1));               // false

Object.seal(person1);
console.log(Object.isExtensible(person1));           // false
```

* 对象冻结：被冻结的对象是一个数据属性都为只读的被封印对象
  * 被冻结对象无法解冻
  * Object.freeze()来冻结一个对象
  * 用Object.isFrozen()来判断一个对象是否被冻结

``` javascript
var person1 = {
    name: 'nike'
};
console.log(Object.isExtensible(person1));           // true
console.log(Object.isSealed(person1));               // false
console.log(Object.isFrozen(person1));               // false

Object.freeze(person1);
console.log(Object.isExtensible(person1));           // false
console.log(Object.isSealed(person1));               // true
console.log(Object.isFrozen(person1));               // true
```










