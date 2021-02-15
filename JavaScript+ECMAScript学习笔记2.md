# JavaScript/ECMAScript学习笔记

### 10. 实例对象与new命令

对象是单个实物的抽象。对象是一个容器，封装了属性和方法。

- 构造函数

JavaScript语言的对象体系，不是基于“类”，而是基于构造函数和原型链。

构造函数，就是专门用来生成实例对象的函数。它就是对象的模板，描述实例对象的基本结构。一个构造函数可以生成多个实例对象。

```js
var Animal = function () {
    this.type = 'a';
};
```

构造函数第一个字母通常大写。

构造函数特点：函数体内使用了this关键字，代表了所要生成的对象实例。生成对象的时候，必须使用new命令

- new命令

```js
var Animal = function () {
    this.type = 'a';
};
var a = new Animal();
console.log(a.type);//'a'

var Animal2 = function (p) {
    this.type = p;
};
var b = new Animal2('b');
console.log(b.type);//'b'
```

如果忘了使用new命令，直接调用构造函数，构造函数就变成普通函数，并不会生成实例对象，this这时代表全局对象。

```js
var Animal = function () {
    this.type = 'a';
};
var a = Animal();
console.log(a);//undefined
console.log(type);//'a'
```

为了保证构造函数必须与new命令一起使用，一个解决方法是：构造函数内部使用严格模式，第一行加上'use strict'。严格模式中this不指向全局对象，默认等于undefined，JavaScript不允许对undefined添加属性。另一解决方法，构造函数内部判断是否使用new命令，如果发现没有使用，则直接返回一个实例对象：

```js
var Animal = function () {
    if (!(this instanceof Animal)) {
        return new Animal();
    }
    this.type = 'a';
};
var a = Animal();
console.log(a.type);//'a'
```

⭐new命令原理

①创建一个空对象，作为将要返回的对象实例

②将这个空对象的原型，指向构造函数的prototype属性

③将这个空对象赋值给函数内部的this关键字

④开始执行构造函数内部代码

this指的是一个新生成的空对象，所有针对this的操作，都会发生在这个空对象上。

如果构造函数内有return语句，而且return后面跟着一个对象，new命令返回return语句指定的对象，否则就不管return语句，返回this对象。但如果return语句返回的是一个跟this无关的新对象，new命令会返回这个新对象，而不是this对象。

```js
var Animal = function () {
    this.type = 'a';
    return {a:2};
};
var a = new Animal();
console.log(a);//{a:2}

var Animal = function () {
    this.type = 'a';
    return {type:2};
};
var a = new Animal();
console.log(a);//{type:2}

var Animal = function () {
    this.type = 'a';
    return 1;
};
var a = new Animal();
console.log(a === 1);//false
```

如果对普通函数（内部没有this关键字的函数）使用new命令，会返回一个空对象

⭐new命令内部简化流程：

```js
function _new(/* 构造函数 */ constructor, /* 构造函数参数 */ params) {
  // 将 arguments 对象转为数组
  var args = [].slice.call(arguments);
  // 取出构造函数
  var constructor = args.shift();
  // 创建一个空对象，继承构造函数的 prototype 属性
  var context = Object.create(constructor.prototype);
  // 执行构造函数
  var result = constructor.apply(context, args);
  // 如果返回结果是对象，就直接返回，否则返回 context 对象
  return (typeof result === 'object' && result != null) ? result : context;
}

// 实例
var actor = _new(Person, '张三', 28);
```

函数内部可使用new.target属性。如果当前函数是new命令调用，new.target指向当前函数，否则为undefined。

```js
function f() {
    console.log(new.target === f);
}
f();//false
new f();//true
```

- Object.create()创建实例对象

有时拿不到构造函数，只能拿到一个现有对象，以现有对象为模板生成新的实例对象。

```js
var p1 = {age:23};
var p2 = Object.create(p1);
console.log(p2.age);//23
```



### ⭐11. this关键字

this总是返回一个对象。this就是属性或方法“当前”所在的对象。

```js
var A = {
    name:'aaa',
    hello:function () {
        return this.name;
    }
};
var B = { name: 'bbb' };
B.hello = A.hello;
console.log(B.hello());//'bbb'
```

🔺JavaScript语言中，一切皆对象，运行环境也是对象，所以函数都是在某个对象之中运行，this就是函数运行时所在的对象（环境）。this指向是动态的。

- 实质

之所以有this的设计，跟内存里面的数据结构有关系。

由于函数可以在不同运行环境执行，所以需要有一种机制，能够在函数体内部获得当前运行环境（context）。this的设计目的就是在函数体内部，指代函数当前运行环境。

```js
var f = function () {
console.log(this.a);
};
var a = 1;
var obj = {
    f:f,
    a:2,
};
f();//1,单独执行
obj.f();//2，环境执行
```

- 使用场景

①全局环境

this指顶层对象window

②构造函数

this指代实例对象

③对象的方法

this指向就是方法运行时所在的对象。该方法赋值给另一对象，就会改变this指向。

```js
var obj = {
    test: function () {
        console.log(this);
    }
};
console.log(obj.test());//obj
console.log((obj.test = obj.test)());//window
console.log((false || obj.test)());//window
console.log((1,obj.test)());//window
```

JavaScript内部，obj和obj.test储存在两个内存地址，称为地址一和地址二。obj.test()调用时，从地址一调用地址二，地址二运行环境是地址一，this指向obj。其他三种情况，直接取出地址二调用，运行环境是全局环境，this指向全局环境。

🔺如果this所在方法不在对象第一层，this只是指向当前一层的对象，而不是继承更上面的层：

```js
var a = {
    text: 'aaa',
    b: {
        f: function () {
            console.log(this.text);
        }
    }
};
a.b.f();//undefined
```

该方法内部的this不是指向a，而是指向a.b。

要达到效果，需要写成这样：

```js
var a = {
    b: {
        text: 'aaa',
        f: function () {
            console.log(this.text);
        }
    }
};
a.b.f();//'aaa'

var v = a.b.f;
v();//undefined

var v2 = a.b;
v2.f();//'aaa'
```

如果这时将嵌套对象内部方法赋值给一个变量，this依然指向全局对象。

- 使用注意点

①避免多层this

```js
var obj = {
    f1:function () {
        console.log(this);
        var f2 = function () {
            console.log(this);
        }();
    }
};
obj.f1();
//Object
//Window
```

两层this，第一层指向对象obj，第二层指向全局对象。

解决方法是在第二层改用一个指向外层this的变量：

```js
var obj = {
    f1:function () {
        console.log(this);
        var that = this;
        var f2 = function () {
            console.log(that);
        }();
    }
};
obj.f1();
//Object
//Object
```

使用严格模式也可以避免这种问题。严格模式下，函数内部this指向顶层对象，会报错。

②避免数组处理方法中的this

数组的map方法和forEach方法，允许提供一个函数作为参数，这个函数内部不应使用this。

```js
var obj = {
    text:'a',
    p: ['a1','a2'],
    f:function f() {
        var that = this;
        this.p.forEach(item => {
            console.log(that.text+item);
        });
    }
};
obj.f();//'aa1' 'aa2'
```

也可以把this当作forEach方法第二个参数，固定它的运行环境：

```js
var obj = {
    text:'a',
    p: ['a1','a2'],
    f:function f() {
        this.p.forEach(item => {
            console.log(this.text+item);
        },this);
    }
};
obj.f();//'aa1' 'aa2'
```

③避免回调函数中的this

回调函数中this往往改变指向

- 绑定this方法

①Function.prototype.call()

指定函数内部this的指向

call方法参数应该是一个对象。如果参数为空、null和undefined，默认传入全局对象。如果参数是一个原始值，这个原始值会自动转成对应的包装对象，然后传入call方法。

```js
var x = 1;
var obj = { x: 2};
function f() {
    console.log(this.x);
}
f.call();//1
f.call(null);//1
f.call(undefined);//1
f.call(window);//1
f.call(obj);//2
```

call方法还可接受多个参数：后面参数表示函数调用时所需参数

```js
func.call(thisValue,arg1,arg2,...)
```

call方法一个应用时调用对象原生方法：

```js
var obj = {};
console.log(Object.prototype.hasOwnProperty.call(obj,'toString'));//false
```

②Function.prototype.apply()

接收一个数组作为函数执行时的参数：

```js
func.apply(thisValue,[arg1,arg2,...])
```

第一个参数如果设为null或undefined，等同于指定全局对象

```js
function f(x,y) {
    console.log(x+2*y);
}
f.call(null,2,4);//10
f.apply(null,[2,4]);//10
```

应用：

Ⅰ找出数组最大元素：

```js
var arr = [2,1,4,5,3];
console.log(Math.max.apply(null,arr));//5
```

Ⅱ将数组空元素变为undefined：

```js
console.log(Array.apply(null,[1,,2]));//[1,undefined,2]
```

Ⅲ转换类似数组的对象：被处理的对象必须有length属性，以及相对应的数字键

```js
console.log(Array.prototype.slice.apply({0:1,1:2,length:2}));//[1,2]
console.log(Array.prototype.slice.apply({0:2}));//[]
```

Ⅳ绑定回调函数的对象

③Function.prototype.bind()

返回一个新函数

如果方法第一个参数是null或undefined，等于将this绑定到全局对象。

```js
var counter = {
    cnt: 0,
    inc: function () {
        this.cnt++;
    }
};
var func = counter.inc.bind(counter);
func();
console.log(counter.cnt);//1

var obj = {cnt: 20};
var func2 = counter.inc.bind(obj);
func2();
console.log(obj.cnt);//21
```

bind()可接受更多参数，将这些参数绑定原函数的参数

```js
function f(a,b) {
    console.log(a+b);
}
var func = f.bind(null,2);//a = 2
func(3);//5，b = 3

f.bind(null,4,5)();//9
```

🔺注意点：

每一次返回一个新函数

```js
var listener = o.m.bind(o);
element.addEventListener('click', listener);
//  ...
element.removeEventListener('click', listener);
```

结合回调函数使用，常见错误是将包含this的方法直接当作回调函数

```js
var obj = {
    name: 'ann',
    test: [1,2],
    print: function () {
        this.test.forEach(function (n) {
            console.log(this.name);
        }.bind(this));
    }
};
obj.print();//"ann" "ann"
```

结合call()使用

```js
var slice = Function.prototype.call.bind(Array.prototype.slice);
console.log(slice([1,2,3],0,1));//[1]
```

将Array.prototype.slice变成Function.prototype.call方法所在的对象，调用时变成Array.prototype.slice.call

```js
function f() {
    console.log(this.v);
}
var obj = { v: 123};
var bind = Function.prototype.call.bind(Function.prototype.bind);
bind(f,obj)();//123
```

将Function.prototype.bind方法绑定在Function.prototype.call上，bind方法就可直接使用，不需要在函数实例上使用。



### ⭐12. 对象的继承

- 原型对象概述

同一个构造函数的多个实例之间，无法共享属性，造成系统资源浪费

```js
function Student(name) {
    this.name = name;
    this.age = function () {
        console.log(22);
    };
}
var stu1 = new Student('a');
var stu2 = new Student('b');
console.log(stu1.age === stu2.age);//false
```

解决方法就是JavaScript的原型对象。

继承机制设计思想：原型对象所有属性与方法，都能被实例对象共享。节省了内存，体现了实例对象之间的联系。

每个函数都有一个prototype属性，指向一个对象。对于普通函数该属性基本无用，对于构造函数，生成实例时该属性会自动成为实例对象的原型。

原型对象的属性不是实例对象自身的属性。只要修改原型对象，变动立刻体现在所有实例对象上。

🔺当实例对象本身没有某个属性或方法时，它会到原型对象寻找该属性或方法。如果实例对象自身就有某个属性或方法，就不会再去原型对象寻找这个属性或方法。

```js
function Student(name) {
    this.name = name;
}
Student.prototype.age = 22;
var stu1 = new Student('a');
var stu2 = new Student('b');
console.log(stu1.age,stu2.age);//22 22

stu1.age = 23;
Student.prototype.age = 24;
console.log(stu1.age,stu2.age);//23 24
```

⭐原型对象作用，是定义所有实例对象共享的属性和方法。实例对象可视为从原型对象衍生出来的子对象。

- 原型链

JavaScript规定，所有对象都有自己的原型对象（prototype）。任何一个对象都可以充当其他对象的原型，由于原型对象也是对象，所以它也有自己的原型。形成”原型链“，从对象到原型，再到原型的原型......

如果一层层往上追溯，所有对象的原型最终都可上溯到Object.prototype，即Object对象的prototype属性。Object.prototype的原型是null，null是原型链的尽头。

读取对象某个属性时，引擎先寻找对象本身的属性，如果找不到，就到它的原型去找，如果还是找不到，就到原型的原型去找，如果直到最顶层的Object.prototype还是找不到，则返回undefined。如果对象自身和它的原型，都定义了一个同名属性，优先读取对象自身属性。

一级级向上，在整个原型链上寻找门楼个属性，对性能是有影响的。

```js
var MyArray = function () {};
MyArray.prototype = new Array();
MyArray.prototype.constructor = MyArray;
var mine = new MyArray();
mine.push(1,2,3);
console.log(mine.length,mine instanceof Array);//3 true
```

- constructor属性

prototype对象有一个constructor属性，默认指向prototype对象所在的构造函数

①由于constructor属性定义在prototype上，意味着可被所有实例对象继承

```js
function F() {}
var f = new F();
console.log(f.constructor === F);//true
console.log(f.constructor === F.prototype.constructor);//true
console.log(f.hasOwnProperty('constructor'));//false
```

f自身没有constructor属性，是读取F.prototype.constructor属性。

②得知某个实例对象是哪一个构造函数产生

```js
function F() {}
var f = new F();
console.log(f.constructor === RegExp);//false
```

③可从一个实例对象新建另一个实例。在实例方法中可调用自身构造函数。

```js
function F() {}
var f = new F();
var f2 = new f.constructor();
console.log(f2 instanceof F);//true

F.prototype.createCopy = function () {
    return new this.constructor();
};
```

🔺修改原型对象时，一般要同时修改constructor属性的指向

```js
C.prototype.method = function (...) { ... };
```

🔺如不能确定constructor属性是什么函数，可通过name属性从实力得到构造函数名

```js
function F() {}
var f = new F();
console.log(f.constructor.name);//'F'
```

⭐普通对象constructor属性指向Object构造函数。

- instanceof运算符

表示对象是否为某个构造函数的实例

运算符左边是实例对象，右边是构造函数。检查优边构造函数的原型对象，是否在左边对象的原型链上。

```js
var a = new Animal();
console.log(a instanceof Animal);//true
console.log(a instanceof Object);//true
console.log(Animal.prototype.isPrototypeOf(a));//true
```

由于任意对象（除了null）都是Object的实例，所以instanceof运算符可判断一个值是否为非null的对象。

```js
console.log(null instanceof Object);//false
```

当左边对象的原型链上，只有null对象，instanceof运算符会失真：

```js
var obj = Object.create(null);
console.log(typeof obj);//"object"
console.log(obj instanceof Object);//false
```

instanceof运算符一个用处是判断值的类型。instanceof运算符只能用于对象，不适用原始类型值。

```js
var x = [];
console.log(x instanceof Array);//true
var y = 123;
console.log(y instanceof Number);//false
```

🔺undefined和null，instanceof运算符总是返回false

⭐利用instanceof运算符，可解决调用构造韩式时忘了加new命令的问题：

```js
function F() {
    if (this instanceof F) {
        this.a = 1;
    } else {
        return new F();
    }
}
var f = F();
console.log(f.a);//1
```

- 构造函数的继承

第一步：在子类构造函数中，调用父类的构造函数

第二步：让子类的原型指向父类的原型

```js
function Shape() {
    this.x = 1;
    this.y = 1;
}
Shape.prototype.move = function (x,y) {
    this.x += x;
    this.y += y;
    console.log(this.x,this.y);
};
function Square() {
    Shape.call(this);
}
Square.prototype = Object.create(Shape.prototype);
Square.prototype.constructor = Square;

var square = new Square();
square.move(2,3);//3 4
console.log(square instanceof Square);//true
console.log(square instanceof Shape);//true
```

上面是字类整体继承父类。有时只需要单个方法继承：

```js
ClassB.prototype.print = function() {
  ClassA.prototype.print.call(this);
  // some code
}
```

- 多重继承

JavaScript不提供多重继承功能，不允许一个对象同时继承多个对象，但还是可以通过混入（mixin）实现：

```js
function F1() {
    this.a = 1;
}
function F2() {
    this.b = 2;
}
function F() {
    F1.call(this);
    F2.call(this);
}
// 继承F1
F.prototype = Object.create(F1.prototype);
// 继承链上加入F2
Object.assign(F.prototype,F2.prototype);
// 指定构造函数
F.prototype.constructor = F;

var f = new F();
console.log(f.a,f.b);//1 2
```

- 模块

①基本实现方法

模块是实现特定功能的一组属性和方法的封装。

简单做法是把模块写成一个对象，所有模块成员都放到这个对象里。但这样写法会暴露所有模块成员，内部状态可被外部改写。

```js
var module1 = new Object({
　_count : 0,
　m1 : function (){
　　//...
　},
　m2 : function (){
  　//...
　}
});
```

②封装私有变量：构造函数写法

③封装私有变量：立即执行函数写法

将相关属性和方法封装在一个函数作用域里面，可达到不暴露私有成员目的。模块的基本写法：

```js
var module = (function () {
    var cnt = 0;
    var m1 = function () {
        console.log('1');
    };
    var m2 = function () {
        console.log('2');
    };
    return { m1: m1, m2: m2};
})();
console.log(module.cnt);//undefined
```

④模块的放大模式

一个模块很大，必须分成几个部分。或者一个模块要继承另一个模块。

```js
var module1 = (function (mod){
　mod.m3 = function () {
　　//...
　};
　return mod;
})(module1);
```

宽放大模式：立即执行函数的参数可以是空对象。

```js
var module1 = (function (mod) {
　//...
　return mod;
})(window.module1 || {});
```

⑤输入全局变量

独立性是模块重要特点，模块内部最好不与程序其他部分直接交互。

立即执行函数还可以起到命名空间的作用。

```js
(function($, window, document) {
  function go(num) {}
  function handleEvents() {}
  function initialize() {}
  function dieCarouselDie() {}
  //attach to the global scope
  window.finalCarousel = {
    init : initialize,
    destroy : dieCarouselDie
  }
})( jQuery, window, document );
```

上面的finalCarousel对象输出到全局，对外暴露init和destroy接口，四个内部方法都是外部无法调用的。



### 13. Object对象的OOP相关方法

- Object.getPrototypeOf()

返回参数对象原型。获取原型对象的标准方法。

```js
var F =  function () {};
var f = new F();
console.log(Object.getPrototypeOf(f) === F.prototype);//true
```

实例对象f的原型是F.prototype

🔺空对象的原型是Object.prototype。Object.prototype的原型是null。函数的原型是Function.prototype。

- Object.setPrototypeOf()

设置原型，返回该参数对象。接受两个参数，第一个是现有对象，第二个是原型对象。

```js
var a = {};
var b = {x:1};
Object.setPrototypeOf(a,b);
console.log(Object.getPrototypeOf(a) === b);//true
console.log(a.x);//1
```

将对象a的原型，设置为对象b。a可共享b属性。

new命令可使用该方法模拟：

```js
var F = function () {
    this.a = 1;
};
var f = new F();
// 等价于以下
var f = Object.setPrototypeOf({},F.prototype);
F.call(f);
```

第一步：将一个空对象的原型设为构造函数的prototype属性。

第二步：将构造函数内部的this绑定这个空对象，然后执行构造函数，使得定义在this上面的方法和属性都转移到这个空对象上。

- Object.create()

从一个实例对象生成另一个实例对象：

```js
var A = {
    test:function () {
        console.log('aaa');
    }
};
var B = Object.create(A);
Object.getPrototypeOf(B) === A;
console.log(B.test === A.test);//true
B.test();//'aaa'
```

实际上，Object.create()方法可用下面代码代替：

```js
if (typeof Object.create !== 'function') {
  Object.create = function (obj) {
    function F() {}
    F.prototype = obj;
    return new F();
  };
}
```

以下三种方式生成新对象是等价的：

```js
var obj1 = Object.create({});
var obj2 = Object.create(Object.prototype);
var obj3 = new Object();
```

如果想要生成一个不继承任何属性（如没有toString()和valueOf()）的对象，可将Object.create()参数设为null。

使用Object.create()时，必须提供对象原型，即参数不能为空，或者不是对象，否则会报错。

在原型上添加或修改任何方法，会立刻反映在新对象之上。

```js
var obj1 = {a:1};
var obj2 = Object.create(obj1);
obj1.a = 3;
console.log(obj2.a);//3
```

Object.create方法可接受第二个参数，该参数是一个属性描述对象，它所描述的对象属性，会添加到实例对象，作为该对象自身的属性。

Object.create()生成的对象，继承了它的原型对象的构造函数：

```js
function A() {}
var a = new A();
var b = Object.create(a);
console.log(b.constructor === A);//true
console.log(b instanceof A);//true
```

- Object.isPrototypeOf()

判断该对象是否为参数对象的原型。

```js
var obj1 = {};
var obj2 = Object.create(obj1);
var obj3 = Object.create(obj2);
console.log(obj2.isPrototypeOf(obj3));//true
console.log(obj1.isPrototypeOf(obj3));//true
```

obj1和obj2都是obj3的原型。

🔺只要实例对象处在参数对象的原型链上，isPrototypeOf方法都返回true。

- `Object.prototype.__proto__`

返回该对象的原型。该属性可读写。尽量少用这个属性。

```js
var A = {a:1};
var B = {a:2};
var proto = {
    test: function () {
        console.log(this.a);
    }
};
A.__proto__ = proto;
B.__proto__ = proto;
A.test();//1
B.test();//2
console.log(A.test === B.test);//true
console.log(A.test === proto.test);//true
console.log(B.test === proto.test);//true
```

A对象和B对象的原型都是proto对象，都共享proto对象的test方法。

- 获取原型对象方法的比较

获取实例对象obj的原型对象有三种方法：

`obj.__proto__`，obj.constructor.prototype，Object.getPrototypeOf(obj)。

前两种不可靠， `__proto__`属性只有在浏览器才需要部署，其他环境可不部署。obj.constructor.prototype在手动改变原型对象时可能失效。推荐用第三种。

- Object.getOwnPropertyNames()

返回数组，成员是参数对象本身的所有属性键名，不包含继承的属性键名，也不管是否可以遍历。只获取可遍历属性，使用Object.keys方法。

- Object.prototype.hasOwnProperty()

判断某个属性定义在对象自身，还是定义在原型链上。

🔺该方法是JavaScript中唯一一个处理对象属性时，不会遍历原型链的方法。

```js
console.log(Array.hasOwnProperty('length'));//true
console.log(Array.hasOwnProperty('toString'));//false
```

- in运算符和for...in循环

in运算符表示一个对象是否具有某个属性，不区分属性时对象自身属性还是继承属性。

获得对象的所有可遍历属性（不管是自身的还是继承的），可用for...in循环。

获得对象所有属性（不管是自身的还是继承的，也不管是否可枚举）：

```js
function inheritedPropertyNames(obj) {
    var props = {};
    while(obj) {
        Object.getOwnPropertyNames(obj).forEach(p => props[p] = true);
        obj = Object.getPrototypeOf(obj);
    }
    return Object.getOwnPropertyNames(props);
}
```

- 对象的拷贝

确保拷贝后的对象，与原对象具有同样的原型，与原对象具有同样的实例属性。

ES2017引入标准的Object.getOwnPropertyDescriptors方法：

```js
function copyObject(obj) {
    return Object.create(Object.getPrototypeOf(obj),Object.getOwnPropertyDescriptors(obj));
}
```



### 14. 严格模式

- 设计目的

①明确禁止一些不合理、不严谨的语法，减少JavaScript语言的一些怪异行为。

②增加更多报错场合，消除代码运行的一些不安全之处。保证代码运行的安全

③提高编译器效率，增加运行速度

④为未来新版本的JavaScript语法做好铺垫

- 启用方法

进入严格模式的标志，是一行字符串 `'use strict;'`

可用于整个脚本：放在脚本文件第一行

可用于单个函数：放在函数体第一行

有时需要把不同脚本合并在一个文件里，如果一个脚本是严格模式，另一个脚本不是，可考虑把整个脚本文件放在一个立即执行的匿名函数中：

```js
(function () {
  'use strict';
  // some code here
})();
```

- 显式报错

有些操作在正常模式下只会默默失败不报错，而严格模式下会报错。

①只读属性不可写

严格模式下，对只读属性赋值，或者删除不可配置属性都会报错。

②只设置了取值器的属性不可写

严格模式下，对一个只有getter、没有setter的属性赋值会报错。

③禁止扩展的对象不可扩展

严格模式下，对禁止扩展的对象添加新属性会报错。

④eval、arguments不可用作标识名

⑤函数不能有重名参数

正常模式下，如果函数有多个重名参数，可用arguments[i]读取。严格模式下属于语法错误。

⑥禁止八进制的前缀0表示法

- 增强的安全措施

①全局变量显式声明

正常模式下，如果一个变量没有声明就赋值，默认是全局变量。严格模式下，变量必须先声明再使用。

②禁止this关键字指向全局对象

严格模式的函数体内部this是undefined。

可用call、apply、bind方法将任意值绑定在this上面。

③禁止使用fn.callee、fn.caller

不能再函数内部得到调用栈

④禁止使用arguments.callee、arguments.caller

⑤禁止删除变量

严格模式下如果使用delete命令删除一个变量会报错。只有对象的属性，且属性的描述对象的configurable属性设为true，才能被delete命令删除。

- 静态绑定

JavaScript语言允许“动态绑定”，某些属性和方法属于哪一个对象，不是在编译时确定，而是在运行时确定。

严格模式规定某些情况下只允许静态绑定，属性和方法归属哪个对象，必须在编译阶段就确定。有利于编译效率提高。

①禁止使用with语句

②创设eval作用域

严格模式下，eval语句本身就是一个作用域，不再能够在其所运行的作用域创设新的变量了。eval所生成的变量只能用于eval内部。

③arguments不再追踪参数变化

严格模式下，函数内部改变参数与arguments的联系被切断了，两者不再存在联动关系。

- 向下一个版本的JavaScript过渡

严格模式引入了一些ES6语法。

①非函数代码块不得声明函数

ES5的严格模式只允许在全局作用域或函数作用域声明函数。ES6引入了块级作用域，允许在代码块中声明函数：

```js
'use strict';
if (true) {
  function f1() { } // 语法错误
}

for (var i = 0; i < 5; i++) {
  function f2() { } // 语法错误
}
```

上面例子ES5会报错，ES6可用。

②保留字

严格模式新增了一些保留字：implements、interface、let、package、private、protected、publc、static、yield等



### ⭐15. 异步操作

**15.1 概述**

- 单线程模型

单线程模型指JavaScript只在一个线程上运行。即JavaScript同时只能执行一个任务，其他任务都必须在后面排队等待。

事实上，JavaScript引擎有多个线程，单个脚本只能在一个线程上运行（主线程），其他线程都是在后台配合。

好处是实现起来简单，执行环境相对单纯；坏处是只要有一个任务耗时很长，后面的任务都必须排队等着，拖延整个程序执行。常见的浏览器无响应（假死），往往是因为某一段JavaScript代码长时间运行（如死循环），导致整个页面卡在这个地方，其他任务无法执行。

事件循环机制：CPU可以不管IO操作，挂起处于等待中的任务，先运行排在后面的任务。等到IO操作返回结果，再回过头，把挂起的任务继续执行下去。

- 同步任务与异步任务

同步任务：那些没有被引擎挂起，在主线程上排队执行的任务。只有前一个任务执行完毕，才能执行后一个任务。

异步任务：那些被引擎放在一边，不进入主线程，而进入任务队列的任务。只有引擎认为某个异步任务可以执行（如Ajax操作从服务器得到结果），该任务（采用回调函数形式）才会进入主线程执行。排在异步任务后面的代码，不用等待异步任务结束会马上运行，即异步任务不具有“堵塞”效应。

- 任务队列和事件循环

JavaScript运行时，除了一个正在运行的主线程，引擎还提供一个任务队列，里面是各种需要当前程序处理的异步任务。（根据异步任务类型，实际上存在多个任务队列）。

主线程会去执行所有同步任务，等同步任务全部执行完，就会去看任务队列里面的异步任务。如果满足条件，那么异步任务就重新进入主线程开始执行，这时它就变成同步任务了。等到执行完，下一个异步任务再进入主线程开始执行。一旦任务队列清空，程序结束执行。

异步任务写法通常是回调函数。一旦异步任务重新进入主线程，就会执行对应的回调函数。如果一个异步任务没有回调函数，就不会进入任务队列，即不会进入主线程，因为没有回调函数指定下一步操作。

引擎在不停检查，一遍又一遍，只要同步任务执行完，引擎就会去检查那些挂起来的异步任务，是不是可以进入主线程了。这种循环检查的机制叫事件循环。事件循环是一个程序结构，用于等待和发送消息和事件。

- 异步操作的模式

①回调函数

```js
function f1() {}
function f2() {}
f1();
f2();
```

如果f1()是异步操作，f2()会立即执行，不会等到f1结束再执行。

想要f2等到f1执行完成再执行，需要改写：把f2写成f1的回调函数。

```js
function f1(callback) {
	//...
	callback();
}
function f2() {}
f1(f2);
```

优点：简单、容易理解和实现。

缺点：不利于代码阅读和维护，各个部分之间高度耦合，程序结构混乱，流程难以追踪，而且每个任务只能指定一个回调函数。

②事件监听

异步任务的执行不取决于代码顺序，而取决于某个事件是否发生。

```js
function f1() {
	setTimeout(function () {
	//...
	f1.trigger('done');//执行完成后立即触发done事件
	},1000);
}
f1.on('done',f2);//当f1发生done事件，就执行f2
```

优点：易理解，可绑定多个事件，每个事件可指定多个回调函数，去耦合，利于实现模块化。

缺点：整个程序变成事件驱动型，运行流程不清晰，阅读代码时难以看出主线程。

③发布/订阅

事件完全可理解为“信号”，如果存在一个“信号中心”，某个任务执行完成，就向信号中心“发布”一个信号，其他任务可以向信号中心“订阅”这个信号。从而知道什么时候自己可以开始执行，叫做“发布/订阅模式”，或者“观察者模式”。

```js
jQuery.subscribe('done',f2);//f2向信号中心订阅done信号
function f1() {
	setTimeout(function () {
	//...
	jQuery.publish('done');//f1执行完成，向信号中心jQuery发布done信号，从而引发f2执行
	},1000);
}
jQuery.unsubscribe('done',f2);//取消订阅
```

与“事件监听”类似，但优于“事件监听”，可以通过查看“消息中心“，了解存在多少信号、每个信号有多少订阅者，从而监控程序运行。

- 异步操作的流程控制

①串行执行

一个任务完成之后，再执行另一个。

```js
var items = [ 1, 2, 3, 4, 5, 6 ];
var results = [];

function async(arg, callback) {
    console.log('参数为 ' + arg +' , 1秒后返回结果');
    setTimeout(function () { callback(arg * 2); }, 1000);
}

function final(value) {
    console.log('完成: ', value);
}

function series(item) {
    if(item) {
        async( item, function(result) {
            results.push(result);
            return series(items.shift());
        });
    } else {
        return final(results[results.length - 1]);
    }
}

series(items.shift());
```

6秒完成整个脚本。

②并行执行

所有异步任务同时执行，等到全部完成后，才执行final函数。

forEach方法同时发起六个异步任务。

这个写法只需要1秒就可以完成整个脚本。并行执行效率较高，但如果并行任务较多，很容易耗尽系统资源，拖慢运行速度。

```js
var items = [ 1, 2, 3, 4, 5, 6 ];
var results = [];

function async(arg, callback) {
    console.log('参数为 ' + arg +' , 1秒后返回结果');
    setTimeout(function () { callback(arg * 2); }, 1000);
}

function final(value) {
    console.log('完成: ', value);
}

items.forEach(function(item) {
    async(item, function(result){
        results.push(result);
        if(results.length === items.length) {
            final(results[results.length - 1]);
        }
    })
});
```

③并行与串行的结合

设置一个门槛，每次最多只能并行执行n个异步任务，避免过分占用系统资源。

```js
var items = [ 1, 2, 3, 4, 5, 6 ];
var results = [];
var running = 0;
var limit = 2;

function async(arg, callback) {
    console.log('参数为 ' + arg +' , 1秒后返回结果');
    setTimeout(function () { callback(arg * 2); }, 1000);
}

function final(value) {
    console.log('完成: ', value);
}

function launcher() {
    while(running < limit && items.length > 0) {
        var item = items.shift();
        async(item, function(result) {
            results.push(result);
            running--;
            if(items.length > 0) {
                launcher();
            } else if(running == 0) {
                final(results);
            }
        });
        running++;
    }
}

launcher();
```

3秒完成整个脚本，通过调节limit变量，达到效率和资源的最佳平衡。

**15.2 定时器**

setTimeout()和setInterval()这两个函数，向任务队列添加定时任务。

- setTimeout()

指定某个函数或某段代码，在多少毫秒之后执行。返回一个整数，表示定时器编号，以后可用来取消这个定时器。

setTimeout的第二个参数如果省略，默认为0

setTimeout允许更多参数，依次传入推迟执行的函数（回调函数）

```js
console.log(1);
setTimeout(function(a,b) {
    console.log(a+b);
},1000,1,1);
console.log(3);
//1 3 2
```

如果回调函数是对象的方法，那么setTimeout使得方法内部this关键字指向全局环境，而不是定义时所在的那个对象。

```js
var a = 1;
var obj = {
    a:2,
    b: function () {
        console.log(this.a);
    }
};
setTimeout(obj.b,1000);//1
//解决方法1：放入一个函数
setTimeout(function () {
    obj.b();//2
},1000);
//解决方法2：bind方法绑定
setTimeout(obj.b.bind(obj),1000);//2
```

- setInterval()

setInterval指定某个任务每隔一段时间执行一次，无限次定时执行。

setInterval还可接受更多参数，它们会传入回调函数。

```js
setInterval(function () {
    console.log(1);
},500);
```

setInterval一个常见用途是实现轮询。

```js
var hash = window.location.hash;
var hashWatcher = setInterval(function() {
  if (window.location.hash != hash) {
    updatePage();
  }
}, 1000);
```

🔺setInterval指定的是”开始执行“之间的间隔，并不考虑每次任务执行本身所消耗的时间。两次执行之间间隔会小于指定时间，如指定100ms执行一次，每次执行需要5ms，那么第一次执行结束后95ms第二次执行会开始。如果某次执行耗时特别长，如需要105ms，那么它结束后下一次执行就会立即开始。

⭐为了确保两次执行之间有固定的间隔，可不用setInterval，而是每次执行结束后，使用setTimeout指定下一次执行的具体时间。

```js
var i = 1;
var timer = setTimeout(function f() {
  // ...
  timer = setTimeout(f, 2000);
}, 2000);
```

- clearTimeout()、clearInterval()

setTimeout和setInterval函数都返回一个整数值表示计数器编号，将该整数传入clearTimeout和clearInterval函数就可取消对应定时器。

setTimeout和setInterval返回整数值是连续的。

```js
var num1 = setTimeout(function() {}, 1000);
var num2 = setInterval(function() {},1000);
clearTimeout(num1);
clearInterval(num2);
```

🔺取消当前所有的setTimeout定时器：

```js
(function() {
  // 每轮事件循环检查一次
  var gid = setInterval(clearAllTimeouts, 0);

  function clearAllTimeouts() {
    var id = setTimeout(function() {}, 0);
    while (id > 0) {
      if (id !== gid) {
        clearTimeout(id);
      }
      id--;
    }
  }
})();
```

- 实例：debounce函数

debounce：防抖动

```js
$('textarea').on('keydown', debounce(ajaxAction, 2500));

function debounce(fn, delay){
  var timer = null; // 声明计时器
  return function() {
    var context = this;
    var args = arguments;
    clearTimeout(timer);
    timer = setTimeout(function () {
      fn.apply(context, args);
    }, delay);
  };
}
```

只要2500ms以内用户再次击键，就取消上一次的定时器，然后再新建一个定时器。保证了回调函数之间的调用间隔，至少是2500ms。

- 运行机制

setTimeout和setInterval运行机制，是将指定代码移出本轮事件循环，等到下一轮事件循环，再检查是否到了指定时间。如果到了就执行对应代码；如果不到就继续等待。

setTimeout和setInterval指定的回调函数，必须等到本轮事件循环所有同步任务都执行完，才会开始执行。

```js
setTimeout(someTask, 100);
veryLongTask();
```

被推迟的someTask只有等着，等veryLongTask运行结束才轮到它执行。

```js
setInterval(function () {
    console.log(2);
}, 1000);

sleep(3000);

function sleep(ms) {
    var start = Date.now();
    while ((Date.now() - start) < ms) {
    }
}
```

过了3000ms才会执行setInterval()

- setTimeout(f,0)

setTimeout(f,0)会在下一轮事件循环一开始就执行。

目的是尽可能早地执行f，但并不能保证立刻就执行f

```js
setTimeout(function () {
    console.log(1);
},0);
console.log(2);
//2 1
```

应用1：调整事件发生顺序。如某个事件先发生在子元素，然后冒泡到父元素，即子元素的事件回调函数，早于父元素的事件回调函数触发。要让父元素的事件回调函数先发生，就用到setTimeout(f,0)

```js
// HTML 代码如下
// <input type="button" id="myButton" value="click">

var input = document.getElementById('myButton');

input.onclick = function A() {
  setTimeout(function B() {
    input.value +=' input';
  }, 0)
};

document.body.onclick = function C() {
  input.value += ' body'
};
```

先触发回调函数A，然后触发函数C。函数A中setTimeout将函数B推迟到下一轮时间循环执行。

应用2：用户自定义的回调函数通常在浏览器默认动作之前触发。如用户在输入框输入文本，keypress事件会在浏览器接受文本之前触发。要使得事件在浏览器接收到文本之后触发：

```js
// HTML 代码如下
// <input type="text" id="input-box">

document.getElementById('input-box').onkeypress = function() {
  var self = this;
  setTimeout(function() {
    self.value = self.value.toUpperCase();
  }, 0);
}
```

应用3：计算量大、耗时长的任务，常常会被放到几个小部分，分别放到setTimeout(f,0)内执行。如代码块很大时代码高亮的处理，以及改变一个网页元素背景色：

```js
var div = document.getElementsByTagName('div')[0];

// 写法一，浏览器“堵塞”，JavaScript执行速度远高于DOM，造成大量DOM操作“堆积”
for (var i = 0xA00000; i < 0xFFFFFF; i++) {
  div.style.backgroundColor = '#' + i.toString(16);
}

// 写法二
var timer;
var i=0x100000;

function func() {
  timer = setTimeout(func, 0);
  div.style.backgroundColor = '#' + i.toString(16);
  if (i++ == 0xFFFFFF) clearTimeout(timer);
}

timer = setTimeout(func, 0);
```

**15.3 Promise对象**

- 概述

Promise对象是JavaScript的异步操作解决方案，为异步操作提供统一接口。起到代理作用，充当异步操作与回调函数之间的中介，使得异步操作具备同步操作的接口。

Promise可让异步操作写起来就像在写同步操作的流程，而不必一层层地嵌套回调函数。

Promise是一个对象，也是一个构造函数。

```js
function f1(resolve, reject) {
  // 异步代码...
}

var p1 = new Promise(f1);
p1.then(f2);//f1的异步操作执行完成，就会执行f2
```

Promise的设计思想是，所有异步任务都返回一个Promise实例。Promise实例有一个then方法，用来指定下一步的回调函数。

- Promise对象的状态

Promise对象通过自身的状态来控制异步操作。

三种状态：异步操作未完成（pending）、异步操作成功（fulfilled）、异步操作失败（rejected）。fulfilled和rejected合在一起称为resolved（已定型）。

三种状态的变化途径：从“未完成”到“成功”和从“未完成”到“失败”。一旦状态发生变化，就凝固了，不会再有新的状态变化。

🔺Promise的最终结果只有两种：①异步操作成功，Promise实例传回一个值（value），状态变为fulfilled。②异步操作失败，Promise实例抛出一个错误（error），状态变为rejected。

- Promise构造函数

```js
var promise = new Promise(function (resolve, reject) {
  // ...

  if (/* 异步操作成功 */){
    resolve(value);
  } else { /* 异步操作失败 */
    reject(new Error());
  }
});
```

Promise构造函数接受一个函数作为参数，该函数的两个参数分别是resolve和reject。

- Promise.prototype.then()

添加回调函数。then方法可接受两个回调函数，第一个是异步操作成功时的回调函数，第二个是异步操作失败时的回调函数。一旦状态改变，就调用相应的回调函数。

```js
var p1 = new Promise((resolve, reject) => {
    resolve('success');
});
p1.then(console.log,console.error);//success
var p2 = new Promise((resolve, reject) => {
    reject(new Error('fail'));
});
p2.then(console.log,console.error);//Error:fail
```

then()方法可链式调用：

```js
p1
  .then(step1)
  .then(step2)
  .then(step3)
  .then(
    console.log,
    console.error
  );
```

console.log只显示step3的返回值，而console.error可显示p1、step1、step2、少特p之中任意一个发生的错误。

如果step1的状态变为rejected，step2和step3都不会执行了。Promise开始寻找接下来第一个为rejected的回调函数。Promise对象的报错具有传递性。

⭐then()用法辨析

```js
// 写法一
f1().then(function () {
  return f2();
});

// 写法二
f1().then(function () {
  f2();
});

// 写法三
f1().then(f2());

// 写法四
f1().then(f2);
```

再用then方法接受一个回调函数f3。

```js
f1().then(function () {
  return f2();
}).then(f3);
```

写法一的f3回调函数的参数，是f2函数的运行结果。

```js
f1().then(function () {
  f2();
  return;
}).then(f3);
```

写法二的f3回调函数的参数是undefined。

```js
f1().then(f2())
  .then(f3);
```

写法三的f3回调函数的参数，是f2函数返回的函数的运行结果。

```js
f1().then(f2)
  .then(f3);
```

写法四与写法一只有一个差别，那就是f2会接收到f1()返回的结果。

- 实例：图片加载

```js
var preloadImage = function (path) {
  return new Promise(function (resolve, reject) {
    var image = new Image();
    image.onload  = resolve;
    image.onerror = reject;
    image.src = path;
  });
};
```

- 微任务

Promise的回调函数属于异步任务，会在同步任务之后执行。

Promise的回调函数不是正常的异步任务，而是微任务。区别在于，正常任务追加到下一轮事件循环，微任务追加到本轮事件循环。意味着，微任务的执行时间一定早于正常任务。

```js
setTimeout(function() {
    console.log(1);
});
new Promise(function (resolve,reject) {
    resolve(2);
}).then(console.log);
console.log(3);
// 3 2 1
```

then是本轮事件循环执行，setTimeout(fn,0)在下一轮事件循环开始时执行。

- 小结

优点：让回调函数变成了规范的链式写法，可实现许多功能，如同时执行多个异步操作，等到它们状态都改变后再执行一个回调函数；或为多个回调函数中抛出的错误统一指定处理方法等。另外，Promise的状态一旦改变，无论何时查询，都能得到这个状态。意味着无论何时为Promise实例添加回调函数，该函数都能正确执行。传统写法通过监听事件来执行回调函数，一旦错过了事件再添加回调函数是不会执行的。

缺点：编写难度较高。



### 16. DOM

- 概述

DOM是JavaScript操作网页的接口，全称为“文档对象模型”。作用是将网页转为一个JavaScript对象，从而可以用脚本进行各种操作。

浏览器会根据DOM类型，将结构化文档（如HTML和XML）解析成一系列节点，再由这些节点组成一个树状结构（DOM Tree）。所有节点和最终的树状结构，都有规范的对外接口。

DOM最小组成单位叫做节点。文档的树形结构（DOM树）就是由各种不同类型节点组成。

节点类型有7种：①document（整个文档树的顶层节点）②DocumentType（doctype标签）③Element（网页各种HTML标签）④Attr（网页元素的属性）⑤Text（标签之间或标签包含的文本）⑥Comment（注释）⑦DocumentFragment（文档的片段）

浏览器提供一个原生节点对象Node，上面这7种节点都继承了Node。

文档的第一层有2个节点，第一个是文档类型节点（`<!doctype html>`），第二个是HTML网页顶层容器标签 `<html>`。后者构成了树结构的根节点，其他HTML标签节点都是它的下级节点。

除了根节点，其他节点都有三种层级关系：①父节点关系（直接的那个上级节点）②子节点关系（直接的下级节点）③同级节点关系（拥有同一个父节点的节点）

DOM提供操作接口，用来获取这三种关系的节点。

**16.1 Node接口**

属性

- Node.prototype.nodeType

返回整数值，表示节点类型

文档节点：9，对应常量Node.DOCUMENT_NODE

元素节点：1，对应常量Node.ELEMENT_NODE

属性节点：2，对应常量Node.ATTRIBUTE_NODE

文本节点：3，对应常量Node.TEXT_NODE

文档片段节点：11，对应常量Node.DOCUMENT_FRAGMENT_NODE

文档类型节点：10，对应常量Node.DOCUMENT_TYPE_NODE

注释节点：8，对应常量Node.COMMENT_NODE

- Node.prototype.nodeName

返回节点名称。

不同节点nodeName属性值如下：

文档节点：#document

元素节点：大写的标签名

属性节点：属性的名称

文本节点：#text

文档片段节点：#document-fragment

文档类型节点：文档的类型

注释节点：#comment

- Node.prototype.nodeValue

返回字符串，表示当前节点本身的文本值，该属性可读写。

只有文本节点、注释节点和属性节点有文本值，可返回结果，可设置值。其他类型的节点一律返回null，设置值无效。

- Node.prototype.textContent

返回当前节点和它的所有后代节点的文本内容。

textContent属性自动忽略当前节点内部的HTML标签，返回所有文本内容。该属性可读写。它可自动对HTML标签转义。

对于文本节点、注释节点和属性节点，textContent属性值与nodeValue属性相同。对于其他类型节点，该属性会将每个子节点（不含注释节点）内容连接在一起返回，如果一个节点没有子节点，则返回空字符串。

文档节点和文档类型节点的textContent属性为null。要读取整个文档内容，可使用document.documentElement.textContent

```js
// HTML
// <p id="p1">111<span>2</span></p>
var text = document.getElementById('p1');
console.log(text.nodeType,text.nodeValue,text.textContent,text.nodeName);//1 null "1112" "P"
text.textContent = '<h1>111</h1>';
console.log(text.textContent);//<h1>111</h1>
```

- Node.prototype.baseURI

返回字符串，表示当前网页绝对路径。浏览器根据这个属性，计算网页上的相对路径URL。该属性只读。如无法读取网页URL，返回null。

该属性值一般由当前网址URL（window.location属性）决定，但可使用HTML的 `<base>`标签改变该属性的值。

- Node.prototype.ownerDocument

返回当前节点所在的顶层文档对象，即document对象。

documentt对象本身的ownerDocument属性，返回null。

- Node.prototype.nextSibling、Node.prototype.previousSibling

返回紧跟在当前节点后面（前面距离最近）的第一个同级节点。如果当前节点后面（前面）没有同级节点，返回null。

该属性还包括文本节点和注释节点。如果当前节点后面（前面）有空格，该属性会返回一个文本节点，内容为空格。

```js
// HTML
// <div id="d1">test</div><div id="d2">test2</div>
var d1 = document.getElementById('d1');
var d2 = document.getElementById('d2');
console.log(d1.nextSibling === d2);//true
console.log(d2.previousSibling === d1);//true
```

- Node.prototype.parentNode

返回当前节点的父节点。对于一个节点，它的父节点只可能是三种：元素节点、文档节点和文档片段节点。

文档节点和文档片段节点的父节点都是null。对那些生成后还没插入DOM树的节点，父节点也是null。

- Node.prototype.parentElement

返回当前节点的父元素节点。如果当前节点没有父节点，或者父节点类型不是元素节点，则返回null。

- Node.prototype.firstChild、Node.prototype.lastChild

返回当前节点的第一个（最后一个）子节点，如果当前节点没有子节点，返回null。

firstChild返回的除了元素节点，还可能是文本节点或注释节点。

- Node.prototype.childNodes

返回一个类似数组的对象（NodeList集合），成员包括当前节点的所有子节点。

文档节点就有两个子节点：文档类型节点和HTML根元素节点

除了元素节点，childNodes属性返回值还包括文本节点和注释节点。如果当前节点不包括任何子节点，则返回一个空的NodeList集合。NodeList对象是一个动态集合，一旦子节点发生变化，立刻反映在返回结果中。

```js
// HTML
// <div id="div1"><p id="text1">111</p><span>23</span></div>
var div1 = document.getElementById('div1');
console.log(div1.childNodes,div1.firstChild,div1.lastChild);
var text1 = document.getElementById('text1');
console.log(text1.parentElement,text1.parentNode);
```

- Node.prototype.isConnected

返回布尔值，表示当前节点是否在文档中。

```js
var test = document.createElement('p');
test.isConnected // false

document.body.appendChild(test);
test.isConnected // true
```

方法

- Node.prototype.appendChild()

接受一个节点对象作为参数，将其作为最后一个子节点，插入当前节点。该方法的返回值就是插入文档的子节点。

如果参数是DOM已经存在的节点，该方法会将其从原来的位置移动到新位置。

如果参数是DocumentFragment节点，插入的是DocumentFragment的所有子节点，而不是DocumentFragment本身。返回值是一个空的DocumentFragment节点。

- Node.prototype.hasChildNodes()

返回布尔值，表示当前节点是否有子节点。子节点包括所有类型的节点。

🔺判断一个节点有没有子节点，有许多方法：

node.hasChildNodes()、node.firstChild !== null、node.childNodes && node.childNodes.length > 0

⭐遍历当前节点的所有后代节点：

```js
function DOMComb(parent, callback) {
  if (parent.hasChildNodes()) {
    for (var node = parent.firstChild; node; node = node.nextSibling) {
      DOMComb(node, callback);
    }
  }
  callback(parent);
}

// 用法
DOMComb(document.body, console.log)
```

- Node.prototype.cloneNode()

克隆一个节点。接受一个布尔值作为参数，表示是否同时克隆子节点。返回值是一个克隆出来的新节点。

🔺①克隆一个节点，会拷贝该节点的所有属性，但会丧失addEventListener方法和on-属性，添加在这个节点上的事件回调函数。②该方法返回的节点不在文档中，即没有父节点。③克隆一个节点后，DOM可能出现两个有相同id属性的网页元素，这时应该修改其中一个元素的id属性。如果原节点有name属性，可能也需要修改。

- Node.prototype.insertBefore()

将某个节点插入父节点内部的指定位置。

接受两个参数，第一个参数是所要插入的节点，第二个参数是父节点内部的一个子节点。返回值是插入的新节点。

如果第二个参数是null，新节点将插在当前节点内部的最后位置。

如果所要插入的节点是当前DOM现有节点，该节点将从原有位置移除，插入新的位置。

⭐如果新节点要插在某个父节点后面：

```js
parent.insertBefore(s1, s2.nextSibling);
```

如果要插入的节点是DocumentFragment类型，那么插入的将是DocumentFragment的所有子节点，而不是DocumentFragment节点本身。返回值将是一个空的DocumentFragment节点。

```js
// HTML
// <div id="div1"><p id="text1">111</p><span>23</span></div>
var div1 = document.getElementById('div1');
var a = document.createElement('a');
div1.insertBefore(a,div1.firstChild);
console.log(div1.childNodes[0]);//<a></a>
```

- Node.prototype.removeChild()

接受一个子节点作为参数，用于从当前节点移除该子节点。返回值是移除的子节点。

```js
var divA = document.getElementById('A');
divA.parentNode.removeChild(divA);
```

⭐移除当前节点的所有子节点：

```js
var element = document.getElementById('top');
while (element.firstChild) {
  element.removeChild(element.firstChild);
}
```

被移除的节点依然存在于内存中，但不再是DOM一部分。一个节点移除后依然可以使用它，如插入到另一个节点下。

如果参数不是当前节点的子节点，方法报错。

- Node.prototype.replaceChild()

将一个新的节点，替换当前节点的某一个子节点。

接受两个参数，第一个参数是用来替换的新节点，第二个参数是将要替换走的子节点。返回值是替换走的那个节点。

- Node.prototype.contains()

返回布尔值，表示参数节点是否满足三个条件之一：参数节点为当前节点、参数节点为当前节点的子节点，参数节点为当前节点的后代节点。

- Node.prototype.compareDocumentPosition()

返回一个六个比特位的二进制值，表示参数节点与当前节点关系：

| 二进制值 | 十进制值 | 含义                   |
| -------- | -------- | ---------------------- |
| 000000   | 0        | 两个节点相同           |
| 000001   | 1        | 两个节点不在同一个文档 |
| 000010   | 2        | 参数节点在当前节点前面 |
| 000100   | 4        | 参数节点在当前节点后面 |
| 001000   | 8        | 参数节点包含当前节点   |
| 010000   | 16       | 当前节点包含参数节点   |
| 100000   | 32       | 浏览器内部使用         |

```js
// HTML
// <div id="div1"><p id="text1">111</p><span>23</span></div>
var div1 = document.getElementById('div1');
var text = document.getElementById('text1');
console.log(div1.compareDocumentPosition(text));//20
```

```js
var head = document.head;
var body = document.body;
if (head.compareDocumentPosition(body) & 4) {
  console.log('文档结构正确');
} else {
  console.log('<body> 不能在 <head> 前面');
}
```

- Node.prototype.isEqualNode()、Node.prototype.isSameNode()

isEqualNode()返回布尔值，用于检查两个节点是否相等（两个节点类型相同，属性相同、子节点相同）。isSameNode()返回布尔值，表示两节点是否为同一个节点。

```js
var a = document.createElement('a');
var b = document.createElement('a');
console.log(a.isEqualNode(b));//true
console.log(a.isSameNode(a));//true
```

- Node.prototype.normalize()

清理当前节点内部的所有文本节点。去除空的文本节点，并将毗邻的文本节点合并成一个，即不存在空的文本节点，以及毗邻的文本节点。

该方法是Text.splitText的逆方法。

```js
var wrapper = document.createElement('div');
wrapper.appendChild(document.createTextNode('1 '));
wrapper.appendChild(document.createTextNode('2 '));
console.log(wrapper.childNodes.length);//2
wrapper.normalize();
console.log(wrapper.childNodes.length);//1
```

- Node.prototype.getRootNode()

返回当前节点所在文档的根节点document，与ownerDocument属性作用相同。

```js
document.getRootNode() // document
document.ownerDocument // null
```

**16.2 NodeList接口，HTMLCollection接口**

DOM提供两种节点集合，用于容纳多个节点：NodeList和HTMLCollection

NodeList可包含各种类型节点，HTMLCollection只能包含HTML元素节点

①NodeList

NodeList实例是一个类似数组的对象，成员是节点对象。

得到NodeList实例的方法：Node.childNodes、document.querySelectorAll()

NodeList可使用length属性和forEach方法。

只有Node.childNodes返回的是一个动态集合（DOM删除或新增一个相关节点，都会立刻反映在NodeList实例），其他NodeList都是静态集合。

- NodeList.prototype.length

返回NodeList实例包含的节点数量

- NodeList.prototype.forEach()

遍历NodeList的所有成员。接受一个回调函数作为参数。函数的三个参数是当前成员、位置和当前NodeList实例

- NodeList.prototype.item()

接受一个整数值作为参数，表示成员位置，返回该位置上的成员。

如果参数值大于实际长度，或者索引不合法，item方法返回null。如果省略参数，方法报错。

一般情况下，使用方括号运算符取出成员而不是item方法。

- NodeList.prototype.keys()、NodeList.prototype.values()、NodeList.prototype.entries()

返回一个ES6遍历器对象，可通过for...of循环遍历获取每一个成员信息。keys()返回键名的遍历器，values()返回键值的遍历器，entries()返回的遍历器同时包含键名和键值信息。

```js
// HTML
// <div id="div1"><p id="text1">111</p><span>23</span></div>
var children = document.getElementById('div1').childNodes;
console.log(children.length,children.item(1));//2 <span>23</span>
document.getElementById('div1').appendChild(document.createElement('a'));
console.log(children.length);//3
children.forEach(item => console.log(item));
```

②HTMLCollection

返回值是一个类似数组的对象，HTMLCollection没有forEach方法，只能用for循环遍历。

返回HTMLCollection实例的，主要是一些Document对象的集合属性，如document.links、document.forms、document.images

实例是动态集合，节点变化实时反映在集合中。

 如果节点有id或name属性，那么HTMLCollection实例上，可使用id或name属性引用该节点元素。如果没有对应节点，返回null。

- HTMLCollection.prototype.length

返回HTMLCollection实例包含的成员数量

- HTMLCollection.prototype.item()

接受一个整数值作为参数，表示成员位置，返回该位置上成员。

如果参数值超出成员数量或者不合法，item方法返回null。

一般情况下，使用方括号运算符取出成员。

- HTMLCollection.prototype.namedItem()

参数是一个字符串，表示id属性或name属性的值，返回对应元素节点。如果没有对应节点，返回null。

```js
// HTML
// <div id="div1"><a id="text1" href="#">111</a><a href="#">23</a></div>
console.log(document.links.namedItem('text1') === document.getElementById('text1'));//true
console.log(document.links.length);//2
```

**16.3 ParentNode接口、ChildNode接口**

ParentNode接口表示当前节点是一个父节点，提供一些处理子节点的方法。

ChildNode接口表示当前节点是一个子节点，提供一些相关方法。

①ParentNode

如果当前节点是父节点，就会混入ParentNode接口。只有元素节点、文档节点和文档片段节点拥有这个接口。

- ParentNode.children

返回一个HTMLCollection实例，成员是当前节点的所有元素子节点。属性只读。

- ParentNode.firstElementChild

返回当前节点的第一个元素子节点，如果没有任何元素子节点，返回null。

- ParentNode.lastElementChild

返回当前节点的最后一个元素子节点，如果不存在任何元素子节点，返回null。

- ParentNode.childElementCount

返回一个整数，表示当前节点的所有元素子节点数目。如果不包含任何元素子节点，返回0.

- ParentNode.append()、ParentNode.prepend()

ParentNode.append()：为当前节点追加一个或多个子节点，位置是最后一个元素子节点后面。不仅可以添加元素子节点（参数为元素节点），还可添加文本子节点（参数为字符串）。没有返回值。

🔺该方法与Node.prototype.appendChild区别

a. append()允许字符串作为参数，appendChild()只允许子节点作为参数

b. append()没有返回值，appendChild()返回添加的子节点

c. append()可添加多个子节点和字符串（多个参数），appendChild()只能添加一个节点（一个参数）

ParentNode.prepend()：为当前节点追加一个或多个子节点，位置是第一个元素子节点前面。没有返回值。用法与append()完全一致。

```js
// HTML
// <div id="div1"><a id="text1" href="#">111</a><a href="#">23</a></div>
var div1 = document.getElementById('div1');
console.log(div1.children.length,div1.childElementCount,div1.firstElementChild);
//2 2 <a id="text1" href="#">111</a>
div1.append(document.createElement('p'));
console.log(div1.childNodes.length);//3
```

②ChildNode

如果一个节点有父节点，该节点拥有了ChildNode接口。

- ChildNode.remove()

从父节点移除当前节点。

- ChildNode.before()、ChildNode.after()

before()方法用于在当前节点前面，插入一个或多个同级节点。两者拥有相同父节点。该方法不仅可以插入元素节点，还可插入文本节点。

after()方法用于在当前节点后面，插入一个或多个同级节点。两者拥有相同父节点。与before()方法完全相同。

- ChildNode.replaceWith()

使用参数节点，替换当前节点。参数可以是元素节点，也可以是文本节点。

```js
// HTML
// <div id="div1"><a id="text1" href="#">111</a><a href="#">23</a></div>
var text1 = document.getElementById('text1');
text1.after('123',document.createElement('p'));
console.log(document.getElementById('div1').childNodes.length);//4
```

**16.4 Text节点和DocumentFragment节点**

①Text节点

文本节点代表元素节点和属性节点的文本内容。

通常使用父节点的firstChild、nextSibling等属性获取文本节点，或使用Document节点的createTextNode创造一个文本节点。

浏览器原生提供一个Text构造函数。返回一个文本节点实例。参数是该文本节点的文本内容：

```js
// 空字符串
var text1 = new Text();

// 非空字符串
var text2 = new Text('This is a text node');
```

🔺空格也是一个字符，哪怕只有一个空格，也会形成文本节点。

文本节点除了继承Node接口，还继承了CharacterData接口。

```js
// HTML
// <div id="div1"><a id="text1" href="#">111</a><a href="#">23</a></div>
var textNode = document.querySelector('a').firstChild;
console.log(textNode);//"111"
document.querySelector('div').appendChild(document.createTextNode('1234'));
console.log(document.getElementById('div1').lastChild);//"1234"
```

- Text节点的属性

data：等同于nodeValue属性，设置或读取文本节点的内容。

wholeText：将当前文本节点与毗邻的文本节点，作为一个整体返回。大多数情况，wholeText属性返回值，与data属性和textContent属性相同，但有些情况有差异：

```js
// HTML
// <p id="p">A <b>B</b> C</p>
var para = document.getElementById('p');
console.log(para.firstChild.wholeText,para.firstChild.data);//"A " "A "
para.removeChild(para.childNodes[1]);
console.log(para.firstChild.wholeText,para.firstChild.data);//"A C" "A "
```

length：返回当前文本节点文本长度

nextElementSibling、previousElementSibling

返回紧跟在当前文本节点后面（前面最近）的那个同级元素节点。如果取不到元素节点，返回null。

```js
// HTML
// <p id="p">A <b>B</b> C</p>
var para = document.getElementById('p');
console.log(para.firstChild.nextElementSibling);//<b>B</b>
```

- Text节点方法

appendData()：在Text节点尾部追加字符串

deleteData()：删除Text节点内部子字符串，第一个参数为子字符串开始位置，第二个参数为子字符串长度。

insertData()：在Text节点插入字符串，第一个参数为插入位置，第二个参数为插入的子字符串

replaceData()：替换文本，第一个参数为替换开始位置，第二个参数为需要被替换掉的长度，第三个参数为新加入的字符串。

subStringData()：获取子字符串，第一个参数为子字符串在Text节点中的开始位置，第二个参数为子字符串长度。

remove()：移除当前Text节点

splitText()：将Text节点一分为二，变成两个毗邻的Text节点，参数是分割位置（从开始），分隔到该位置字符前结束。如果分割位置不存在将报错。分割后方法返回分割位置后方字符串，原Text节点变成只包含分割位置前方的字符串。

🔺父元素的normalize方法可实现逆操作，将两个Text节点合并。

```js
// HTML
// <p>abcde</p>
var para = document.querySelector('p').firstChild;
para.appendData('ijk');//"abcdeijk"
para.deleteData(2,5);//"abk"
para.insertData(3,'zzz');//"abkzzz"
para.replaceData(4,1,'x');//"abkzxz"
var newText = para.splitText(3);
console.log(newText, para);//"zxz" "abk"
```

- DocumentFragment节点

本身就是一个完整DOM树形结构。没有父节点，parentNode返回null，但可以插入任意数量子节点。不属于当前文档。操作DocumentFragment节点比直接操作DOM树快得多。

一般用于构建一个DOM结构，然后插入当前文档。document.createDocumentFragment方法以及浏览器原生的DocumentFragment构造函数，可创建一个空的DocumentFragment节点。然后再使用其他DOM方法，向其添加子节点。

```js
var docFrag = document.createDocumentFragment();
// 等同于
var docFrag = new DocumentFragment();

var li = document.createElement('li');
li.textContent = 'Hello World';
docFrag.appendChild(li);

document.querySelector('ul').appendChild(docFrag);
```

🔺DocumentFragment节点本身不能被插入当前文档。当它作为appendChild()、insertBefore()、replaceChild()方法参数时，是它的所有子节点插入当前文档，而不是它自身。一旦DocumentFragment节点被添加进当前文档，它自身就变成了空姐点（textContent属性为空字符串），可被再次使用。如果想要保存DocumentFragment节点内容，可使用cloneNode方法：

```js
document
  .querySelector('ul')
  .appendChild(docFrag.cloneNode(true));
```

如上添加DocumentFragment节点进入当前文档，不会清空DocumentFragment节点。

```js
function reverse(n) {
  var f = document.createDocumentFragment();
  while(n.lastChild) f.appendChild(n.lastChild);
  n.appendChild(f);
}
```

如上，DocumentFragment反转一个指定节点的所有子节点顺序。

🔺DocumentFragment节点对象没有自己属性和方法，全部继承自Node节点和ParentNode接口。即DocumentFragment节点比Node节点多出以下四个属性：

children：返回一个动态HTMLCollection集合对象，包括当前DocumentFragment对象的所有子元素节点

firstElementChild：返回当前DocumentFragment对象的第一个元素子节点，如果没有返回null。

lastElementChild：返回当前DocumentFragment对象的最后一个子元素节点，如果没有返回null。

childElementCount：返回当前DocumentFragment对象的所有子元素数量。

**16.5  Document节点**

document节点对象代表整个文档，每张网页都有自己的document对象。window.document属性就指向这个对象。只要浏览器开始载入HTML文档，该对象就存在了，可直接使用。

获取document对象的方法：

- 正常网页，直接用document或window.document
- iframe标签里网页，使用iframe节点的contentDocument属性
- Ajax操作返回的文档，使用XMLHttpRequest对象的responseXML属性
- 内部结点的ownerDocument属性

document对象继承了EventTarget接口和Node接口，并且混入了ParentNode接口。

①属性

- 快捷方式属性：指向文档内部某个节点的快捷方式

document.defaultView：返回document对象所属的window对象。如果当前文档不属于window对象，返回null。

document.doctype：document对象一般有两个子节点，第一个子节点document.doctype，指向 `<DOCTYPE>`节点，即文档类型（DTD)。HTML的文档类型节点，一般写成 `<!DOCTYPE html>`。如果没有声明DTD，返回null。

document.documentElement：返回当前文档的根元素节点。通常是document节点的第二个子节点。HTML网页的该属性一般是 `<html>`节点。

document.body，document.head：分别指向 `<body>`节点和 `<head>`节点。这两个属性总是存在。是可写的，如果改写它们相当于移除所有子节点。

document.scrollingElement：返回文档的滚动元素。当文档整体滚动时，到底哪个元素在滚动。标准模式下返回 `<html>`，兼容模式下返回`<body>`元素，如果该元素不存在，返回null。

document.activeElement：返回当前焦点的DOM元素。通常是 `<input>`、`<textarea>`、`<select>`等表单元素，如果当前没有焦点元素，返回 `<body>`元素或null。

document.fullscreenElement：返回当前以全屏状态展示的DOM元素。如果不是全屏状态，返回null。

- 节点集合属性：文档内部特定元素集合。返回HTMLCollection实例。集合都是动态的，原节点任何变化，会立刻反映在集合中。

document.links：返回当前文档所有设定了href属性的 `<a>`和`<area>`节点。

document.forms：返回所有 `<form>`表单节点。除了用位置序号，id属性和name属性也可引用表单

document.images：返回页面所有 `<img>`图片节点

document.embeds，document.plugins：返回所有 `<embed>`节点

document.scripts：返回所有 `<script>`节点

document.styleSheets：返回文档内嵌或引入的样式表集合。

🔺除了document.styleSheets，以上集合属性返回都是HTMLCollection实例。都可用方括号运算符引用成员，还可用成员的id或name属性引用。

```js
// HTML
// <form name="myForm">
//    <a href="#"></a>
// </form>
console.log(document.links,document.forms);
console.log(document.forms.myForm === document.forms[0]);//true
```

- 文档静态信息属性：返回文档信息

document.documentURI，document.URL：返回字符串，表示当前文档的网址。documentURI继承自Document接口，可用于所有文档；URL继承自HTMLDocument接口，只能用于HTML文档。如果文档的锚点变化，这两个属性都会跟着变化。

document.domain：返回当前文档域名，不包含协议和端口。如 `http://www.example.com:80/hello.html`，属性返回 `www.example.com`，如无法获取域名，返回null。该属性基本只读，但有一种情况，次级域名的网页可把document.domain设为对应的上级域名。如当前域名 `a.sub.example.com`，可设为 `sub.example.com`或`example.com`。修改后，document.domain相同的两个网页可读取对方资源，比如设置的Cookie。另外，设置document.domain会导致端口被改成null。如果通过document.domain来进行通信，双方网页都必须设置这个值才可保证端口相同。

document.location：Location对象是浏览器提供原生对象，提供URL相关信息和操作方法。通过window.location和document.location属性可拿到这个对象。

document.lastModified：返回字符串，表示当前文档最后修改时间。如果页面上有JavaScript生成的内容，document.lastModified属性返回总是当前时间

document.title：返回当前文档标题。默认返回 `<title>`节点值。但该属性可写，一旦被修改返回修改后的值。

document.characterSet：返回当前文档编码，如UTF-8、ISO-8859-1。

document.referrer：返回字符串，表示当前文档的访问者来自哪里。如果无法获取来源，或者用户直接键入网址而不是从其他网页点击进入，返回一个空字符串。document.referrer值总是与HTTP头信息的Referer字段保持一致。

document.dir：返回字符串，表示文字方向。只有两个可能值：①rtl：文字从右到左，阿拉伯文。②ltr：从左到右，英语和汉语。

document.compatMode：返回浏览器处理文档的模式，可能的值为BackCompat（向后兼容模式）和CSS1Compat（严格模式）。如果网页代码第一行设置了明确DOCTYPE，都是CSS1Compat。

- 文档状态属性

document.hidden：表示当前页面是否可见。如果窗口最小化，浏览器切换了Tab，都会导致页面不可见，属性返回true。一般配合Page Visibility API使用。

document.visibilityState：返回文档可见状态。有四种可能值：①visible：页面可见。页面可能是部分可见，不是焦点窗口，前面被其他窗口部分挡住。②hidden：页面不可见。③prerender：页面处于正在渲染状态，对用户来说该页面不可见。④unloaded：页面从内存里卸载了。

🔺可用在页面加载时，防止加载某些资源；或页面不可见时停掉一些页面功能。

document.readyState：返回当前文档状态，有三种可能值：①loading：加载HTML代码阶段（尚未完成解析）。②interactive：加载外部资源阶段。③complete：加载完成。

🔺属性变化过程：①浏览器开始解析HTML文档，等于loading。②浏览器遇到HTML文档的 `<script>`元素，并且没有async或defer属性，就暂停解析，开始执行脚本，还是loading。③HTML文档解析完成，变成interactive。④浏览器等待图片、样式表、字体文件等外部资源加载完成，一旦加载完成，变为complete。

每次状态变化都会触发一个readystatechange事件。

- document.cookie：操作浏览器Cookie
- document.designMode：控制文档是否可编辑。有两个值on和off。默认off，一旦设为on，用户可编辑整个文档内容。

```js
// HTML 代码如下
// <iframe id="editor" src="about:blank"></iframe>
var editor = document.getElementById('editor');
editor.contentDocument.designMode = 'on';//变为一个所见即所得的编辑器
```

- document.currentScript：只用在 `<script>`元素的内嵌脚本或加载的外部脚本中，返回当前脚本所在的DOM节点，即 `<script>`元素的DOM节点。
- document.implementation：返回一个DOMImplementation对象。对象有三个方法，用于创建独立于当前文档的新的Document对象：①DOMImplementation.createDocument()：创建一个XML文档。②DOMImplementation.createHTMLDocument()：创建一个HTML文档。③DOMImplementation.createDocumentType()：创建一个DocumentType对象。

②方法

- document.open()、document.close()：open方法清除当前文档所有内容，使得文档处于可写状态，供write方法写入内容，close方法关闭打开的文档。
- document.write()、document.writeIn()：向文档写入内容。在网页首次渲染阶段，只要页面没有关闭写入，写入的内容就会追加在已有内容的后面。该方法会将HTML代码解析，不会转义。如果页面解析完成（DOMContentLoaded事件发生后），再调用write方法，会先调用open方法擦除当前文档所有内容然后再写入。如果也缪按渲染过程中调用write方法，并不会自动调用open方法（open方法已调用，close方法还未调用）。writeIn方法会在输出内容尾部添加换行符。

🔺write方法不建议使用。writeIn方法添加的是ASCII码的换行符，渲染成HTML网页时不起作用，即在网页上显示不出换行。

```js
document.open();
document.write("123");//页面显示123
document.close();
```

- document.querySelector()、document.querySelectorAll()：querySelector方法接受一个CSS选择器作为参数，返回匹配该选择器的元素节点。如有多个节点满足条件则返回第一个匹配的节点。如果没有匹配节点就返回null。querySelectorAll方法返回NodeList对象，包含所有匹配给定选择器节点。

🔺①参数可以是逗号分隔的多个CSS选择器。②不支持CSS伪元素选择器（如 :first-line和:first-letter）和伪类选择器（如:link和:visited），即无法选中伪元素和伪类。③如果querySelectorAll方法参数是字符串*，返回文档中所有元素节点。该方法返回结果不是动态集合，不会实时反映元素节点变化。④这两个方法除了定义在document对象上，还定义在元素节点上。

```js
// HTML
// <p id="p">123</p>
// <div class="div">1234</div>
console.log(document.querySelector('.div'));
console.log(document.querySelector('#p'));
console.log(document.querySelectorAll('*'));
```

- document.getElementsByTagName()：搜索HTML标签名，返回符合条件元素。返回值是HTMLCollection实例，实时反映HTML文档变化。如没有任何匹配元素，返回一个空集。

🔺①如果传入字符串*，返回文档所有HTML元素。②元素节点本身也定义了getElementsByTagName方法。③该方法参数大小写不敏感。

```js
// HTML
// <div>1234<p>111</p></div>
console.log(document.getElementsByTagName('div')[0].getElementsByTagName('p'));
```

- document.getElementsByClassName()：返回HTMLCollection实例，包含所有class名符合条件的元素。元素变化实时反映在返回结果中。

🔺①参数可是多个class，之间用空格分隔。②可在元素节点调用。

```js
// HTML
// <div class="div a">1234<p class="p">111</p></div>
console.log(document.getElementsByClassName('div a')[0].getElementsByClassName('p')[0]);
```

- document.getElementsByName()：选择拥有name属性的HTML元素（如 `<form>`,`<img>`,`<radio>`,`<frame>`等），返回NodeList实例

```js
// HTML
// <form name="my"><radio name="my"></radio></form>
console.log(document.getElementsByName('my').length);//2
```

- document.getElementById()：返回匹配指定id属性元素节点。如没有发现匹配节点，返回null。

🔺①参数大小写敏感。②只能在document对象上用。

```js
// HTML
// <div id="div">11</div>
console.log(document.getElementById('div') === document.querySelector('#div'));//true
```

- document.elementFromPoint()、document.elementsFromPoint()：返回位于页面指定位置最上层的元素节点（返回数组，成员是位于指定坐标的所有元素）。两个参数依次是相对于当前视口左上角的横坐标和纵坐标，单位是像素。如果该位置HTML元素不可返回，则返回它的父元素。如果坐标值无意义，返回null。
- document.createElement()：生成元素节点并返回该节点。参数为元素标签名，可以是自定义标签名。

- document.createTextNode()：生成文本节点（Text实例）并返回该节点。参数是文本节点内容。该方法可确保返回的节点被浏览器当作文本渲染，而不是当作HTML代码渲染。因此可用来栈实用户输入，避免XSS攻击。该方法不对单引号和双引号转义，不能用来对HTML属性赋值。

```js
var newDiv = document.createElement('div');
var text = document.createTextNode('234');
newDiv.appendChild(text);
document.body.appendChild(newDiv);//页面展示234
```

- document.createAttribute()：生成一个新的属性节点（Attr实例）并返回它。

```js
var div = document.getElementById('myDiv');
var a = document.createAttribute('myAttr');
a.value = 'value';
div.setAttributeNode(a);
// 等同于 div.setAttribute('myAttr','value');
```

- document.createComment()：生成一个新的注释节点并返回该节点。参数是字符串，称为注释节点的内容。
- document.createDocumentFragment()：生成一个空的文档片段对象（DocumentFragment实例）。

```js
var doc = document.createDocumentFragment();
doc.appendChild(document.createTextNode('111'));
document.getElementById('myDiv').appendChild(doc);//显示111
```

- document.createEvent()：生成一个事件对象（Event实例），该对象可被element.dispatchEvent方法使用，触发指定事件。参数是事件类型，如UIEvents、MouseEvents、HTMLEvents等

- document.addEventListener()，document.removeEventListener()，document.dispatchEvent()：用于处理document节点事件，都继承自EventTarget接口。

```js
// 添加事件监听函数
document.addEventListener('click', listener, false);

// 移除事件监听函数
document.removeEventListener('click', listener, false);

// 触发事件
var event = new Event('click');
document.dispatchEvent(event);
```

- document.hasFocus()：返回布尔值，表示当前文档中是否有被激活或获得交点。有焦点的文档必定被激活，激活的文档不一定有焦点。如用户点击按钮，从当前窗口跳出新窗口，该新窗口就是激活的，但不拥有焦点。
- document.adoptNode()，document.importNode()：①adoptNode方法将某个节点及其子节点，从原来所在的文档或DocumentFragment里移除，归属当前document对象，返回插入后的新节点。插入的节点对象的ownerDocument属性，变成当前document对象，而parentNode是null。还要用appendChild或insertBefore方法将新节点插入当前文档树。②importNode方法是从原来所在的文档或DocumentFragment里拷贝某个节点或子节点，让它们归属当前document对象。拷贝的节点对象的ownerDocument属性变成当前document对象，parentNode是null。第一个参数是外部节点，第二个参数是一个布尔值，表示对外部节点是深拷贝还是浅拷贝，默认浅拷贝（false）。建议设为true。还要把节点插入当前文档树。

```js
var node1 = document.adoptNode(externalNode1);
document.appendChild(node1);

var node2 = document.importNode(externalNode2, deep);
```

🔺importNode应用：从iframe窗口拷贝一个指定节点，插入当前文档。

- document.createNodeIterator()：返回子节点遍历器。第一个参数为所要遍历的根节点，第二个参数为所要遍历节点类型。节点类型主要有：①所有节点：NodeFilter.SHOW_ALL。②元素节点：NodeFilter.SHOW_ELEMENT。③文本节点：NodeFilter.SHOW_TEXT。④注释节点：NodeFilter.SHOW_COMMENT。该方法返回NodeFilter实例。该实例nextNode方法和previousNode方法可用来遍历所有子节点。nextNode方法将根节点的所有子节点依次读入一个数组。nextNode方法会先返回遍历器内部指针所在节点，然后指针移向下一节点。所有成员遍历完后返回null。previousNode方法先将指针移向上一个节点，然后返回该节点。

```js
var nodeIterator = document.createNodeIterator(document.body);
var node = [];
var curr;
while (curr = nodeIterator.nextNode()) {
    node.push(curr);
}
```

🔺遍历器返回的第一个节点，总是根节点。

- document.createTreeWalker()：返回一个DOM的子树遍历器。与createNodeIterator方法类似，区别在于它返回TreeWalker实例。它的第一个节点不是根节点。第一个参数是所要遍历的根节点，第二个参数指定所要遍历的节点类型（与createNodeIterator方法第二参数相同）。

```js
var treeWalker = document.createTreeWalker(document.body);
var node = [];
while (treeWalker.nextNode()) {
    node.push(treeWalker.currentNode);
}
console.log(node);
```

- document.execCommand()，document.queryCommandSupported()，document.queryCommandEnabled()：①execCommand：如果document.designMode属性设为on，整个文档用户可编辑；如果元素contenteditable属性设为true，该元素可编辑。这两种情况下，可使用该方法改变内容样式。

```js
document.execCommand(command, showDefaultUI, input)
```

三个参数：command为字符串，表示所要实施的样式。showDefaultUI为布尔值，表示是否使用默认的用户界面，建议设为false。input为字符串，表示该样式辅助内容，如生成超链接时，这个参数就是所要链接的网址。如果第二个参数为true，浏览器弹出提示框，要求用户在提示框输入该参数。

该方法返回布尔值，如果为false，表示这个方法无法生效。大部分情况，该方法只对选中内容生效，如有多个内容可编辑区域，只对当前焦点所在元素生效。

第一个参数可以是bold，selectAll，createLink，insertText，unlink，redo等。

②queryCommandSupported：返回布尔值，表示浏览器是否支持document.execCommand()的某个命令。

③queryCommandEnabled：返回布尔值，表示当前是否可用document.execCommand()某个命令。

```js
// HTML
// <input type="button" value="Copy" onclick="doCopy()">
function doCopy(){
    // 浏览器是否支持 copy 命令（选中内容复制到剪贴板）
    if (document.queryCommandSupported('copy')) {
        copyText('你好');
    }else{
        console.log('浏览器不支持');
    }
}

function copyText(text) {
    var input = document.createElement('textarea');
    document.body.appendChild(input);
    input.value = text;
    input.focus();
    input.select();

    // 当前是否有选中文字
    if (document.queryCommandEnabled('copy')) {
        var success = document.execCommand('copy');
        input.remove();
        console.log('Copy Ok');
    } else {
        console.log('queryCommandEnabled is false');
    }
}
```

- document.getSelection()：指向window.getSelection()

**16.6 Element节点**

Element节点对象对应网页HTML元素。Element对象继承了Node接口。

不同HTML元素对应的元素节点不一样，如 `<a>`元素构造函数时HTMLAnchorElement()，`<button>`是HTMLButtonElement()。元素节点不是一种对象，而是许多种对象。

①属性

- 元素特性相关属性

Element.id：返回指定元素id属性，可读写。id属性大小写敏感。

Element.tagName：返回指定元素大写标签名，与nodeName属性值相等。

Element.dir：用于读写当前元素文字方向，可能是从左到右ltr或从右到左rtl。

Element.accessKey：读写分配给当前元素的快捷键

Element.draggable：返回布尔值，表示当前元素是否可拖动，可读写。

Element.lang：返回当前元素语言设置，可读写

Element.tabIndex：返回整数，表示当前元素在Tab键遍历时的顺序，可读写。

🔺tabIndex属性值如为负数，Tab键不会遍历到该元素。如为正整数，按顺序从小到大遍历，如果两个元素tabIndex值相等，按出现顺序遍历。遍历完所有tabIndex为正整数元素后，再遍历所有tabIndex等于0或是非法值或没有tabIndex属性的元素，顺序为它们在网页中出现顺序。

Element.title：读写当前元素的HTML属性title，通常用来指定鼠标悬浮时弹出的文字提示框。

- 元素状态相关属性

Element.hidden：返回布尔值，表示当前元素hidden属性，控制当前元素是否可见，可读写。CSS的设置高于此属性。如果CSS指定了该元素不可见（display：none）或可见（display：hidden），那么该属性不能改变该元素实际可见性。即该属性不能判断元素实际可见性，只有在CSS没明确设定当前元素可见性时才有效。

Element.contentEditable，Element.isContentEditable：HTML元素可设置contenteditable属性，使得元素内容可编辑。contentEditable属性返回字符串，表示是否设置了contenteditable属性，可写，三种可能值：①“true”：元素内容可编辑。②“false”：元素内容不可编辑。③“inherit”：元素是否可编辑，继承了父元素设置。isContentEditable属性返回布尔值，表示是否设置了contentEditable属性，只读。

```html
<!--页面此区域可编辑-->
<div id="div" class="test" contenteditable="true">123</div>
```

- Element.attributes

返回类似数组的对象，成员是当前元素节点的所有属性节点。属性节点对象有name属性和value属性，对应该属性的属性名和属性值，等同于nodeName属性和nodeValue属性。

🔺HTML元素的属性名大小写不敏感，但JavaScript对象属性名大小写敏感。转换规则：①一律采用小写。②属性名包含多个单词时驼峰拼写。③有些HTML属性名是JavaScript保留字，for属性改为htmlFor，class属性改为className。④HTML属性值一般都是字符串，JavaScript属性会自动转换类型。

```js
// HTML
// <div id="div" class="test"></div>
var p = document.getElementById('div');
console.log(p.attributes);
console.log(p.attributes[0].name,p.attributes[0].nodeName);//"id" "id"
console.log(p.attributes[1].value,p.attributes[1].nodeValue);//"test" "test"
```

- Element.className，Element.classList

className用来读写当前元素节点class属性。值是字符串，每个class之间空格分割。

classList返回一个类似数组的对象，当前元素节点的每个class就是这个对象的一个成员。

🔺classList有如下方法：add()：增加一个class。remove()：移除一个class。contains()：检查当前元素是否包含某个class。toggle()：将某个class移入或移出当前元素。可添加第二个参数，如果为true则添加属性，为false则去除属性。item()：返回指定索引位置的class。toString()：将class的列表转为字符串。

```js
// HTML
// <div class="one two three">111</div>
var div = document.querySelector('div');
console.log(div.className);//"one two three"
console.log(div.classList.item(1));//"two"
div.classList.toggle('four',true);
console.log(div.classList[div.classList.length-1]);//"four"
```

- Element.dataset

网页元素可自定义data-属性，用来添加数据。该属性返回一个对象，可从这个对象读写data-属性。dataset各个属性返回都是字符串。data-属性名，只能包含英文字母、数字、连词线（-）、点（.）、冒号和下划线。转化规则：①开头的data-属性会省略。②如果连词线后跟了一个英文字母，连词线取消，字母变大写。③其他字符不变。

删除一个data-属性，可用delete命令。

🔺也可通过Element.getAttribute()和Element.setAttribute()读写这些属性。

```js
// HTML
// <div data-abc-1="2" data-abd-def="data">111</div>
var div = document.querySelector('div');
console.log(div.dataset.abdDef,div.dataset["abc-1"]);//"data" "2"
div.dataset.a = 'b';
console.log(div.getAttribute('data-a'));//"b"
```

- Element.innerHTML

返回字符串，等同于该元素包含的所有HTML代码，可读写，常用来设置某个节点内容。能改写所有元素节点内容。如果将该属性设为空，等于删除所有它包含的所有节点。

🔺①读取属性值，如果文本节点包含&、小于号和大于号，属性会将它们转为实体性式&amp、&lt、&gt。要得到原文，建议用element.textContent属性。②如果插入的文本包含HTML标签，会被解析成为节点对象插入DOM。如果文本中含 `<script>`标签，虽可生成script节点，但插入代码不会执行。③为了安全考虑，如果插入的是文本，最好用textContent代替innerHTML。

```js
// HTML
// <div data-abc-1="2" data-abd-def="data">2>1</div>
var div = document.querySelector('div');
console.log(div.innerHTML,div.textContent);//2&gt;1 2>1
div.textContent = '111';//页面显示111
```

- Element.outerHTML

返回字符串，表示当前元素节点的所有HTML代码，包括该元素本身和所有子元素。可读写。

🔺如果一个节点没有父节点，设置outerHTML属性会报错。

```js
// HTML
// <div id="div"><div id="div2"><p>2</p></div></div>
var div = document.getElementById('div2');
console.log(div.outerHTML);//<div id="div2"><p>2</p></div>
```

- Element.clientWidth，Element.clientHeight

clientHeight返回整数值，表示元素节点CSS高度（单位像素），只对块级元素生效，对行内元素返回0.如块级元素没有设置CSS高度，返回实际高度。除了元素本身高度，还包括padding部分，但不包括border、margin。如有水平滚动条，还减去水平滚动条高度。

clientWidth返回元素节点CSS宽度，只对块级元素有效，只包括元素本身宽度和padding，如有垂直滚动条，还减去垂直滚动条宽度。

🔺document.documentElement的clientHeight属性，返回当前视口的高度（浏览器窗口高度），等同于window.innerHeight属性减去水平滚动条高度（如有）。document.body高度则是网页实际高度。一般document.body.clientHeight大于document.documentElement.clientHeight

- Element.clientLeft，Element.clientTop

clientLeft属性等于元素节点左边框的宽度（单位像素），不包括左侧的padding和margin。如没有设置左边框，或者是行内元素（display：inline），属性返回0。返回整数值。

clientTop属性等于网页元素顶部边框的宽度（单位像素），其他特点与clientLeft同。

- Element.scrollHeight，Element.scrollWidth

scrollHeight返回整数值，表示当前元素总高度（单位像素），包括溢出容器，当前不可兼得部分。包括padding，但不包括border、margin以及水平滚动条高度。还包括伪元素(::before或::after)的高度。只读。

scrollWidth表示当前元素总宽度（单位像素），其他地方都与scrollHeight类似。只读。

🔺①整张网页总高度可从document.documentElement或document.body上读取：

```js
// 返回网页的总高度
document.documentElement.scrollHeight
document.body.scrollHeight
```

②如果元素节点内容溢出，即使溢出内容隐藏，scrollHeight仍然返回元素总高度。

- Element.scrollLeft，Element.scrollTop

scrollLeft表示当前元素水平滚动条右侧滚动的像素数量，scrollTop表示当前元素垂直滚动条向下滚动像素数量。对于没有滚动条网页元素，返回0。这两个属性都可读写，设置值之后导致浏览器将当前元素自动滚动到相应位置。

🔺如要查看整张网页水平和垂直滚动距离：

```js
document.documentElement.scrollLeft
document.documentElement.scrollTop
```

```js
// HTML
// <div id="div" class="test">123</div>
var div = document.getElementById('div');
console.log(div.scrollHeight,div.scrollWidth,div.scrollLeft,div.scrollTop);//21 736 0 0
```

- Element.offsetParent

返回最靠近当前元素的，并且CSS的position属性不等于static的上层元素。主要用于确定子元素位置偏移的计算基准。如果该元素不可见（display为none）或位置是固定的（position属性为fixed），属性返回null。

🔺如果某个元素所有上层节点的position属性都是static，则该属性指向 `<body>`元素。

```html
<div style="position: absolute;">
  <p>
    <span>Hello</span>
  </p>
</div>
```

如上，span元素的offsetParent属性是div元素

- Element.offsetHeight，Element.offsetWidth

offsetHeight返回整数，表示元素的CSS垂直高度（单位像素），包括元素本身高度，padding和border，以及水平滚动条高度（如有）。只读。

offsetWidth表示元素CSS水平宽度（单位像素），其他与offsetHeight一致。只读。

🔺只比Element.clientHeight和Element.clientWidth多了边框高度或宽度。如果元素CSS设为不可见（display为none），返回0。

- Element.offsetLeft，Element.offsetTop

offsetLeft返回当前元素左上角相对于offsetParent节点的水平位移。

offsetTop返回垂直位移，单位为像素。

通常这两个值指相对于父节点的位移。

⭐八个类型数值实例比较：

```js
// HTML
// <div id="div" class="test">123<div id="div2">123</div></div>
var div2 = document.getElementById('div2');
console.log(div2.clientHeight,div2.clientWidth,div2.clientLeft,div2.clientTop);//21 736 0 0
console.log(div2.offsetHeight,div2.offsetWidth,div2.offsetLeft,div2.offsetTop);//21 736 8 29
```

🔺元素左上角相对于整张网页坐标：

```js
function getElementPosition(e) {
  var x = 0;
  var y = 0;
  while (e !== null)  {
    x += e.offsetLeft;
    y += e.offsetTop;
    e = e.offsetParent;
  }
  return {x: x, y: y};
}
```

- Element.style

每个元素节点都有style用来读写该元素行内样式信息

- Element.children，Element,childElementCount

children属性返回一个类似数组的对象（HTMLCollection实例），包括当前元素节点的所有子元素。如果当前元素没有子元素，返回的对象包含零个成员。

🔺与Node.childNodes区别是，它只包括元素类型子节点，不包括其他类型子节点。

childElementCount返回当前元素节点包含的子元素节点个数，与Element.children.length相同

- Element.firstElementChild，Element.lastElementChild

返回当前元素的第一个（最后一个）元素子节点。如没有元素子节点，返回null。

- Element.nextElementSiblng，Element.previousElementSibing

返回当前元素节点的后一个（前一个）同级元素节点，如没有则返回null。

```js
// HTML
// <div id="div"><div id="div2"></div><p>2</p></div>
var div = document.getElementById('div');
console.log(div.childElementCount,div.firstElementChild,div.lastElementChild);
//2 <div id="div2"></div> <p>2</p>
```

②方法

- 属性相关方法

getAttribute()：读取某个属性值。返回字符串。如果某个属性不存在，则返回null。

getAttributeNames()：返回一个数组，表示当前元素所有属性名。如当前元素没任何属性，则返回一个空数组。使用Element.attributes属性同样得到结果，唯一区别是它返回类似数组的对象。

setAttribute()：写入属性值。属性值总是字符串，其他类型值会被转成字符串。如果同名属性已存在，则相当于编辑已存在属性。没有返回值。

hasAttribute()：某个属性是否存在

hasAttributes()：当前元素是否有属性

removeAttribute()：删除属性，没有返回值。

```js
// HTML
// <div id="div" class="test"></div>
var p = document.getElementById('div');
p.setAttribute('user','a');
console.log(p.hasAttribute('user'),p.getAttribute('user'));//true "a"
```

- Element.querySelector()

接受CSS选择器作参数，返回父元素的第一个匹配的子元素，如没有找到匹配子元素，返回null。可接受任何复杂的CSS选择器：

```js
document.body.querySelector("style[type='text/css'], style:not([type])");
```

可接受多个选择器，之间逗号分隔：

```js
element.querySelector('div, p')//返回element的第一个div或p子元素
```

🔺①无法选中伪元素。②该方法现在全局范围搜索给定的CSS选择器，然后过滤哪些属于当前元素的子元素。

- Element.querySelectorAll()

接受CSS选择器作参数，返回NodeList实例，包含所有匹配子元素。可接受多个CSS选择器，之间逗号分隔。

🔺①如果选择器含伪元素选择器，返回空的NodeList实例。②先在全局范围内查找，再过滤出当前元素的子元素。选择器实际上针对整个文档。

```js
// HTML
// <div id="div"><div id="div2"></div><p>2</p></div>
var div = document.getElementById('div');
console.log(div.querySelector('div'));//<div id="div2"></div>
console.log(div.querySelectorAll('div,p').length);//2
```

- Element.getElementsByClassName()

返回HTMLCollection实例，成员是当前元素节点的所有具有指定class的子元素节点。参数大小写敏感。与document.getElementsByClassName类似，只是搜索范围不是整个文档，而是当前元素节点。HTMLCollection实例是一个活的集合。

- Element.getElementsByTagName()

返回HTMLCollection实例，成员是当前元素节点的所有匹配指定标签名的子元素节点。大小写不敏感。搜索范围不是整个文档而是当前元素节点。

```js
// HTML
<div id="div" class="test"><div id="div2" class="test"></div><p>2</p></div>
var div = document.getElementById('div');
console.log(div.getElementsByTagName('DIV'));//div2
console.log(div.getElementsByClassName('test'));//div2
```

- Element.closest()

接受CSS选择器作参数，返回匹配该选择器的、最接近当前节点的一个祖先节点（包括当前节点本身）。如没有任何节点匹配CSS选择器，返回null。

```js
// HTML
// <div id="div" class="test"><div id="div2" class="test"><p id="p" class="test">2</p></div></div>
var p = document.getElementById('p');
console.log(p.closest('.test'));//<p id="p" class="test">2</p>
console.log(p.closest('div > div'));//<div id="div2" class="test"><p id="p" class="test">2</p></div>
```

- Element.matches()

表示当前元素是否匹配给定的CSS选择器

- 事件相关

继承自EventTarget接口

Element.addEventListener()：添加事件回调函数

Element.removeEventListener()：移除事件监听函数

Element.dispatchEvent()：触发事件

- Element.scrollIntoView()

滚动当前元素，进入浏览器可见区域。类似于设置window.location.hash效果。

可接受布尔值作为参数，如为true，表示元素顶部与当前区域可见部分顶部对齐（前提是当前区域可滚动）；如为false，表示元素底部与当前区域可见部分尾部对齐（前提是当前区域可滚动）。如没有参数，默认true。

- Element.getBoundingClientRect()

返回一个对象，提供当前元素节点的大小、位置等信息，基本就是CSS盒状模型的所有信息。返回对象具有如下属性（全部只读）：

x：元素左上角相对于视口的横坐标

y：元素左上角相对于视口的纵坐标

height：元素高度

width：元素宽度

left：元素左上角相对于视口的横坐标，与x属性相等

right：元素右边界相对于视口的横坐标（等于x+width）

top：元素顶部相对于视口的纵坐标，与y属性相等

bottom：元素底部相对于视口的纵坐标（等于y+height）

🔺①元素相对于视口位置会随着页面滚动变化。表示位置的四个属性值，都不是固定不变。如想要得到绝对位置，可将left属性加上window.scrollX，top属性加上window.scrollY。②该对象属性都把边框算作元素一部分，即width和height包括了元素本身+padding+border。③这些属性都是继承自原型属性，Object.keys()返回空数组。

```js
// HTML
// <div id="div" class="test">123</div>
var div1 = document.getElementById('div');
console.log(div1.getBoundingClientRect());
// bottom: 29.090909957885742
// height: 21.090909957885742
// left: 8
// right: 744
// top: 8
// width: 736
// x: 8
// y: 8
```

- Element.getClientRects()

返回一个类似数组的对象，里面是当前元素在页面上形成的所有矩形。每个矩形都有bottom、height、left、right、top和width六个属性，表示它们相对于视口的四个坐标以及本身高度宽度。

对于盒状元素（如 `<div>`和`<p>），返回的对象中只有该元素一个成员。对于行内元素（如` `<span>`,`<a>`)，返回的对象有多少成员取决于该元素在页面上占据多少行。和Element.getBoundingClientRect()最大区别在于后者对于行内元素总是返回一个矩形。

🔺①该方法主要用于判断行内元素是否换行，以及行内元素每一行的位置偏移。②如果行内元素包含换行符，该方法会把换行符考虑在内。

- Element.insertAdjacentElement()

在相对于当前元素指定位置插入一个新节点。返回被插入的节点。如果插入失败，返回null。接受两个参数，第一个参数是字符串，表示插入位置，第二个参数是将要插入的节点。第一个参数可能取值：①beforebegin：当前元素之前。②afterbegin：当前元素内部的第一个子节点前。③beforeend：当前元素内部最后一个子节点后。④afterend：当前元素后。

🔺①beforebegin和afterend，只在当前节点有父节点时才生效。如当前节点由脚本创建，没有父节点，插入会失败。②如插入的节点是一个文档里现有节点，它会从原有位置删除，放置到新位置。

```js
// HTML
// <div id="div" class="test">123</div>
var div1 = document.getElementById('div');
div1.insertAdjacentElement("afterend",document.createElement('p'));
console.log(div1.nextElementSibling);//<p></p>
```

- Element.insertAdjacentHTML()，Element.insertAdjacentText()

insertAdjacentHTML方法用于将一个HTML字符串，解析生成DOM结构，插入相对于当前节点指定位置。接受两个参数，第一个参数和insertAdjacentElement()类似，第二个参数是待解析的HTML字符串。

🔺①只是在现有DOM结构里插入节点，执行速度比innerHTML快得多。②不会转义HTML字符串，导致不能用来插入用户输入的内容。

insertAdjacentText方法在相对于当前节点指定位置，插入一个文本节点，用法与insertAdjacentHTML()完全一致。

- Element.remove()

继承自ChildNode接口，将当前元素节点从它的父节点移除。

- Element.focus()，Element.blur()

focus方法用于将当前页面焦点，转移到指定元素。可接受对象作参数，参数对象的preventScroll属性是布尔值，指定是否将当前元素停留在原始位置，而不是滚动到可见区域。

blur方法用于将焦点从当前元素移除。

- Element.click()

在当前元素上模拟一次鼠标点击，相当于触发click事件。

**16.7 CSS操作**

- HTML元素的style属性

可使用getAttribute()、setAttribute()、removeAttribute()方法，直接读写或删除网页元素style属性：

```js
div.setAttribute(
  'style',
  'background-color:red;' + 'border:1px solid black;'
);
```

也可直接读写个别属性：

```js
e.style.color = 'black';
```

- CSSStyleDeclaration接口

三个地方部署了这个接口：①元素节点的style属性。②CSSStyle实例的style属性。③window.getComputedStyle()的返回值。

可直接读写CSS样式属性，不过连词号需要变成驼峰拼写：

```js
// HTML
// <div id="div" class="test">123</div>
var divStyle = document.getElementById('div').style;
divStyle.backgroundColor = 'red';
divStyle.width = '100px';
```

属性值都是字符串。Element.style返回的只是行内样式，并不是该元素全部样式。通过样式表设置的样式或从父元素继承的样式无法通过这个属性得到。元素全部样式通过window.getComputedStyle()得到。

👉CSSStyleDeclaration实例属性：

CSSStyleDeclaration.cssText：读写当前规则的所有样式声明文本。属性值不用改写CSS属性名。把该属性设为空字符串可删除一个元素所有行内样式。

CSSStyleDeclaration.length：返回整数值，表示当前规则包含多少条样式声明。

CSSStyleDeclaration.parentRule：返回当前规则所属的那个样式块（CSSRule实例），如不存在所属的样式块，返回null。只读，且只在使用CSSRule接口是才有意义。

👉CSSStyleDeclaration实例方法

CSSStyleDeclaration.getPropertyPriority()：接受CSS样式的属性名作参数，返回字符串，表示有没有设置important优先级。如有设置就返回important，否则返回空字符串。

CSSStyleDeclaration.getPropertyValue()：接受CSS样式属性名作参数，返回字符串，表示属性值。

CSSStyleDeclaration.item()：接受整数值作为参数，返回该位置CSS属性名。如没有提供参数，方法报错；如参数值不合法，返回空字符串。

CSSStyleDeclaration.removeProperty()：接受属性名作参数，在CSS规则里移除这个属性，返回这个属性原来的值。

CSSStyleDeclaration.setProperty()：设置新的CSS属性，没有返回值。

三个参数：①第一个参数：属性名，必需。②属性值，可选，默认空字符串。③优先级，可选，如设置，唯一合法值是important，表示CSS规则里的 !important

```js
// <div id="div" style="background-color: black!important;width: 200px">123</div>
var divStyle = document.getElementById('div').style;
console.log(divStyle.length,divStyle.getPropertyValue('background-color'))//2 "black"
console.log(divStyle.item(1));//"width"
console.log(divStyle.getPropertyPriority('background-color'));//"important"
```

- CSS模块的侦测

需要置到当前浏览器是否支持某个模块。

一个普遍方法是：判断元素style对象的某个属性值是否为字符串。如果属性不存在，则返回undefined。

🔺考虑不同浏览器的侦测方法：

```js
function isPropertySupported(property) {
  if (property in document.body.style) return true;
  var prefixes = ['Moz', 'Webkit', 'O', 'ms', 'Khtml'];
  var prefProperty = property.charAt(0).toUpperCase() + property.substr(1);

  for(var i = 0; i < prefixes.length; i++){
    if((prefixes[i] + prefProperty) in document.body.style) return true;
  }

  return false;
}
```

- CSS对象

两个静态方法：

CSS.escape()：转义CSS选择器里特殊字符。

```js
// <div id="foo#bar">
document.querySelector('#' + CSS.escape('foo#bar'))
```

CSS.supports()：返回布尔值，表示当前环境是否支持某一句CSS规则。

参数两种写法：①第一个参数属性名，第二个参数属性值。②整个参数是一行完整CSS语句。

```js
// 第一种写法
CSS.supports('transform-origin', '5px') // true

// 第二种写法，参数结尾不能带有分号
CSS.supports('display: table-cell') // true
```

- window.getComputedStyle()

行内样式具有最高优先级，改变行内样式，通常会立刻反映出来。但如果想得到元素实际样式，只读取行内样式是不够的，需要得到浏览器最终计算出来的样式规则。

接受一个节点对象作为参数，返回CSSStyleDeclaration实例，包含了指定节点CSS规则叠加后的结果。CSSStyleDeclaration实例是个活的对象，任何对于样式修改会实时反映到实例上。该实例只读。

该方法可接受第二个参数，表示当前元素的伪元素（如 :before，:after，:first-line，:first-letter等）

🔺①CSSStyleDeclaration实例返回的CSS值是绝对单位。如长度都是像素单位（返回值包含px后缀），颜色是rgb(#,#,#)或rgba(#,#,#,#)。②CSS规则简写形式无效。如margin属性值不能直接读，只能读marginLeft，marginTop等属性。③如果读取CSS原始属性名，要用方括号运算符，如obj['z-index']，如果读取驼峰拼写的，可直接读取obj.zIndex。④返回的实例的cssText属性无效，返回undefined。

- CSS伪元素

CSS伪元素是通过CSS向DOM添加的元素，主要通过:before和:after选择器生成。然后用content属性指定伪元素内容。

```css
<div id="test">Test content</div>

#test:before {
  content: 'Before ';
  color: #FF0;
}
```

🔺节点元素style对象无法读写伪元素样式，可用window.getComputedStyle()获取伪元素。

- StyleSheet接口

代表网页的一张样式表，包括 `<link>`元素加载的样式表和`<style>`元素内嵌的样式表。

document对象的styleSheets属性可返回当前页面所有StyleSheet实例（即所有样式表）。是个类似数组的对象。

如果是`<style>`元素嵌入的样式表，还有另一种获取StyleSheet实例方法，就是这个节点元素sheet属性。

🔺StyleSheet接口不仅包括网页样式表，还包括XML文档样式表。所以它有一个子类CSSStyleSheet表示网页CSS样式表。

①实例属性

StyleSheet.disabled：返回布尔值，表示该样式是否处于禁用状态。disabled属性只能在JavaScript脚本设置，不能在HTML语句中设置。手动设置该属性为true，等同于在`<link>`元素里将这张样式表设为alternate stylesheet，即样式表不会生效。

StyleSheet.href：返回样式表网址。对于内嵌样式表，该属性返回null。只读。

StyleSheet.media：返回类似数组的对象（MediaList实例），成员是表示适用媒介的字符串。表示当前样式表用于屏幕screen，还是用于打印print，或手持设备handheld，或各种媒介都适用all。只读，默认screen。MediaList实例的appendMedium方法和deleteMedium方法用于增删媒介。

StyleSheet.title：返回样式表title属性。

StyleSheet.type：返回样式表type属性，通常是text/css

StyleSheet.parentStyleSheet：CSS的@import命令匀速在样式表中加载其他样式表。返回包含了当前样式表的那张样式表。如果当前样式表是顶层样式表，返回null。

StyleSheet.ownerNode：返回StyleSheet对象所在的DOM节点，通常是 `<link>`或`<style>`。对于那些由其他样式表引用的样式表，返回null。

CSSStyleSheet.cssRules：指向一个类似数组的对象（CSSRuleList实例），每一个成员就是当前样式表的一条CSS规则。使用该规则的cssText属性可得到CSS规则对应字符串。每条CSS规则还有一个style属性，用来读写具体CSS命令。

CSSStyleSheet.ownerRule：有些样式表通过@import规则输入，它的ownerRule属性返回一个CSSRule实例，代表那行@import规则。如果当前样式表不是通过@import引入，返回null。

②实例方法

CSSStyleSheet.insertRule()：在当前样式表插入一个新的CSS规则。

接受两个参数，第一个参数是表示CSS规则的字符串，只能有一条规则，否则报错。第二个参数是该规则在样式表的插入位置（从0开始），参数可选，默认为0（默认插在样式表头部），如果插入位置大于现有规则数目就报错。返回值是新插入规则的位置序号。

CSSStyleSheet.deleteRule()：在样式表里移除一条规则，参数是该条规则在cssRules对象中的位置，没有返回值。

- 添加样式表实例：

一种方法是添加一张内置样式表，即文档中添加一个`<style>`节点。另一种方法是添加外部样式表，在文档中添加一个`<link>`节点，然后将href属性指向外部样式表URL。

```js
// 写法一
var style = document.createElement('style');
style.setAttribute('media', 'screen');
style.innerHTML = 'body{color:red}';
document.head.appendChild(style);

// 写法二
var style = (function () {
  var style = document.createElement('style');
  document.head.appendChild(style);
  return style;
})();
style.sheet.insertRule('.foo{color:red;}', 0);
```

```js
var linkElm = document.createElement('link');
linkElm.setAttribute('rel', 'stylesheet');
linkElm.setAttribute('type', 'text/css');
linkElm.setAttribute('href', 'reset-min.css');

document.head.appendChild(linkElm);
```

- CSSRuleList接口

类似数组的对象，表示一组CSS谷子额，成员都是CSSRule实例。

获取CSSRuleList实例，一般通过StyleSheet.cssRules属性。

CSSRuleList实例里，每一条规则（CSSRule实例）可通过rules.item(index)或rules[index]拿到。

添加和删除规则不能在CSSRuleList实例操作，而要在它的父元素StyleSheet实例上。

- CSSRule接口

一条CSS规则包括两个部分：CSS选择器和样式声明。

👉CSSRule实例属性

CSSRule.cssText：返回当前规则文本，如果规则是加载(@import)其他样式表，返回@import 'url'。

CSSRule.parentStyleSheet：返回当前规则所在的样式表对象（StyleSheet实例）

CSSRule.parentRule：返回包含当前规则的父规则，如不存在父规则（当前规则是顶层规则），返回null。父规则最常见情况是，当前规则包含在@media规则代码块中。

CSSRule.type：返回整数值，表示当前规则类型。有几种常见取值：①1：普通样式规则（CSSStyleRule实例）。②3：@import规则。③4：@media规则（CSSMediaRule实例）。④5：@font-face规则。

👉CSSStyleRule接口

如果一条CSS规则是普通的样式规则（不含特殊CSS命令），那么除了CSSRule接口，还部署了CSSStyleRule接口。

CSSStyleRule.selectorText：返回当前规则的选择器。这个属性可写。

CSSStyleRule.style：返回一个对象（CSSStyleDeclaration实例），代表当前规则的样式声明，即选择器后面的大括号里的部分。

🔺CSSStyleDeclaration实例的cssText属性，可返回所有样式声明，格式为字符串。

👉CSSMediaRule接口

如果一条CSS规则是@media代码块，那么它除了CSSRule接口，还部署了CSSMediaRule接口。主要提供media属性（返回@media规则的一个对象，MediaList实例）和conditionText属性（返回@media规则生效条件）

- window.matchMedia()

将CSS的MediaQuery条件语句转换为MediaQueryList实例。如果参数不是有效的MediaQuery条件语句，方法不报错，依然返回MediaQueryList实例。

①实例属性

MediaQueryList.media：返回字符串，表示对应的MediaQuery条件语句

MediaQueryList.matches：返回布尔值，表示当前页面是否符合指定的MediaQuery条件语句

MediaQueryList.onchange：如果MediaQuery条件语句适配环境发生变化，触发change事件。属性指定change事件的监听函数，函数参数是change事件对象（MediaQueryListEvent实例），该对象与MediaQueryList实例类似，也有media和matches属性。

②实例方法

MediaQueryList.addListener()为change事件添加监听函数。

MediaQueryList.removeListener()撤销监听函数。该方法不能撤销MediaQueryList.onchange属性指定的监听函数。

⭐实例

```css
/*test.css*/
p {
    font-size: 10px;
}
body {
    margin: 10px;
    background-color: blue;
}
@media screen and (min-width: 1000px){
    div { background-color: yellow;}
}
```

```js
// <link rel="StyleSheet" href="test.css">
// <div id="div">123</div>
console.log(document.styleSheets[0].cssRules[0].cssText);//"p { font-size: 10px; }"
console.log(document.styleSheets[0].cssRules[1].selectorText);//"body"
console.log(document.styleSheets[0].cssRules[2].conditionText);//"screen and (min-width: 1000px)"
```

**16.8 Mutation Observer API**

监视DOM变动（节点增减、属性变动、文本内容变动等）。

可理解为DOM发生变动就会触发Mutation Observer事件。但事件是同步触发而Mutation Observer是异步触发，等到所有DOM操作都结束才触发。

🔺①它等待所有脚本任务完成后才会运行（异步触发）②它把DOM变动记录封装成一个数组进行处理，而不是一条条个别处理DOM变动。③它既可以观察DOM所有类型变动，也可以指定只观察某一类变动。

- MutationObserver构造函数

指定实例的回调函数，回调函数在每次DOM变动后调用。回调函数有两个参数，第一个是变动数组，第二个是观察器实例：

```js
var observer = new MutationObserver(function (mutations, observer) {
  mutations.forEach(function(mutation) {
    console.log(mutation);
  });
});
```

- MutationObserver实例方法

①observe()：启动监听。两个参数，第一个参数是所要观察的DOM节点，第二个参数是一个配置对象，指定所要观察的特定变动。

观察器所能观察的DOM变动类型为：

childList：子节点变动（新增、删除或更改）

attributes：属性变动

characterData：节点内容或节点文本变动

想要观察哪种变动类型，就指定它的值为true。至少必须同时指定以上这三种观察的一种，如果均未指定将报错。

除了变动类型，还可设定以下属性：

subtree：布尔值，表示是否将该观察器应用于该节点的所有后代节点。

attributeOldValue：布尔值，表示观察attributes变动时，是否需要记录变动前的属性值。

characterDataOldValue：布尔值，表示观察characterData变动时，是否需要记录变动前的值。

attributeFilter：数组，表是需要观察的特定属性，如 ['class','src']。

🔺对于一个节点添加观察器，就像使用addEventListener()一样，多次添加同一个观察器无效，回调函数依然只会触发一次。如果第二个参数指定不同的对象，后面添加的会覆盖前面的。

②disconnect()，takeRecords()

disconnect()停止观察。takeRecords()清除变动记录，不再处理未处理的变动，返回变动记录的数组。

```js
// 保存所有没有被观察器处理的变动
var changes = mutationObserver.takeRecords();

// 停止观察
mutationObserver.disconnect();
```

- MutationRecord对象

DOM每次发生变化，会生成一条变动记录（MutationRecord实例）。实例包含了与变动相关的所有信息。Mutation Observer处理的就是一个个MutationRecord实例组成的数组。

MutationRecord对象包含了DOM相关信息：

type：观察的变动类型

target：发生变动的DOM节点

addedNodes：新增的DOM节点

removedNodes：删除的DOM节点

previousSibling：前一个同级节点，如没有返回null。

nextSibling：下一个同级节点，如没有返回null。

attributeName：发生变动的属性。如设置了attributeFilter，则只返回预先指定属性。

oldValue：变动前的值。只对attribute和characterData变动有效，如childList变动返回null。

```js
// <div id="div"></div>
var div = document.getElementById('div');
var options = {
    'childList':true,
};
var observer = new MutationObserver((mutations, observer1) => mutations.forEach(m => console.log(m.addedNodes[0])));
//<p></p> <span></span>
observer.observe(div,options);
var p = document.createElement('p');
div.appendChild(p);
var span = document.createElement('span');
div.appendChild(span);
```

⭐应用实例

①子元素变动

```js
var callback = function (records){
  records.map(function(record){
    console.log('Mutation type: ' + record.type);
    console.log('Mutation target: ' + record.target);
  });
};

var mo = new MutationObserver(callback);

var option = {
  'childList': true,
  'subtree': true
};

mo.observe(document.body, option);
```

②属性的变动

```js
var callback = function (records) {
  records.map(function (record) {
    console.log('Previous attribute value: ' + record.oldValue);
  });
};

var mo = new MutationObserver(callback);

var element = document.getElementById('#my_element');

var options = {
  'attributes': true,
  'attributeOldValue': true
}

mo.observe(element, options);
```

③取代DOMContentLoaded事件

