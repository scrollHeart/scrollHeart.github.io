---
title: JavaScript面向对象精要（下篇）
date: 2019/4/29 9:12:05 PM
tags: [JavaScript]
categories: [读书笔记,JavaScript]

---
### 第4章 构造函数和原型对象

#### 1. 构造函数：

* 定义
  * 用new 创建对象时调用的函数
  * 和其他函数一样，构造函数名应该首字母大写

  ``` javascript
  function Person(){
    //do something
  }
  var person1 = new Person();    // person1,person2被认为是一个新								  Person类型实例
  var person2 = new Person;      // 若没传参数的必要，可忽略()
  ```

* new 作用

  * new 操作符会自动创建给定类型的对象并返回它们
  * 用instanceof操作符获取对象类型，返回true

* 对象实例与其构造函数的关系

  * 对象实例的构造函数属性指向创建它的构造函数

  ``` javascript
  console.log(person1 instanceof Person);           // true
  console.log(person1.constructor === Person);      // true
  ```

> 在严格模式下，当不通过new 调用Person构造函数时会出现错误，因为没有为全局对象设置this



#### 2.原型对象：

* 理解
  * 所有函数（除一些内建函数）都有prototype属性
  * prototype指向一个对象，这个对象的用途就是包含所有实例共享的属性和方法（我们把这个对象叫做原型对象）

  ``` javascript
  //鉴别原型属性
  function hasPrototypeProperty(object, name){
      return name in object && !object.hasOwnProperty(name);
  }
  ```

* [[Prototype]] 属性

  * 一个对象实例通过内部属性[[Prototype]]跟踪其原型对象
  * 通过调用Object.getPrototypeOf()方法读取[[Prototype]]属性的值
  * 大部分js引擎支持  _proto_ 的属性,可以直接读写[[Prototype]]属性

  ``` javascript
  var object = {};
  var prototype = Object.getPrototypeOf(object);
  console.log(prototype === Object.prototype);          // true
  //isPrototypeOf()检查某个对象是否是另一个对象的原型对象
  console.log(Object.prototype.isPrototypeOf(object));  // true
  ```

  ``` javascript
  var object = {};
  // toString()来自原型对象
  console.log(object.toString());          // "[object Object]"
  object.toString = function(){
      return "[object Custom]";
  };
  //定义toString()自有属性后，每次调用自有属性，覆盖原型属性
  console.log(object.toString());          // "[object Custom]"
  //自有属性被删除，原型属性才会再次被使用
  delete object.toString;
  console.log(object.toString());          // "[object Object]"
  //无法删除原型属性，无法给一个对象的原型属性赋值
  delete object.toString;
  console.log(object.toString());          // "[object Object]"
  ```

* 在构造函数中使用原型对象

  ``` javascript
  function Person(name) {
      this.name = name;
  }
  Person.prototype.sayName = function() {
      console.log(this.name);
  };
  var person1 = new Person('nide');
  var person2 = new Person('gred');
  
  ```

  * 简洁方式： 直接用一个对象字面形式替换原型对象

  ``` javascript
  function Person(name) {
    this.name = name;
  }
  Person.prototype = {
      sayName: function() {
          console.log(this.name);
      },
      toString: function() {
          return "[Person " + this.name + "]";
      }
  };
  var person1 = new Person('nike');
  
  console.log(person1 instanceof Person);         // true
  console.log(person1.constructor === Person);    // false
  console.log(person1.constructor === Object);    // true
  ```

    > 使用对象字面形式改写原型对象,其constructor属性被置为泛用对象Object,为避免这点，要手动重置constructor属性

  ``` javascript
  function Person(name) {
    this.name = name;
  }
  Person.prototype = {
    constructor: Person,
    sayName: function(){
       console.log(this.name);
    }
  };
  var person1 = new Person('nike');
  console.log(person1.constructor === Person);     // true
  ```

* 内建对象的原型对象

  所有内建对象都有构造函数，都有原型对象去改变

  ``` javascript
  Array.prototype.sum = function() {
      return this.reduce(function(pre,cur) {
          return pre + cur;
      });
  };
  var num = [1,3,5,7];
  var result = num.sum();
  ```

  > 生成环境中不要改变内建对象的原型对象，导致其他开发者无法正常使用



### 第5章 继承

#### 1. 原型对象链和Object.prototype

* 原型对象链 : JavaScript 内建的继承方法，又可称为原型对象继承。

  * 对象继承其原型对象，而原型对象继承它的原型对象

* Object.prototype

  * 所有对象都继承自Object.prototype
  * 以对象字面形式定义的对象，其[[Prototype]]值被设为Object.prototype

* 继承自Object.prototype的方法

  * hasOwnProperty(): 检查是否存在一个给定名字的自有属性
  * propertyIsEnumerable()：检查一个自有属性是否为可枚举属性
  * isPrototypeOf(): 检查一个对象是否是另一个对象的原型对象
  * valueOf(): 返回一个对象的值表达
  * toString(): 返回一个对象的字符串表达




#### 2.对象继承

* 指定哪个对象是新对象的[[Prototype]]
* 对象字面形式会隐式指定Object.prototype为其[[Prototype]]
* Object.create()显示指定，接受两个参数
  * 需要被设置为新对象的[[Prototype]]的对象
  * 属性描述对象

``` javascript
var book = {
    title: 'The Principles of Object JavaScript'
};
// is the same as
var book = Object.create(Object.prototype,{
    title: {
        configurable: true,
        enumerable: true,
        value: "The Principles of Object JavaScript",
        writable: true
    }
})
```

``` javascript
var person1 = {
    name: 'nike',
    sayName: function(){
        console.log(this.name);
    }
};
var person2 = Object.create(person1,{
    name: {
        configurable: true,
        enumerable: true,
        value: 'greg',
        writable: true
    }
});

person1.sayName();                      // 'nike'
//自有属性隐藏并代替原型对象的同名属性
person2.sayName();                      // 'greg'

console.log(person1.hasOwnProperty('sayName'));     // true
console.log(person1.isPrototypeOf(person2));        // true
//sayName()依然只存在于person1,并被person2继承
console.log(person2.hasOwnProperty('sayName'));     // false
```

* 继承链：访问对象的属性，在对象实例上没发现该属性，则搜索[[Prototype]],仍没有发现，继续搜索该原型对象的[[Prototype]],直到继承链末端，末端通常是Object.prototype,其[[Prototype]]被置为null

``` javascript
//无原型对象链的对象，Object.prototype的内建方法都不存在于该对象
var nakedObject = Object.create(null);
console.log('toString' in nakedObject);            // false
```



#### 3.构造函数继承

修改或替换函数的prototype属性，此属性被自动设置为新继承自Object.prototype泛用对象，该对象有一个自有属性constructor

``` javascript
function YourConstructor() {
    // initialization
}

//js引擎会做
YourConstructor.prototype = Object.create(Object.prototype, {
                               constructor: {
                                  configurable: true,
                                   enumerable:  true,
                                   value: YourConstructor,
                                   writable: true
                               }
                            })
```

> YourConstructor创建出来的任何对象都继承自Object.prototype. YourConstructor是Object的子类，Object是YourConstructor的父类

``` javascript
function Rectangle(length, width) {
    this.length = length;
    this.width = width;
}
Rectangle.prototype = {
    constructor: Rectangle,
    getArea: function() {
        return this.length* this.width;
    },
    toString: function() {
        return "[Rectangle " + this.length + "x" + this.width + "]";
	}
}
function Square(size) {
    this.length = size;
    this.width = size;
}

Square.prototype = Object.create(Rectangle.prototype, {
                        constructor: {
                           configurable: true,
                            enumerable: true,
                            value: Square,
                            writable: true
                        }
                    });
Square.prototype.toString = function() {
    return "[Square" + this.length + "x" + this.width+ "]"
}
```

> 在对原型对象添加属性前要确保你已经改写了原型对象，否则在改写时会丢失之前添加的方法



#### 4.构造函数窃取（伪类继承）

在子类构造函数中调用父类的构造函数,做法如下

* 在子类构造函数中用call()或apply()调用父类的构造函数
* 并将新创建的对象传进去即可

``` javascript
function Square(size) {
    Rectangle.call(this, size, size);
}
```

伪类继承: 需要修改prototype来继承方法并用构造函数窃取来设置属性。这种做法模仿基于类的语言的类继承，被称为伪类继承。

#### 5.访问父类方法

上例中，Square类型有自己的toString()方法隐藏了其原型对象的toString()方法。若还想访问父类的方法：

* 通过call() 或apply()调用父类的原型对象的方法时传入一个子类的对象

``` javascript
Square.prototype.toString = function() {
    var text = Rectangle.prototype.toString.call(this);
    return text.replace("Rectangle", "Square");
}
```

> Square.prototype.toString() 通过call() 调用 Rectangle.prototype.toString(),这是唯一访问父类方法的手段。



###  第6章 对象模式

#### 1.实现私有属性

JavaScript对象的所有属性都是公有的，且没有显式的方法指定某个属性不能被外界某个对象访问。

在不希望公有的属性名字前加上下划线（this._name）,这种命名规则只是形式上的。

* 模块模式：

  * 用于创建拥有私有数据的单件对象的模式

  * 使用立调函数表达（IIFE）来返回一个对象

``` javascript
var person = (function() {
    var age = 25;
    function getAge() {
        return age;
    }
    function growOlder() {
        age++;
    }
    return {
        name: 'nicholas',
        getAge: getAge,
        growOlder: growOlder
    };
})
```

* 构造函数：

``` javascript
function Person(n,a) {
    this.name = n;
    //本地变量age， 用于getAge()，growOlder()方法
    var age = a;
    this.getAge = function(){
        return age;
    };
    this.growOlder = function() {
        age++;
    };
}

var person = new Person('nide',21);
console.log(person.getAge());                         // 21
```



#### 2.混入

  JavaScript还有一种伪继承的手段叫混入。

定义： 一个对象在不改变原型对象链的情况下得到另一个对象的属性被称为混入。

``` javascript
// 第一个对象(接收者)通过直接复制第二个对象(提供者)的属性从而接收了这些属性
function mixin(receiver, supplier) {
    for(var property in supplier) {
        if(supplier.hasOwnProperty(property)) {
            receiver[property] = supplier[property]
        }
    }
    return receiver;
}
```

> 记住这是浅拷贝，如果属性内包含的是一个对象，提供者和接收者将指向同一个对象

 **如果想要访问器属性被复制成访问器属性，需要一个不同的mixin() 函数，如下** 

``` javascript
function mixin(receiver, supplier) {
    object.keys(supplier).forEach(function(property) {
        var descriptor = Object.getOwnPropertyDescriptor(supplier, property);
        Object.defineProperty(receiver, property, descriptor);
    });
    return receiver;
}
```



#### 3.作用域安全的构造函数

当用new 调用一个函数时，this指向的新创建对象已经属于该构造函数所代表的自定义类型。

在严格模式下，构造函数不用new 操作符调用，会报错。

为保护那些偶尔忘记使用new的情况，可用instanceof 来检查自己是否被new 调用

``` javascript
function Person(name) {
    if(this instanceof Person) {
        this.name = name;
    } else {
        return new Person(name);
    }
}

var person1 = new Person('nike');
var person2 = Person('ali');

console.log(person1 instanceof Person);                   // true
console.log(person2 instanceof Person);                   // true

```




