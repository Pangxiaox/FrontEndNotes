# JavaScript/ECMAScript学习笔记

### 0. 导论与基本语法/编程风格

①导论与基本语法

- JavaScript是一种轻量级脚本语言（不具备开发操作系统的能力，只用来编写控制其他大型应用程序，如浏览器，的“脚本”。
- JavaScript也是一种“嵌入式”语言，适合嵌入更大型的应用程序环境，去调用宿主环境提供的底层API。已经嵌入JavaScript的宿主环境有多种，最常见的是浏览器；另外还有服务器环境，即Node项目。
- ECMAScript和JavaScript的关系是：前者是后者的规格，后者是前者的一种实现。

- JavaScript是一种动态类型语言，变量可以随时更改类型。
- Java语言需要编译，而JavaScript语言则是运行时由解释器直接执行。

- 如果用var重新声明一个已经存在的变量，是无效的。如果第二次声明的时候还进行了赋值，则会覆盖掉前面的值。

```js
var x = 1;
var x = 2;
console.log(x);//2
```

- JavaScript引擎的工作方式是：先解析代码，获取所有被声明的变量，然后再一行一行运行。这造成的结果是所有变量的声明语句，都会被提升到代码头部，叫做“变量提升”。

```js
console.log(x);//undefined
var x = 1;
//等价于如下
var x;
console.log(x);
x = 1;
```

- 对于var命令，JavaScript的区块不构成单独的作用域。

- switch语句内部采用的是“严格相等运算符”，意味着不会发生类型转换。
- break/continue语句：如果存在多重循环，不带标签的话都只针对最内层循环。带标签的话就可以跳出特定循环。

```js
test:
    for (var i=0;i<2;i++){
        for (var j=0;j<3;j++){
            if (i===1 && j===1) break test;
            console.log(i,j);
        }
    }
```

- 标签也可以用于跳出代码块

```js
myLabel: {
    console.log('a');//'a'
    break myLabel;
    console.log('b');//不输出
}
```

②编程风格（建议）

缩进：Tab键和空格选择其中一个习惯用

区块：表示区块起首的大括号不要另起一行。

圆括号：表示函数调用时，函数名与左括号之间没有空格。表示函数定义时，函数名与左括号之间没有空格。其他情况，前面位置的语法元素与左括号之间都有一个空格。

分号：尽量不省略结尾分号。do...while循环是有分号的。

全局变量：建议避免使用。可考虑大写字母表示全局变量名。

变量声明：由于有“变量提升”，最好把变量声明都放在代码块头部。

不要使用with语句。

运算符：相等运算符会自动转换变量类型，建议只使用严格相等运算符。

不要将不同目的的语句合并成一行。

自增和自减运算符尽量用 += 和 -= 代替。

switch...case可用对象结构代替：

```js
function doAction(action) {
  var actions = {
    'hack': function () {
      return 'hack';
    },
    'slash': function () {
      return 'slash';
    },
    'run': function () {
      return 'run';
    }
  };

  if (typeof actions[action] !== 'function') {
    throw new Error('Invalid action.');
  }

  return actions[action]();
}
```



### ⭐1. call()、bind()、apply()

目的是**改变函数执行时的上下文**，即**改变函数运行时的this指向**。三个函数的第一个参数都是this的指向对象。

```js
let obj = {
    name: 'ann',
    run: function (param1,param2) {
        console.log(param1.toString()+param2.toString()+this.name);
    },
};
let obj1 = {
    name: 'bob',
};
obj.run(0,0); //00ann
obj.run.call(obj, 1,1); //11ann
obj.run.call(obj1,2,2); //22bob
obj.run.apply(obj1,[3, 3, ]); //33bob
obj.run.bind(obj1,4, 4)(); //44bob
```

🔺区别：

①call()和apply()是立即调用的，bind()返回对应函数，便于稍后调用

②call()的第二个参数是单独的，以逗号分隔，apply()的第二个参数需要放在一个数组中传进去

⭐特殊情况：

```js
function fn(){
    console.log(this);
}
// apply()方法结果同下
fn.call();//普通模式下this是window，严格模式下this是undefined
fn.call(null);//普通模式下this是window，严格模式下this是null
fn.call(undefined);//普通模式下this是window，严格模式下this是undefined
```



### 2. 数据结构/集合对象

#### 2.1 数组

**数组元素增删查改**

```js
let arr = [1, 3, 3, 4, ];
console.log(arr.indexOf(3)); // 1
console.log(arr.lastIndexOf(3)); // 2
console.log(arr.indexOf(0)); // -1(找不到元素)
arr.push(7,9); // 往数组末尾添加元素
console.log(arr);// [1,3,3,4,7,9]
arr.unshift(0,2); // 在数组前端添加元素
console.log(arr);// [0,2,1,3,3,4,7,9]
console.log(arr.pop());//9 移除数组最后一个元素并返回该元素
console.log(arr);// [0,2,1,3,3,4,7]
console.log(arr.shift());//0 移除数组最前一个元素并返回元素
console.log(arr);// [2,1,3,3,4,7]
arr.splice(1,2,10,20);// 在数组中删除或添加元素
console.log(arr);// [2,10,20,3,4,7]
```

**拼接、分割与拷贝**

```js
let arr2 = [1,2,3,4];
console.log(arr2.slice(1,3));// 分割 [2,3]
console.log(arr2.concat(2));// 拼接 [1,2,3,4,2]
```

**排序**

```js
let arr = [2, 3, 7, 5];
console.log(arr.reverse()); // [5,7,3,2] 数组反转
console.log(arr.sort((a, b) => a-b));// [2,3,5,7] 升序
console.log(arr.sort((a, b) => b-a));// [7,5,3,2] 降序
console.log(arr.sort()); // [2,3,5,7] 默认升序排列
```

**过滤**

```js
let arr = [2,4,6,8];
console.log(arr.map(item => item+1));// [3,5,7,9]
console.log(arr.filter(item => item<5));// [2,4]
console.log(array.some(item => item >10));// false
console.log(array.every(item => item <10));// true
```

**遍历**

```js
let array = [1, 2, 3, 4, ];
for(let item in array){
    console.log(array[item]);
}
for(let item of array){
    console.log(item);
}
array.forEach((item, index) => {
    console.log(index+'-'+item);
});
array.forEach((item, index) => {
    array[index] = item*2;
});
```



#### 2.2 Map

Map是一组键值对结构，具有极快查找速度

**创建**

```js
//创建空Map，之后添加元素
let map = new Map();
map.set('a', 1);
map.set('b', 2);
map.set('c', 3);
console.log(map.size);//3
//创建时初始化
let map2 = new Map([['a', 1],['b',3],['c',5]]);
console.log(map2.size);//3
```

**Map增删查改**

```js
let map = new Map([['a', 1],['b',3],['c',5]]);
console.log(map.has('b'));//true
console.log(map.get('a'));//1
map.set('a',2);
console.log(map.get('a'));//2
map.delete('c');
console.log(map);// Map { 'a' => 2, 'b' => 3 }
```

**遍历**

```js
let map = new Map([['a', 1],['b',3],['c',5]]);
map.forEach((value, key) => {
    console.log(key+'-'+value);
});
for(const item of map){
    console.log(item);//[ 'a', 1 ] [ 'b', 3 ] [ 'c', 5 ]
}
```

⭐for-of遍历出来是数组，item[0]为key，item[1]为value



#### 2.3 Set

**创建**

```js
let set = new Set([1,2,3,2]);
console.log(set.size);//3

let set2 = new Set();
set2.add(2).add(4).add(6);
set2.add(4);
console.log(set2.size);//3
```

**Set增删查改**

```js
let set = new Set([1,2,3,2]);
set.add(4);
console.log(set);// Set { 1, 2, 3, 4 }
set.add(1);// 无效
console.log(set);// Set { 1, 2, 3, 4 }
set.delete(5);// 无效
console.log(set);// Set { 1, 2, 3, 4 }
set.delete(2);
console.log(set);// Set { 1, 3, 4 }
console.log(set.has(3));// true
```

**遍历**

```js
let set = new Set([1,2,3,2]);
set.forEach(item => {
    console.log(item);// 1 2 3
});
for(const item of set){
    console.log(item);// 1 2 3
}
console.log(set.entries());// [Set Entries] { [ 1, 1 ], [ 2, 2 ], [ 3, 3 ] }
console.log(set.values());// [Set Iterator] { 1, 2, 3 }
console.log(set.keys());// [Set Iterator] { 1, 2, 3 }
```



#### 2.4 对象

```js
let obj = {
    name: 'abc',
    age: 23,
};
console.log(Object.keys(obj));// [ 'name', 'age' ]
console.log(Object.values(obj));// [ 'abc', 23 ]
console.log(Object.entries(obj));// [ [ 'name', 'abc' ], [ 'age', 23 ] ]
```



### ⭐3. 数组去重的方法

**利用Set**

```js
let arr = [1,2,2,3,1,5];
console.log(Array.from(new Set(arr)));
```

**利用数组的includes()**

```js
let arr = [1,2,2,3,1,5];
function unique(array){
    let arr1 = [];
    for (let i=0;i<array.length;i++){
        if (!arr1.includes(array[i])){
            arr1.push(array[i]);
        }
    }
    return arr1;
}
console.log(unique(arr));
```

**利用数组的indexOf()**

```js
let arr = [1,2,2,3,1,5];
function unique(array){
    let arr1 = [];
    for(let i=0;i<array.length;i++){
        if(arr1.indexOf(array[i]) === -1){
            arr1.push(array[i]);
        }
    }
    return arr1;
}
console.log(unique(arr));
```



### 4. 运算符

**数值运算符、负数值运算符**

```js
console.log(+'123');//123
console.log(+true);//1
console.log(+[]);//0
console.log(+{});//NaN
console.log(-'234');//-234
console.log(-true);//-1
console.log(-[]);//-0
console.log(-{});//NaN
```

作用是把任何值转化为数值，相当于 `Number()`。

**加法运算符**

```js
console.log('1'+'2');//12
console.log(1+3+'4');//44
console.log('1'+3+4);//134
console.log(true+true);//2
```

**指数运算符**

```js
console.log(2**3**2);//512，相当2**(3**2)
```

右结合

**余数运算符**

```js
console.log(1%3);// 1
console.log(-1%3);// -1
```

运算结果正负号由第一个运算子正负号决定

**取反运算符**

undefined、null、0、false、NaN和空字符串(""  或  '')取反之后返回true，其他值取反之后返回false

```js
console.log(![]);// false
console.log(!{});// false
```

🔺如果对一个值连续做两次取反运算，等于将这个值转化为它的布尔值，相当于 `Boolean()`。

```js
!!x // 相当于 Boolean(x)
```

**且运算符、或运算符**

且运算符：如果第一个运算子布尔值是true，则返回第二个运算子的值；如果第一个运算子的布尔值是false，则直接返回第一个运算子的值，且不再对第二个运算子求值。

或运算符：如果第一个运算子布尔值是true，则返回第一个运算子的值，且不再对第二个运算子求值；如果第一个运算子的布尔值是false，则返回第二个运算子的值。

```js
console.log(''&&'');//''
console.log(''&&3);//''
console.log(3&&'');//''
console.log('f'&&3);//3
console.log(1&&2&&'c');//'c'
console.log(3&&'c'&&''&&1);//''
console.log(''||'');//''
console.log(''||3);//3
console.log(3||'');//3
console.log('f'||3);//'f'
console.log(1||2||'c');//1
console.log(''||0||3||'c');//3
```

应用：

```js
// &&运算符
if(x){
    run();
}
// 等价于
x && run();

// ||运算符，为一个变量设置默认值，如下没有提供参数时默认为空字符串
test(this.x || '');
```

**严格相等运算符、严格不相等运算符、相等运算符、不相等运算符**

⭐相等运算符(==)比较两个值是否相等，严格相等运算符(===)比较它们是否是同一个值。如果两个值不是同一类型，则===返回false，而==则会把它们转换为同一类型，再用严格相等运算符进行比较。

👉严格相等运算符、严格不相等运算符

严格不相等运算符(!==)就是先求严格运算符的结果，然后取反。

①不同类型值

```js
console.log(1 === "1");//false
```

②同一类的原始类型值

数值、字符串、布尔值

```js
console.log(16 === 0x10);//true
```

🔺特殊情况

```js
console.log(+0 === -0);//true 正0等于负0
console.log(NaN === NaN);//false NaN与任何值（包括自身）都不相等
```

③复合类型值

对象、数组、函数。比较时是比较它们**是否指向同一个地址**。

```js
console.log([] === []);//false
console.log({} === {});//false
console.log(function (){} === function(){});//false
```

④undefined和null

```js
console.log(null === null);//true
console.log(undefined === undefined);//true
console.log(undefined === null);//false
```

👉相等运算符、不相等运算符

不相等运算符(!=)就是先求相等运算符的结果，然后取反。

①同一类型值

```js
console.log(1 == 1.0);//true
```

②原始类型值

转换为数值再比较

```js
console.log(1 == true);//true
console.log(0 == false);//true
console.log(2 == true);//false
console.log(2 == false);//false
console.log('' == 0);//true，Number('')为0
console.log('' == false);//true
console.log('1' == true);//true
console.log('true' == true);//false，Number('true')为NaN
```

③对象与原始类型值比较

先调用对象的 `valueOf()`方法，如果得到原始类型的值那就直接比较，如果得到的还是对象，则再调用 `toString()`方法，得到字符串形式，再比较

```js
console.log([1,2] == '1,2');//true
console.log([1] == true);//true
```

④undefined和null

undefined和null只有与自身比较或者互相比较时才返回true，与其他类型值比较时都返回false

```js
console.log(null == null);//true
console.log(undefined == undefined);//true
console.log(null == undefined);//true
console.log(0 == null);//false
console.log(0 == undefined);//false
```

**非相等运算符**

先看两个运算子是否都是字符串，如果是那就按字典顺序比较（实际上比较Unicode码点）；否则将两个运算子都转成数值，再比较数值大小。

```js
console.log('cat'<'Cat');//false，'c'的Unicode码点为99，'C'的Unicode码点为67
console.log('cbd'<'dbc');//true
```

①原始类型值

```js
console.log(5>'4');//true
console.log(true>false);//true
console.log(NaN > NaN);//false，任何值（包括NaN本身），与NaN使用非相等运算符比较，都是返回false
console.log(NaN <= NaN);//false
```

②对象

先调用 `valueOf()`方法，如果返回还是对象，再调用 `toString()`方法

```js
console.log([3]>[111]);//true，即'3' > '111'
```

**void运算符**

作用是执行一个表达式，然后不返回任何值，或者说返回undefined

主要应用是浏览器的书签工具，以及在超链接中插入代码防止网页跳转。

```js
<a href="javascript: void(f())">文字</a>
```

**逗号运算符**

用于对两个表达式求值，并返回后一个表达式的值。或者在返回一个值之前进行一些辅助操作。

```js
var x = 1;
var y = (x += 3, 5);
console.log(x);//4
console.log(y);//5
```

**运算顺序**

```js
var y = arr.length <= 0 || arr[0] === undefined ? x : arr[0];
//等价于
var y = ((arr.length <= 0) || (arr[0] === undefined)) ? x : arr[0];
```

右结合的运算符：赋值运算符、指数运算符、三元条件运算符

```js
q = a ? b : c ? d : e ? f : g;
//等价于
q = a ? b : (c ? d : (e ? f : g));
```

**二进制位运算符**

速度极快，但不直观

⭐位运算符只对整数起作用，如果一个运算子不是整数，会自动转为整数再执行。另外，在JavaScript内部，数值都是以64位浮点数形式存储，但做位运算是是以32位带符号的整数进行运算的，而且返回值也是一个32位带符号的整数。

```js
console.log(0|5);//，二进制或，5
console.log(0&5);//二进制与，0
console.log(~5);//二进制非，-6
console.log(0^5);//二进制异或，5
```

🔺异或运算可以在不引入临时变量前提下，互换两个变量的值

```js
let a = 23;
let b = 34;
a ^= b;
b ^= a;
a ^= b;
console.log(a,b);// 34 23
```

移位运算符

```js
console.log(16<<1);//32
console.log(16>>1);//8
console.log(-16<<1);//-32
console.log(-16>>1);//-8
console.log(16>>>1);//8
console.log(-16>>>1);//2147483640
```

左移运算符(<<)表示将一个数的二进制值向左移动指定位数，尾部补0。即乘以2的指定次方，向左移动时，最高位的符号位一起移动。

右移运算符(>>)表示将一个数的二进制值向右移动指定的位数。如果是正数，头部全部补0；如果是负数，头部全部补1。右移运算符相当于除以2的制定次方（最高位即符号位参与移动）

头部补零右移运算符(>>>)表示一个数的二进制形式向右移动时，头部一律补0，而不考虑符号位。该运算总是得到正值。



### 5. 数据类型

JavaScript提供六种数据类型：数值、字符串、布尔值、undefined、null、对象。ES6又新增了第7种Symbol类型的值。

数值、字符串、布尔值这三种类型称为原始类型。对象称为合成类型，又分为狭义的对象、数组、函数三个子类型。

JavaScript有三种方法确定一个值到底是什么类型：typeof运算符、instanceof运算符、Object.prototype.toString方法。

```js
console.log(typeof  123);//number
console.log(typeof '123');//string
console.log(typeof true);//boolean
function f(){}
console.log(typeof f);//function
console.log(typeof undefined);//undefined
console.log(typeof null);//object
console.log(typeof []);//object
console.log(typeof {});//object
```

instanceof运算符可区分数组和对象：

```js
var obj = {};
var arr = [];
console.log(obj instanceof Array);//false
console.log(arr instanceof Array);//true
```

**5.1 字符串**

单引号字符串内可以使用双引号，双引号字符串内可以使用单引号。

如果要在单引号字符串内部使用单引号，就必须在内部的单引号前面加上反斜杠用来转义，双引号字符串内部使用双引号同理。

```js
console.log('It is a \'dog\'');//It is a 'dog'
console.log("It is a \"cat\"");//It is a "cat"
```

- 需要用反斜杠转义的特殊字符：

\0：null，\b：后退键，\f：换页符，\n：换行符，\r：回车键，\t：制表符，\v：垂直制表符，\‘：单引号，\“：双引号，\\：反斜杠

- 字符串与数组，length属性

字符串可被视为字符数组。字符串内部的单个字符无法改变和增删。

length属性返回字符串长度，该属性无法改变。

```js
let str = 'happy';
console.log(str[0]);//'h'
console.log(str.length);//5
delete str[0];
console.log(str);//'happy'
str[1] = 'b';
console.log(str);//'happy'
```

- 字符集

JavaScript使用Unicode字符集。JavaScript引擎内部，所有字符都用Unicode表示。

每个字符在JavaScript内部都是以16位（2个字节）的UTF-16格式存储。即JavaScript的单位字符长度固定为16位长度，即2个字节。

- Base64转码

Base64可以将任意值转成0~9、A~Z、a~z、+和/这64个字符组成的可打印字符。主要目的不是为了加密，而是为了不出现特殊字符。另一使用场景是需要以文本格式传递二进制数据时。

JavaScript原生提供了两个Base64相关的方法。

`btoa(str)`：任意值转为Base64编码   `atob(str)`：Base64编码转为原来的值

如果将非ASCII码字符转为Base64编码，必须在中间插入一个转码环节：

`btoa(encodeURIComponent(str))`、`decodeURIComponent(atob(str))`

**5.2 数值**

- 整数和浮点数、数值精度、数值范围

JavaScript语言的底层没有整数，所有数字都是小数（64位浮点数）

JavaScript浮点数的64个二进制位，从最左边开始，组成如下：

第1位：符号位，0表示正数，1表示负数

第2位到第12位（共11位）：指数部分

第13位到第64位（共52位）：小数部分（有效数字）

JavaScript提供的有效数字最长为53位二进制位： `(-1)^符号位 * 1.xx...xxx * 2^指数部分`

如果一个数大于等于2的1024次方，发生“正向溢出”，如果一个数小于等于2的-1075次方，发生“负向溢出”。

```js
console.log(Math.pow(2,1024));//Infinity
console.log(Math.pow(2,-1075));//0
console.log(Number.MAX_VALUE);//1.7976931348623157e+308
console.log(Number.MIN_VALUE);//5e-324
```

- 数值表示法、数值进制

科学计数法：当小数点前数字多于21位或者小数点后的零多于5个，自动将数值转为科学计数法。

```js
console.log(12e2);//1200
console.log(.3E-1);//0.03
```

十进制：没有前导0的数值。八进制：有前缀0o或0O的数值。十六进制：有前缀0x或0X的数值。二进制：有前缀0b或者0B的数值。

- 特殊数值

JavaScript内部实际有两个0，一个+0，一个-0。区别在于64位浮点数表示法的符号位不同。它们是等价的。

唯一区别场景是当+0或-0当作分母，返回的值不相等：

```js
console.log((1 / +0) === (1 / -0));//false
```

除以正0得到+Infinity，除以负0得到-Infinity。

JavaScript中NaN主要出现在字符串解析成数字出错的场合。NaN的数据类型依然是Number。

```js
console.log(Boolean(NaN));//false
console.log([NaN].indexOf(NaN));//-1,数组的indexOf方法内部使用严格相等运算符
console.log(NaN === NaN);//false,NaN不等于任何值，包括自身
console.log(NaN + 1);//NaN,NaN与任何数（包括自身）运算结果NaN
console.log(0/0);//NaN
```

JavaScript中，Infinity表示“无穷”，表示正德数值太大或者负的数值太小。

🔺Infinity：运算与比较法则

①Infinity大于一切数值（除了NaN）、-Infinity小于一切数值（除了NaN）。Infinity与NaN比较，总是返回false。

②Infinity加上或乘以Infinity，返回的还是Infinity。Infinity减去或除以Infinity，得到NaN。

③0乘以Infinity，返回NaN。0除以Infinity，返回0。Infinity除以0，返回Infinity。

④Infinity与null计算时，null转为0，等同于与0的计算。

⑤Infinity与undefined计算，返回都是NaN。

⑥非零正数除以-0，得到-Infinity。负数除以-0，得到Infinity。

⑦Infinity普通四则运算，符合无穷的数学计算规则：

```js
console.log(2*Infinity);//Infinity
console.log(2-Infinity);//-Infinity
console.log(Infinity/2);//Infinity
console.log(2/Infinity);//0
```

- 与数值相关的全局方法

①`isFinite()`：返回布尔值，表示某个值是否为正常的数值

除了Infinity、-Infinity、NaN和undefined这几个值返回false，对于其他数值都返回true。

② `isNaN()`：判断一个值是否为NaN

isNaN()只对数值有效，如果传入其他值，会被先转成数值。如传入字符串，字符串会被先转成NaN，最后返回true。即isNaN为true的值，有可能不是NaN，而是一个字符串。

```js
console.log(isNaN({}));//true
console.log(isNaN(['abc']));//true
console.log(isNaN(['123']));//false
console.log(isNaN([]));//false
console.log(isNaN([123]));//false
```

⭐判断NaN最可靠的方法是利用NaN为唯一不等于自身的值这一特点：

```js
function myIsNaN(num) {
    return num !== num;
}
```

③ `parseFloat()`：将一个字符串转为浮点数

如果字符串符合科学计数法，则会进行相应转换。

如果字符串包含不能转为浮点数的字符，则不再往后转换，返回已经转好的部分。

如果参数不是字符串，或者字符串的第一个字符不能转化成浮点数，则返回NaN。

```js
console.log(parseFloat('21e-3'));//0.021
console.log(parseFloat('3.1aa'));//3.1
console.log(parseFloat(''));//NaN
console.log(parseFloat([]));//NaN
console.log(parseFloat({}));//NaN
```

④ `parseInt()`：将字符串转为整数

如果parseInt的参数不是字符串，则会先转为字符串再转换。

如果转换时遇到不能转为数字的字符，就不再继续，返回已经转好的部分。

如果字符串的第一个字符不能转化为数字（后面跟着数字的正负号除外），返回NaN。

如果字符串以0x或0X开头，parseInt会将其按照十六进制数解析；如果字符串以0开头，则按照十进制解析。

对于会自动转为科学计数法的数字，parseInt会将科学计数法表示方法视为字符串。

```js
console.log(parseInt(1.02));//1
console.log(parseInt('2bc'));//2
console.log(parseInt(''));//NaN
console.log(parseInt('+3'));//3
console.log(parseInt('.2'));//NaN
console.log(parseInt('0x12'));//18
console.log(parseInt('024'));//24
console.log(parseInt(0.00000005));//5
```

parseInt方法还可以接受第二个参数（2到36之间），表示被解析的值的进制，返回该值对应的十进制数。

如果第二个参数不是数值，会被自动转为一个整数。这个整数一旦超出2到36之间这个范围，则返回NaN。如果第二个参数是0、undefined和null，则直接忽略。

如果字符串包含对于指定进制无意义字符，则从最高位开始，只返回可以转换的数值。如果最高位无法转换，则直接返回NaN。

```js
console.log(parseInt('100', 13));//169
console.log(parseInt('100',null));//100
console.log(parseInt('100',1));//NaN
console.log(parseInt('1112',2));//7
```

**5.3 null、undefined和布尔值**

```js
console.log(undefined === null);//false
console.log(undefined == null);//true
console.log(Number(null));//0
console.log(2+null);//2
console.log(Number(undefined));//NaN
console.log(2+undefined);//NaN
```

null表示空值，即该处的值现在为空。调用函数时，某个参数未设置任何值，这时可以传入null，表示该参数为空。undefined表示“未定义”，常用于变量声明了但没有赋值这种情况等。

undefined、null、false、0、NaN、空字符串会被转为false，其他值（空数组、空对象等）都被视为true。

**5.4 对象**

对象是一组“键值对”的集合，是一种无序的复合数据集合。

- 概述

对象的所有键名都是字符串（ES6又引入了Symbol值也可以作为键名），所以加不加引号都可以。如果键名不符合标识名的条件（如第一个字符为数字，或含有空格或运算符），且也不是数字，则必须加上引号。

```js
let obj = {
    '1':'a',
    a:'1',
    '1a':'a1',
};
console.log(obj.a);//1
console.log(obj['a']);//1
console.log(obj['1']);//a
```

对象的每一个键名称为“属性”，键值可以实任何数据类型。如果一个属性的值为函数，通常把这个属性称为“方法”。如果属性的值还是一个对象，就形成了链式调用。属性可以动态创建，不必在对象声明时就指定。

```js
var obj = {
    func: function (a,b) {
        console.log(a+b);
    }
};
obj.func(1,2);//3
obj['func'](1,2);//3

var obj1 = {};
var obj2 = { a: 'test'};
obj1.b = obj2;
console.log(obj1.b.a);//'test'
```

如果不同的变量名指向同一个对象，那么它们都是这个对象的引用，即指向同一个内存地址。修改其中一个变量，会影响到其他所有变量。此时如果取消某一个变量对于原对象的引用，不会影响到另一个变量。

```js
var o1 = {};
var o2 = o1;
o1.a = 2;
console.log(o2.a);//2
o1 = 1;
console.log(o2);// { a: 2 }
```

对象采用大括号表示，与语句块很相似，如果要解释为对象，最好在大括号前加上圆括号。因为圆括号里只能是表达式，所以确保大括号只能解释为对象。

```js
console.log(eval('{a:1}'));//1
console.log(eval('({a:1})'));//{ a: 1 }
```

- 属性的操作

①读取

一种是使用点运算符，还有一种是使用方括号运算符。

如果使用方括号运算符，键名必须放在引号里面，否则会被当作变量处理。

数字键名可以不加引号，因为会自动转成字符串。

```js
var obj =  {1:'a','a':1};
console.log(obj['1']);//'a'
console.log(obj[1]);//'a'
console.log(obj['a']);//1
```

②赋值

点运算符和方括号运算符，不仅可以用来读取值，还可以用来赋值。

③查看

查看一个对象本身的所有属性，用 `Object.keys()`方法。

```js
var obj1 = {'a':1,2:'b'};
console.log(Object.keys(obj1));//[ '2', 'a' ]
```

④删除

delete命令用于删除对象属性，删除成功后返回true。

```js
var obj1 = {'a':1,2:'b'};
delete obj1.a;//true
console.log(obj1);//{ '2': 'b' }
```

🔺删除一个不存在属性，delete不报错，而且返回true。不能根据delete命令的结果认定某个属性是存在的。

只有一种情况，delete命令会返回false，即属性存在且不得删除：

```js
var obj = Object.defineProperty({}, 'p', {
    value: 123,
    configurable: false
});

console.log(obj.p);// 123
delete obj.p; // false
```

delete命令只能删除对象本身的属性，无法删除继承的属性。即使delete返回true，该属性以然可能读取到值。

```js
var obj = {};
delete obj.toString;//true
console.log(obj.toString);//[Function: toString]
```

⑤判断属性是否存在

in运算符检查对象是否包含某个属性。

对象的 `hasOwnProperty()`方法判断是否为对象自身属性。

```js
var obj = {a:1};
console.log('a' in obj);//true
if ('toString' in obj) {
    console.log(obj.hasOwnProperty('toString'));//false
}
```

⑥属性的遍历

for...in循环：遍历的是对象所有可遍历的属性，会跳过不可遍历的属性。不仅遍历对象自身的属性，还遍历继承的属性。

```js
var obj = {a:1,b:2,c:3};
for (var i in obj){
    console.log(i,obj[i]);//键名为i，键值为obj[i]
}
```

- with语句

操作同一个对象的多个属性时，提供一些书写方便，但不建议这样写。

```js
var obj = {a:1,b:2,c:3};
with (obj) {
    console.log(a);//1
}
var obj2 = {};
with (obj) {
    a = 10;
}
console.log(obj2.a);//undefined
```

with区块内部有变量赋值操作时，必须是当前对象已经存在的属性，否则会创造一个当前作用域的全局变量。

**5.5 函数**

- 函数的声明

①function命令

```js
function print() {
    console.log('aaa');
}
```

②函数表达式

```js
var print = function(str) {
    console.log(str);
};
print('aaa');
```

当使用函数表达式时，function命令后带有函数名时，该函数名只在函数体内部有效，在函数体外部无效：

```js
var print = function p() {
    console.log(typeof p);
};
print();//function
```

③Function构造函数

较少使用

```js
var add = new Function('x','y','return x+y');
console.log(add(1,2));//3
```

🔺函数的重复声明

如果同一个函数被多次声明，后面的声明就会覆盖前面的声明。

```js
function f(){
    console.log(1);
}
f();//2
function f(){
    console.log(2);
}
f();//2
```

- 第一等公民

函数与其他数据类型地位平等。如把函数赋值给变量和对象的属性，也可以当作参数传入其他函数，或者作为函数的结果返回。

```js
function minus(a,b) {
    console.log(a-b);
}
// 将函数赋值给一个变量
var op = minus;
op(3,2);//1

// 将函数作为参数和返回值
function m(op){
    return op;
}
m(minus)(2,1);//1
```

- 函数名的提升

采用function命令你个声明函数时，整个函数会像变量声明一样，被提升到代码头部。

```js
f();
function f(){}
```

如上，实际上在调用函数前就已经声明了。

但如果采用赋值语句定义函数就会报错：

```js
f();
var f = function f(){};
//等价于以下
var f;
f();//undefined
f = function(){};
```

🔺当采用function命令和var赋值语句声明同一个函数，由于存在函数提升，最后采用var赋值语句的定义：

```js
var f = function() {
    console.log(1);
};
function f() {
    console.log(2);
}
f();//1
```

- 函数的属性与方法

①name属性

返回函数的名字

```js
function f1(){}
console.log(f1.name);//"f1"
var f2 = function(){};
console.log(f2.name);//"f2"
var f3 = function func(){};
console.log(f3.name);//"func",真正的函数名是f3，func只在函数体内部可用
```

②length属性

返回函数预期传入的参数个数，即函数定义之中的参数个数。length属性提供了一种机制：判断定义时和调用时参数差异，实现OOP的“方法重载”。

```js
function f1(a,b,c){}
console.log(f1.length);//3
```

③toString()

返回一个字符串，内容是函数源码。

```js
function f() {
    a();
}
console.log(f.toString());
//function f() {
//     a();
// }
console.log(Math.sqrt.toString());//function sqrt() { [native code] }
```

- 函数作用域

①定义

作用域指变量存在的范围。

ES5中，有两种作用域：全局作用域（变量在整个程序中一直存在，所有地方都可以读取）和函数作用域（变量只在函数内部存在）。

ES6中新加了块级作用域。

函数内部定义的变量，会在该作用域内覆盖同名全局变量。

```js
var v = 3;//全局变量
function f() {
    var v = 4;//局部变量
    console.log(v);
}
f();//4
console.log(v);//3
```

对于var命令，局部变量只能在函数内部声明，在其他区块中声明一律都是全局变量：

```js
if (true) {
    var x = 1;
}
console.log(x);//1
```

②函数内部的变量提升

与全局作用域一样，函数作用域也会产生"变量提升"现象。var命令声明的变量，不管在什么位置，变量声明都会被提升到函数体头部：

```js
function test(x) {
    if (x > 1) {
        var tmp = x - 1;
    }
}
//等价于
function test(x) {
    var tmp;
    if (x > 1) {
        tmp = x - 1;
    }
}
```

③函数本身的作用域

函数执行时所在的作用域，是定义时的作用域，而不是调用时所在的作用域。

```js
var a = 1;
var func = function() {
    console.log(a);
};
function f() {
    var a = 3;
    func();
}
f();//1
```

```js
var func = function() {
    console.log(a);
};
function f(f1) {
    var a = 3;
    f1();
}
f(func);//ReferenceError: a is not defined
```

函数体内部声明的函数，作用域绑定函数体内部：

```js
function a() {
    var x = 1;
    function b() {
        console.log(x);
    }
    return b;
}
var x = 2;
var f = a();
f();//1
```

- 参数

①参数的省略

省略的参数值会变为undefined。但没办法只省略靠前的参数而保留靠后的参数，如果一定要省略靠前参数，只有显式传入undefined。

```js
function f(a,b){
    console.log(a);
}
f(2);//2
f();//undefined
f(undefined,2);//undefined
```

②传递方式

函数参数如果是原始类型值（数值、字符串、布尔值），传递方式是传值传递。即在函数体内修改参数值，不会影响到函数外部。

函数参数如果是复合类型值（数组、对象、其他函数），传递方式是传址传递。即传入函数的原始值的地址，因此在函数内部修改参数，将会影响到原始值。

🔺如果函数内部修改的不是参数对象的某个属性，而是替换掉整个参数，这是不会影响到原始值。

```js
var a = 1;
function f(b){
    b = 2;
}
f(a);
console.log(a);//1

var obj1 = {a:1};
function f2(b){
    b.p = 2;
}
f2(obj1);
console.log(obj1.p);//2

var arr = [1,3];
function f3(b) {
    b = [1,2];
}
f3(arr);
console.log(arr);// [1,3]
```

③同名参数

如果有同名参数，则取最后出现的那个值。即使后面的参数没有值或被省略也是以其为准。如果想要获取前面参数的值，可以使用 arguments对象。

```js
function f(a,a){
    console.log(a);
}
f(2,3);//3
f(1);//undefined

function f2(a,a){
    console.log(arguments[0]);
}
f2(1);//1
```

④arguments对象

由于JavaScript允许函数有不定数目的参数，arguments对象包含了函数运行时的所有参数。

正常模式下，arguments对象可以在运行时修改。严格模式下，arguments对象与函数参数不具有联动关系，修改arguments对象不会影响到实际的函数参数。

通过arguments.length属性可判断函数调用时到底带几个参数。

```js
function f(test){
    console.log(arguments[0]);
    console.log(arguments[1]);
    console.log(arguments.length);
}
f(2,1);// 2 1 2
f();// undefined undefined 0
```

🔺arguments很像数组，但实际它是一个对象。如果要让arguments对象使用数组的方法，要将arguments转为真正的数组。

```js
var args = Array.prototype.slice.call(arguments);
```

arguments对象带有一个callee属性，返回它所对应的原函数。可通过arguments.callee，达到调用函数自身目的。在严格模式下禁用，不建议使用。

```js
var f = function() {
    console.log(arguments.callee === f);
};
f();//true
```

- 函数的其他知识点

①闭包

用处1：读取外层函数内部的变量。

JavaScript语言特有“链式作用域”结构，子对象会一级一级向上寻找所有父对象的变量。所以，父对象的所有变量，对子对象都是可见的，反之则不成立。

```js
function f1(){
    var n = 12;
    function f2(){
        console.log(n);
    }
    return f2;
}
f1()();//12
```

闭包就是函数f2，f2记住了它诞生的环境f1，所以f2可以得到f1的内部变量。闭包就是将函数内部和函数外部连接起来的一座桥梁。

用处2：让这些变量始终保持在内存中，即闭包可以使得它诞生环境一直存在：

```js
function increment(inc){
    return function(){
        return ++inc;
    };
}
var i = increment(2);
console.log(i());//3
console.log(i());//4
```

闭包可以看作是函数内部作用域的一个接口。

闭包用到了外层变量inc，导致外层函数increment()不能从内存释放。只要闭包没有被垃圾回收机制清除，外层函数提供的运行环境也不会被清除，它的内部变量就始终保存着当前值，供闭包读取。

用处3：封装对象的私有属性和私有方法

```js
function Student(name){
    var age;
    function setAge(n){
        age = n;
    }
    function getAge(){
        return age;
    }
    return {
        name: name,
        getAge:getAge,
        setAge:setAge,
    };
}
var s1 = Student('bob');
s1.setAge(26);
console.log(s1.getAge());//26
```

🔺外层函数每次运行，都会生成一个新的闭包，而这个闭包又会保留外层函数的内部变量，所以内存消耗很大。不能滥用闭包，否则会造成网页性能问题。

②立即调用的函数表达式（IIFE）

有时需要在定义函数之后立即调用该函数，这时不能再函数的定义之后加上圆括号。

```js
function(){ /* code */ }();
// SyntaxError: Unexpected token (
```

当作表达式时，函数可以定义后直接加圆括号调用：

```js
var f = function f(){
    console.log(1);
}();
f;//1
```

🔺函数定义后立即调用的解决方法是不要让function出现在行首，让引擎将其理解成一个表达式：

```js
(function(){ /* code */ }());
// 或者
(function(){ /* code */ })();
```

通常情况，只对匿名函数使用这种“立即执行的函数表达式”。目的为：不必为函数命名，避免污染全局变量；IIFE内部形成一个单独的作用域，封装一些外部无法读取的私有变量：

```js
(function () {
  var tmp = newData;
  processData(tmp);
  storeData(tmp);
}());
```

- eval命令

接受一个字符串作为参数，并将这个字符串当作语句执行。

放在eval中的字符串，应该有独自存在的意义。

eval没有自己的作用域，都在当前作用域内执行，因此可能会修改当前作用域变量的值，有安全风险。

```js
eval('var a = 3');
console.log(a);//3
console.log(eval(1234));//1234
```

🔺eval本质是在当前作用域之中注入代码。由于安全风险和不利于JavaScript引擎优化执行速度，所以不推荐使用。eval最常见的场合是解析JSON数据的字符串，但正确解析方法是使用原生的JSON.parse方法。

eval的别名调用·：

```js
eval.call(null, '...')
window.eval('...')
(1, eval)('...')
(eval, eval)('...')
```

凡是使用别名执行eval，eval内部一律是全局作用域。

**5.6 数组**

- 定义

数组除了在定义时赋值，也可以先定义后赋值。

任何类型的数据，都可以放入数组。

如果数组的元素还是舒徐，就形成了多维数组。

```js
var arr1 = [
    {a:2},[1,2],function(){return 1;}
];
console.log(arr1[0],arr1[1],arr1[2]);// { a: 2 } [ 1, 2 ] [Function]

var arr2 = [[1,2],[3,4]];
console.log(arr2[0][1]);//2
```

- 数组的本质

数组属于一种特殊的对象。typeof运算符会返回数组的类型是object。

数组成员的键名就是整数0、1、2...，其实也是字符串，之所以可以用数值读取是因为非字符串的键名会被转为字符串。

```js
var arr = [1,2,3];
console.log(Object.keys(arr));//[ '0', '1', '2' ]
console.log(arr['1']);//2
```

- length属性

返回数组的成员数量。

只要是数组，就一定有length属性，该属性是一个动态值，等于键名中的最大整数加上1。

```js
var arr = [1,2];
arr[12] = 10;
console.log(arr.length);//13
console.log(arr[10]);//undefined
```

length属性可写，如果设置一个小于当前成员个数的值，该数组成员数量会自动减少到length设置的值。如果设置一个大于当前成员个数的值，则数组成员数量会增加到这个值，新增位置都是空位。

清空数组的一个方法就是将length属性设为0.

```js
var arr = [2,4,6];
arr.length = 2;
console.log(arr);//[ 2, 4 ]
arr.length = 10;
console.log(arr[6]);//undefined
arr.length = 0;
console.log(arr);//[]
```

可以为数组添加属性，但这不影响length属性的值：

```js
var a = [];
a['a'] = 'abc';
a[1.1] = 'bcd';
console.log(a.length);//0
```

- in运算符

检查某个键名是否存在

🔺如果数组某个位置时空为，in运算符返回false。

```js
var arr = ['a','b','d'];
console.log(2 in arr);//true
console.log('2' in arr);//true

var arr1 = [];
arr1[20] = 10;
console.log(10 in arr1);//false

var arr2 = [undefined, 1, undefined];
console.log('0' in arr2);//true
```

- for...in循环和数组的遍历

for...in循环不仅遍历数组所有的数字键，还会遍历非数字键。

```js
var a = [1,2,3];
a.a = 'a';
for(var i in a){
    console.log(a[i]);// 1 2 3 'a'
}
```

因此数组遍历不推荐用for...in循环，可考虑for循环或while循环。

也可用forEach循环：

```js
var a = [1,2,3];
a.forEach(item => {
    console.log(item);// 1 2 3
});
```

- 数组的空位

当数组某个位置是空元素，即两个逗号之间没有任何值，称该数组存在空位。

空位不影响length属性。空位可以读取，返回undefined。使用delete命令删除一个数组成员会形成空位，并且不会影响length属性。

```js
var a = [1,,3];
console.log(a.length);//3
console.log(a[1]);//undefined

var b = [1,4,7];
delete b[1];
console.log(b.length);//3
console.log(b[1]);//undefined
```

🔺数组某个位置是空位，与某个位置是undefined不一样。如果书空位，使用数组的forEach方法，for...in结构以及Object.keys方法遍历，空位都会被跳过。总之，空位就是数组没有这个元素，所以不会遍历到；undefined表示数组有这个元素，值是undefined，所以遍历不会跳过。

- 类似数组的对象

如果一个对象的所有键名都是正整数或零，并且有length属性，这个对象称为“类似数组的对象”。

```js
var obj = {
    0:'a',
    1:'b',
    length:2
};
console.log(obj[0]);//'a'
```

本质上它是一个对象而不是数组，length属性不是动态值，不会随着成员的变化而变化。

典型的“类似数组的对象”是函数的arguments对象，以及大多数DOM元素集，还有字符串。

数组的slice方法可将“类似数组的对象”变成真正的数组。

```js
var arr = Array.prototype.slice.call(arrayLike);
```

“类似数组的对象”另一个使用数组的方法，是通过call方法把数组的方法放到对象上。

```js
Array.prototype.forEach.call(arrayLike, print);//print为某个函数名
```

```js
var obj = {
    0:'a',
    1:'b',
    length:2
};
var arr = Array.prototype.slice.call(obj);
arr.push(3);
console.log(arr);//[ 'a', 'b', 3 ]
```



### 6. 数据类型转换

如果运算符发现，运算子的类型与预期不符，就会自动转换类型。

```js
console.log('3'-'1');//2
```

- 强制转换

指使用 `Number()`、`String()`、`Boolean()`三个函数，分别转换为数字、字符串或布尔值。

①Number()

原始类型值

```js
console.log(Number(123));//123
console.log(Number('123'));//123
console.log(Number('12a3'));//NaN
console.log(Number(''));//0
console.log(Number(true));//1
console.log(Number(false));//0
console.log(Number(undefined));//NaN
console.log(Number(null));//0
```

对象：参数是对象时，将返回NaN，除非是包含单个数值的数组

```js
console.log(Number({x:1}));//NaN
console.log(Number([1,2,3]));//NaN
console.log(Number([3]));//3
console.log(Number({}));//NaN
console.log(Number([]));//0
```

🔺Number()的转换规则：

第一步，调用对象自身的valueOf方法。如果返回原始类型值，直接对该值使用Number函数，不再进行后续步骤。

第二步，如果valueOf方法返回的还是对象，则改为调用对象自身的toString方法。如果toString方法返回原始类型值，则对该值使用Number函数，不再进行后续步骤。

第三步，如果toString方法返回的是对象，就报错。

②String()

原始类型值

```js
console.log(String(123));//"123"
console.log(String('123'));//"123"
console.log(String(true));//"true"
console.log(String(undefined));//"undefined"
console.log(String(null));//"null"
```

对象

String方法的参数如果是对象，返回一个类型字符串；如果是数组，返回该数组的字符串形式。

```js
console.log(String({x:1}));//"[object Object]"
console.log(String({}));//"[object Object]"
console.log(String([]));//""
console.log(String([1,2,4]));//"1,2,4"
```

🔺String()转换规则

第一步，先调用对象自身的toString方法。如果返回原始类型值，则对该值使用String函数，不再进行后续步骤。

第二步，如果toString方法返回的是对象，再调用原对象的valueOf方法。如果valueOf方法返回原始类型值，则对该值使用String函数，不再进行后续步骤。

第三步，如果valueOf方法返回的是对象，就报错。

③Boolean()

```js
console.log(Boolean(null));//false
console.log(Boolean(undefined));//false
console.log(Boolean(0));//false
console.log(Boolean(NaN));//false
console.log(Boolean(''));//false
console.log(Boolean({}));//true
console.log(Boolean([]));//true
console.log(Boolean(new Boolean(false)));//true
```

所有对象（包括空对象）转换结果都是true

- 自动转换

情况1：不同类型的数据互相运算

```js
console.log('ab'+23);//"ab23"
```

情况2：对非布尔值类型数据求布尔值

```js
if ('123'){
    console.log('true');//'true'
}
```

情况3：对非数值类型的值使用一元运算符（即 + 和 - ）

```js
console.log(+ {x:1});//NaN
console.log(- [1,2]);//NaN
```

自动转换规则：预期什么类型的值，就调用该类型的转换函数。如某个位置预期为字符串，则调用String()函数，如该位置既可以是字符串也可能是数值，那么默认转为数值。

①自动转换为布尔值

除了undefined、null、+0或-0、NaN、空字符串，其他都是自动转为true

②自动转换为字符串

先将复合类型的值转为原始类型的值，再将原始类型的值转为字符串。

在字符串的加法运算种，当一个值为字符串，另一个值为非字符串，则后者转为字符串：

```js
console.log('2'+1);//'21'
console.log('2'+{});//'2[object Object]'
console.log('2'+[]);//'2'
console.log('2'+[1,2]);//'21,2'
```

③自动转换为数值

```js
console.log('3'*'6');//18
console.log('3'-2);//1
console.log('4'*[]);//0
console.log(true/'4');//0.25
console.log('a'-1);//NaN
console.log(null+3);//3
console.log(undefined+3);//NaN

console.log(+true);//1
console.log(-'1');//-1
console.log(+'ab');//NaN
```

🔺null转为数值时为0、undefined转为数值时为NaN。



### 7. 错误处理机制

- Error实例对象

JavaScript原生提供Error构造函数，所有抛出的错误都是这个构造函数的实例。

Error实例对象的几个属性：

message：错误提示信息

name：错误名称（非标准属性）

stack：错误的堆栈（非标准属性）

- 原生错误类型

存在Error的6个派生对象

①SyntaxError对象

解析代码时发生的错误

```js
var 3a;//变量名错误
console.log 'abc');//缺少括号
```

②ReferenceError对象

引用一个不存在的变量时发生的错误。或将一个值分配给无法分配的对象（如对函数运行结果赋值）

```js
a;//不存在的变量
console.log() = 1;//等号左侧不是变量
```

③RangeError对象

一个值超出有效范围发生的错误。几种情况：数组长度为负数、Number对象的方法参数超出范围、函数堆栈超过最大值。

```js
var a = new Array(-2);//数组长度为负数
```

④TypeError对象

变量或参数不是预期类型发生的错误，如对字符串、布尔值、数值等原始类型的值使用new命令，因为new命令参数应该是一个构造函数。

```js
new 123;//error

var obj = {};
obj.unknown();//error
```

⑤URIError对象

URI相关函数的参数不正确抛出的错误。 主要是 `encodeURI()`、`decodeURI()`、`encodeURIComponent()`、`decodeURIComponent()`、`escape()`、`unescape()`这六个函数。

```js
decodeURI('%1');
```

⑥EvalError对象

eval函数没有被正确执行时抛出错误，该错误类型已不再使用。

🔺这6种派生错误，连同Error对象，都是构造函数。

```js
var err1 = new TypeError('error');
console.log(err1.message);//'error'
```

- 自定义错误

```js
function userError(msg){
    this.msg = msg;
}
userError.prototype = new Error();
userError.prototype.constructor = userError;
var obj = new userError('自定义错误');
console.log(obj.msg);//'自定义错误'
```

定义了一个错误对象userError，让它继承Error对象。

- throw语句

手动中断程序执行，抛出一个错误。

throw抛出的错误是它的参数，可以是Error实例，也可以抛出自定义错误，也可以是任何类型值（字符串、数值、布尔值、对象等）

```js
if (x<1) {
    throw new Error('x必须大于等于1');
}
```

- try...catch结构

允许对错误进行处理，选择是否往下执行。

```js
try {
    throw new Error('err');
} catch (e) {
    console.log(e.message);//'err'
}
console.log('ok');//'ok'
```

catch代码块中，还可以再抛出错误，甚至使用嵌套的try...catch结构

catch代码块中可加入判断语句捕捉不同类型错误。

- finally代码块

try...catch允许在最后加入finally代码块，表示不管是否出现错误都必须执行的语句

```js
function work(){
    try {
        throw new Error('err');
        console.log('111');
    } finally {
        console.log('222');
    }
}
work();
// 222
// Error: err
// ...
```

中断执行之前先执行finally代码块

```js
var cnt = 0;
function count(){
    try {
        return cnt;
    } finally {
        cnt++;
    }
}
console.log(count());//0
console.log(cnt);//1
```

🔺return语句的执行排在finally代码块之前，只是等finally代码执行完毕后才返回。

⭐try...catch...finally执行顺序：

```js
function f(){
    try {
        console.log(0);
        throw 'err';
    } catch (e){
        console.log(1);
        return true;//原本延迟到finally代码块结束再执行
        console.log(2);//不执行
    } finally {
        console.log(3);
        return false;//覆盖前面的return
        console.log(4);//不执行
    }
    console.log(5);//不执行
}
var res = f();
console.log(res);// 0 1 3 false
```

⭐catch代码块结束执行前，会先执行finally代码块。

catch代码块中，触发转入finally代码块的标志不仅有return语句，还有throw语句：

```js
function f(){
    try {
        throw 'err';
    } catch(e) {
        console.log('catch err');
        throw e;
    } finally {
        return false;
    }
}

try {
    f();//'catch err'
} catch(e) {
    console.log('outer err');//不执行
}
```

⭐进入catch代码块后，一遇到throw语句，就会去执行finally代码块，直接返回，不再回去执行catch代码块剩下部分。

try代码块内部还可以使用try代码块：

```js
try {
    try {
        throw 1;
    } finally {
        console.log(2);
    }
    console.log(3);//不执行
} catch(e){
    console.log(e);
}
//2
//1
```

内层try抛出错误。执行内层finally代码块，然后抛出错误，被外层catch捕获。

⭐finally代码块常用场景：

```js
openFile();

try{
	writeFile(data);
} catch(e) {
	handleError(e);
} finally {
	closeFile();
}
```



### 8. console对象与控制台

console常见用途有：①调试程序，显示网页代码运行时的错误信息。②提供了一个命令行接口，用来与网页代码互动。

- console对象的静态方法

①console.log()、console.info()、console.debug()

在控制台输出信息，接受一个或多个参数，将它们连接起来输出。

console.log支持以下占位符：

%s：字符串、%d：整数、%i：整数、%f：浮点数、%o：对象的链接、%c：CSS格式字符串

```js
console.log('%s %d items','abc',13);//abc 13 items
```

如果参数是一个对象，会显示该对象值：

```js
console.log({x:1});//{ x: 1 }
console.log(Date);//[Function: Date]
```

console.info是console.log别名，console.info方法会在输出信息前加一个蓝色图标。

console.debug方法默认输出信息不显示，只有在打开显示级别在verbose情况下才显示调试信息。

②console.warn()、console.error()

warn方法输出信息时，在最前面加一个黄色三角，表示警告。

error方法输出信息时，在最前面加一个红色叉，表示出错。同时会高亮显示输出文字和错误发生的堆栈。

③console.table()

对于某些复合类型数据，console.table方法可将其转为表格显示。

```js
var table = [{a:1,b:2},{a:3,b:1}];
console.table(table);
```

输出：

| (index) | a    | b    |
| ------- | ---- | ---- |
| 0       | 1    | 2    |
| 1       | 3    | 1    |

④console.count()

用于计数，输出它被调用了多少次

```js
function say(name) {
    console.count(name);
}
say('ann');//ann: 1
say('ann');//ann: 2
```

⑤console.dir()、console.dirxml()

dir方法用于对一个对象检查，并以易于阅读和打印格式显示。

🔺对输出DOM对象很有用，会显示DOM对象的所有属性

```js
console.dir(document.body);
```

dirxml方法主要以目录树形式显示DOM节点

```js
console.dirxml(document.body);
```

如果参数不是DOM节点，而是普通JavaScript对象，console.dirxml方法等同于console.dir方法。

⑥console.assert()

用于程序运行过程中，进行条件判断，如不满足条件就显示一个错误，但不会中断程序执行。就相当于提示用户，内部状态不正确。

接受两个参数，第一个参数是表达式，第二个参数是字符串，只有当第一个参数为false，才提示有错误，在控制台输出第二个参数，否则不会有任何结果。

```js
console.assert(list.childNodes.length < 500, '节点个数大于等于500')
```

⑦console.time()、console.timeEnd()

用于计时，算出一个操作花费的准确时间。

time方法表示计时开始，timeEnd方法表示计时结束。参数是计时器名称。

```js
console.time('time');
var a = 2**12;
console.timeEnd('time');//time: 0.084ms
```

⑧console.group()、console.groupEnd()、console.groupCollapsed()

console.group和console.groupEnd用于将显示的信息分组。只在输出大量信息时有用，分在一组的信息可用鼠标折叠/展开。

```js
console.group('1');//标题
console.log('111');//内容
console.group('2');//标题
console.log('222');//内容
console.groupEnd();//二级分组结束
console.groupEnd();//一级分组结束
/*
1
  111
  2
    222
*/
```

console.groupCollapsed与console.group很类似，唯一区别是该组的内容，在第一次显示时收起而不是展开。

```js
console.groupCollapsed('111');//标题
console.log('11111');//内容
console.groupEnd();
```

⑨console.trace()、console.clear()

console.trace方法显示当前执行的代码在堆栈中的调用路径。

console.clear方法用于清除当前控制台所有输出，将光标回置到第一行。如果用户选中了控制台“Preserve log”选项，console.clear方法不起作用。

- 控制台命令行API

$_：返回上一个表达式的值

$0 ~ $4：控制台保存了最近5个在Elements面板选中的DOM元素，$0代表倒数第一个（最近一个），$1代表倒数第二个，以此类推到$4。

$(selector)：返回第一个匹配的元素，等同于document.querySelector()

$$(selector)：返回选中的DOM对象，等同于document.querySelectorAll。

$x(path)：返回一个数组，包含匹配特定XPath表达式的所有DOM元素。

```js
$x("//p[a]")//返回所有包含a元素的p元素
```

inspect(object)：打开相关面板，并选中相应元素，显示它的细节。

getEventListeners(object)：返回一个对象，该对象的成员为object登记了回调函数的各种事件（如click或keydown），每个事件对应一个数组，数组的成员为该事件回调函数。

keys(object)：返回一个数组，包含object的所有键名。

values(object)：返回一个数组，包含object的所有键值。

monitorEvents(object[, events])，unmonitorEvents(object[, events])：监听特定对象上发生的特定事件。事件发生时返回一个Event对象，包含该事件相关信息。停止监听。

```js
monitorEvents(window, ["resize", "scroll"]) // 对多个事件监听
```

clear()：清除控制台历史

copy(object)：复制特定DOM元素到剪贴板

dir(object)：显示特定对象的所有属性，是console.dir方法别名

dirxml(object)：显示特定对象的XML形式，是console.dirxml方法别名

- debugger语句

主要用于除错，作用是设置断点。如果有正在运行的除错工具，程序运行到debugger语句时自动停下。如没有除错工具，debugger语句不产生任何结果，JavaScript引擎自动跳过这一句。

Chrome浏览器，当代码运行到debugger语句时暂停运行，自动打开脚本源码界面。

```js
for(var i = 0; i < 5; i++){
  console.log(i);
  if (i === 2) debugger;
}
```



### 9. 标准库

**9.1 Math对象**

①静态属性

- Math.E：常熟e
- Math.LN2：2的自然对数
- Math.LN10：10的自然对数
- Math.LOG2E：以2为底的e的对数
- Math.LOG10E：以10为底的e的对数
- Math.PI：圆周率
- Math.SQRT1_2：0.5的平方根
- Math.SQRT2：2的平方根

②静态方法

- Math.abs()：求绝对值
- Math.max()、Math.min()：返回参数之中的最大值/最小值

```js
console.log(Math.max());//-Infinity
console.log(Math.min());//Infinity
```

- Math.floor()、Math.ceil()：返回小于或等于参数值的最大整数/大于或等于参数值的最小整数

- Math.round()：四舍五入，相当于Math.floor(x + 0.5)

```js
console.log(Math.round(-1.5));//-1
console.log(Math.round(1.5));//2
```

- Math.pow()：返回以第一个参数为底数、第二个参数为指数的幂运算值
- Math.sqrt()：返回参数值平方根

```js
console.log(Math.sqrt(-4));//NaN
```

- Math.log()：返回以e为底的自然对数值
- Math.exp()：返回常数e的参数次方
- Math.random()：返回0到1之间的一个伪随机数，可能等于0，但一定小于1

```js
Math.random() * (max - min) + min;//[min,max)区间的随机数
Math.floor(Math.random() * (max - min + 1)) + min;//[min,max)区间的随机整数
```

- Math.sin()、Math.cos()、Math.tan()：返回参数的正弦、余弦、正切（参数为弧度值）
- Math.asin()、Math.acos()、Math.atan()：返回参数的反正弦、反余弦、反正切（返回值为弧度值）

**9.2 Boolean对象**

Boolean对象是JavaScript的三个包装对象之一。作为构造函数，主要用于生成布尔值的包装对象实例。

```js
var bool = new Boolean(false);
console.log(typeof bool);//"object"
console.log(bool.valueOf());//false
if (bool) {
    console.log('1');//1
}
```

🔺false对应的包装对象实例，布尔运算结果为true

使用双重否定运算符( !! )也可以将任意值转为对应布尔值：

```js
console.log(!!0);//false
console.log(!!1);//true
```

9.3 Date对象**

以国际标准时间(UTC)1970年1月1日00：00：00作为时间零点，可表示时间范围是前后各1亿天（单位为ms）

- 普通函数

无论有没有参数，直接调用Date总是返回当前时间。

```js
console.log(Date());//"Thu Jan 07 2021 13:18:33 GMT+0800 (GMT+08:00)"
```

- 构造函数

使用new命令，返回一个Date对象的实例。不加参数，实例代表就是当前时间。Date实例求值时，默认调用toString方法。

Date对象可接受多种格式参数，返回一个该参数对应的时间实例：

```js
console.log(new Date(1688218728000));//时间零点开始计算的ms数
console.log(new Date('June 20,2017'));//日期字符串
console.log(new Date(2016,3,2,0,0,0,0));//年、月、日、小时、分、秒、毫秒
```

🔺①参数可以是负数，代表1970年元旦之前时间。②只要能被Date.parse()方法解析的字符串，都可以当作参数。③参数为年、月、日等多个参数时，年和月不能省略，其他参数可以省略。

各个参数取值范围：

年：四位数年份，如果写成两位数或个位数，则加上1900，如果是负数，表示公元前。

月：0表示一月，以此类推，11表示12月

日：1到31

小时：0到23

分钟：0到59

秒：0到59

毫秒：0到999

🔺如果参数超出正常的范围，会被自动折算。参数还可以是负数，表示扣去的时间：

```js
new Date(2013, 15)// Tue Apr 01 2014 00:00:00 GMT+0800 (CST)
new Date(2013, 0, 0)// Mon Dec 31 2012 00:00:00 GMT+0800 (CST)
new Date(2013, -1)// Sat Dec 01 2012 00:00:00 GMT+0800 (CST)
new Date(2013, 0, -1)// Sun Dec 30 2012 00:00:00 GMT+0800 (CST)
```

- 日期运算

两个日期实例对象减法运算，返回它们间隔的毫秒数。

两个日期实例对象加法运算，返回两个日期字符串连接而成的新字符串。

```js
var d1 = new Date(2010,4,1);
var d2 = new Date(2010,5,1);
console.log(d2-d1);//2678400000
console.log(d2+d1);//Tue Jun 01 2010 00:00:00 GMT+0800 (GMT+08:00)Sat May 01 2010 00:00:00 GMT+0800 (GMT+08:00)
```

- 静态方法

①Date.now()：返回当前时间距离时间零点的毫秒数，相当于Unix时间戳乘以1000.

②Date.parse()：解析日期字符串，返回该时间距离时间零点的毫秒数。如果解析失败，返回NaN。格式一般应满足：`YYYY-MM-DDTHH:mm:ss.sssZ`，Z表示时区。

③Date.UTC()：接受年、月、日等变量为参数，返回该时间距离时间零点的毫秒数。

🔺Date.UTC方法的参数解释为UTC时间，Date构造函数的参数解释为当前时区的时间。

- 实例方法

①Date.prototype.valueOf()：返回实例对象距离时间零点对应毫秒数，等同于getTime方法。

```js
var date = new Date();
console.log(date.valueOf());
console.log(date.getTime());
```

②to类方法

Date.prototype.toString()：返回一个完整日期字符串

Date.prototype.toUTCString()：返回对应的UTC时间，比北京时间晚8小时

Date.prototype.toISOString()：返回UTC时区时间的ISO8601写法

Date.prototype.toJSON()：返回符合JSON格式的ISO日期字符串

Date.prototype.toDateString()：返回日期字符串（不含小时、分和秒）

Date.prototype.toTimeString()：返回时间字符串（不含年月日）

Date.prototype.toLocaleString()：完整的本地时间

Date.prototype.toLocaleDateString()：本地日期（不含小时、分和秒）

Date.prototype.toLocaleTimeString()：本地时间（不含年月日）

③get类方法

getTime()、getDate()、getDay()（返回星期几，星期日为0，星期一为1，以此类推）、getFullYear()、getMonth()、getHours()、getMilliseconds()、getMinutes()、getSeconds()、getTimezoneOffset()（返回当前时间与UTC时区差异，以分钟表示，返回结果考虑夏令时因素）

返回UTC时间：getUTCDate()、getUTCFullYear()、getUTCMonth()、getUTCDay()、getUTCHours()、getUTCMinutes()、getUTCSeconds()、getUTCMilliseconds()

④set类方法

setDate(date)、setFullYear(year[,month,date])、setHours(hour[,in,sec,ms])、setMilliseconds()、setMinutes(min[,sec,ms])、setMonth(month[,date])、setSeconds(sec[,ms])、setTime(milliseconds)

🔺相对于get方法，set方法没有setDay()。set方法的参数会自动折算。

设置UTC时区时间：setUTCDate()、setUTCFullYear()、setUTCHours()、setUTCMilliseconds()、setUTCMinutes()、setUTCMonth()、setUTCSeconds()

**9.4 String对象**

- 实例属性

String.prototype.length：返回字符串长度

- 实例方法

String.prototype.charAt()：返回指定位置的字符。如果参数为负数，charAt返回空字符串

```js
console.log('abcd'.charAt(1));//'b'
console.log('abcd'.charAt(-1));//""
```

String.prototype.charCodeAt()：返回字符串指定位置的Unicode码点。相当于String.fromCharCode()逆操作。如果没有任何参数，返回首字符的Unicode码点。如果参数为负数，或大于等于字符串长度，返回NaN。

```js
console.log('abcd'.charCodeAt(1));//98
console.log('abcd'.charCodeAt(-1));//NaN
```

String.prototype.concat()：连接字符串，返回一个新字符串，不改变原字符串。如果参数不是字符串，concat方法将其先转为字符串再连接。

```js
var num = 2;
var str = 'ab';
console.log(''.concat(str,num));//'ab2'
```

String.prototype.slice()：从原字符串取出子字符串返回，不改变原字符串。如果参数是负值，表示从结尾开始倒数计算的位置，即该负值加上字符串长度。如果第一个参数大于第二个参数（正数情况下），slice()返回空字符串。

```js
console.log('abcde'.slice(2));//'cde'
console.log('abcde'.slice(1,3));//'bc'
console.log('abcde'.slice(0,-2));//'abc'
```

String.prototype.substring()：和slice()很相似，但如果第一个参数大于第二个参数，substring方法会自动更换两个参数位置。如果参数是负数，substring方法会将负数转为0.

```js
console.log('abcde'.substring(4,2));//'cd'
console.log('abcde'.substring(3,-1));//'abc'
```

String.prototype.substr()：和slice()和substring()很相似，但第二个参数是子字符串的长度。如果省略第二个参数，表示子字符串一直到原字符串结束。如果第一个参数是负数，表示倒数计算的字符位置。如果第二个参数是负数，将被自动转为0，返回空字符串。

```js
console.log('abcde'.substr(1,3));//'bcd'
console.log('abcde'.substr(-3));//'cde'
console.log('abcde'.substr(3,-1));//""
```

String.prototype.trim()：去除字符串两端空格，返回新字符串，不改变原字符串。该方法也去除制表符、换行符和回车符。

```js
console.log('   abc '.trim());//'abc'
```

String.prototype.indexOf()：确定一个字符串在另一个字符串中第一次出现的位置，返回匹配开始的位置。如果不匹配返回-1。还可接受第二次个参数，表示从该位置开始向后匹配。

String.prototype.lastIndexOf()：从尾部开始匹配，可接受第二个参数表示从该位置起向前匹配。

```js
console.log('abcdabcdabcd'.indexOf('b'));//1
console.log('abcdabcdabcd'.lastIndexOf('b'));//9
console.log('abcdabcdabcd'.indexOf('b',4));//5
console.log('abcdabcdabcd'.lastIndexOf('b',4));//1
```

String.prototype.toLowerCase()、String.prototype.toUpperCase()：大小写切换。返回新字符串，不改变原字符串。

String.prototype.match()：确定原字符串是否匹配某个子字符串，返回一个数组，成员为匹配的第一个字符串。如果没有匹配返回null。返回的数组有index属性和input属性，分别表示匹配字符串开始的位置和原始字符串。match方法可使用正则表达式作为参数。

```js
var m = 'abc,bbc'.match('bc');
console.log(m);//[ 'bc', index: 1, input: 'abc,bbc', groups: undefined ]
```

String.prototype.search()：基本等同于match()，但返回值为匹配的第一个位置。如果没有匹配，返回-1.search方法可使用正则表达式作为参数。

```js
console.log('abc,bbc'.search('bc'));//1
```

String.prototype.replace()：替换匹配的子字符串，一般情况下只替换第一个匹配（除非使用带有g修饰符的正则表达式）。replace方法可使用正则表达式作为参数。

```js
console.log('aabb'.replace('a','b'));//'babb'
```

String.prototype.split()：按给定规则分割字符串，返回一个由分割出来的子字符串组成的数组。如果省略参数，返回数组的唯一成员是原字符串。如果分割规则为空字符串，返回数组成员是原字符串每一个字符。split方法可接受第二个参数，限定返回数组最大成员数。split方法可使用正则表达式作为参数。

```js
console.log('a|b|c'.split('|'));//[ 'a', 'b', 'c' ]
console.log('a|b|c'.split(''));//[ 'a', '|', 'b', '|', 'c' ]
console.log('a|b|c'.split());//[ 'a|b|c' ]
console.log('a||c'.split('|'));//[ 'a', '', 'c' ]
console.log('|b|c'.split('|'));//[ '', 'b', 'c' ]
console.log('a|b|'.split('|'));//[ 'a', 'b', '' ]
console.log('a|b|c'.split('|',2));//[ 'a', 'b' ]
```

String.prototype.localeCompare()：比较两个字符串。返回一个整数，如果小于0，表示第一个字符串小于第二个字符串；如果等于0，表示两者相等；如果大于0，表示第一个字符串大于第二个字符串。该方法会考虑自然语言的顺序。

```js
console.log('B'.localeCompare('a'));//1
```

**9.5 Number对象**

数值对应的包装对象，可作为构造函数使用，也可做工具函数使用。

```js
var n = new Number(1);
console.log(typeof n);//"object"
```

- 静态属性

Number.POSITIVE_INFINITY：正的无限，指向Infinity

Number.NEGATIVE_INFINITY：负的无限，指向-Infinity

Number.NaN：非数值，指向NaN

Number.MIN_VALUE：最小的正数，即最接近0的正数，-Number.MIN_VALUE：最接近0的负数

Number.MAX_SAFE_INTEGER：能够精确表示的最大整数

Number.MIN_SAFE_INTEGER：能够精确表示的最小整数

- 实例方法

Number.prototype.toString()：将数值转为字符串。可接受一个参数，表示输出的进制。可直接对一个小数使用toString方法。通过方括号运算符也可调用toString方法。

```js
console.log((20).toString(2));//"10100"
console.log(10.5.toString());//"10.5"
console.log(12['toString'](16));//"c"
```

Number.prototype.toFixed()：将一个数转为指定位数小数，然后返回这个小数对应字符串。参数有效范围是0到100。由于浮点数原因，小数5四舍五入不确定，小心使用。

```js
console.log((10).toFixed(1));//"10.0"
console.log(10.005.toFixed(2));//"10.01"
console.log(10.055.toFixed(2));//"10.05"
```

Number.prototype.toExponential()：将一个数转为科学计数法。参数是小数点后有效数字位数，范围0到100。

```js
console.log((123).toExponential(1));//"1.2e+2"
```

Number.prototype.toPrecision()：将一个数转为指定位数的有效数字。参数为有效数字位数，范围1到100。由于浮点数不是精确储存，所以该方法用于四舍五入不太可靠。

```js
console.log((123.45).toPrecision(4));//"123.5"
```

Number.prototype.toLocaleString()：接受一个地区码作为参数，返回字符串，表示当前数字在该地区的当地书写形式。还可接受第二个参数配置对象，style属性默认decimal，以十进制形式输出，可选percent，表示输出百分数。style属性设为currency，则可搭配currency属性，输出指定格式的货币字符串形式。

```js
(123).toLocaleString('zh-Hans-CN-u-nu-hanidec') // "一二三"
(123).toLocaleString('zh-Hans-CN', { style: 'percent' }) // "12,300%"
(123).toLocaleString('en-US', { style: 'currency', currency: 'USD' }) // "$123.00"
```

- 自定义方法

与其他对象一样，Number.prototype对象上可自定义方法，被Number的实例继承。

```js
Number.prototype.add = function (x) {
    return this + x;
};
Number.prototype.multiply = function (x) {
    return this * x;
};
console.log(((10).add(3)).multiply(2));//26
```

**9.6 Array对象**

- 构造函数

参数是一个正整数，返回数组的成员都是空位，虽然读取时返回undefined，但实际上该位置没有任何值。这时可读取到length属性，但取不到键名。

```js
var arr1 = new Array(2);
console.log(arr1[1]);//undefined
console.log(0 in arr1);//false
```

- 静态方法

Array.isArray() 返回布尔值，表示参数是否为数组。

```js
var arr = [1,2,3];
console.log(typeof arr);//"object"
console.log(Array.isArray(arr));//true
```

- 实例方法

valueOf()、toString()：数组valueOf()方法返回数组本身，toString()返回数组的字符串形式。

```js
console.log([1,2].valueOf());//[1,2]
console.log([1,2,[3,4]].toString());//"1,2,3,4"
```

push()：在数组末端添加一个或多个元素，返回添加新元素后的数组长度，会改变原数组。

pop()：删除数组最后一个元素，并返回该元素，会改变原数组。

对空数组使用pop()，返回undefined。

🔺push()和pop()结合使用实现了 栈 

```js
var arr = [1,2,3];
arr.push(4,5);//5
arr.pop();//5
```

shift()：删除数组第一个元素，并返回元素，会改变原数组。

unshift()：在数组第一个位置添加元素，返回添加新元素后数组长度，会改变原数组。

🔺push()和shift()结合使用实现了 队列

```js
var arr = [1,2,3];
arr.shift();//1
arr.unshift(1,2);//4
```

join()：以指定参数为分隔符，将所有数组成员连接为一个字符串返回。如不提供参数，默认逗号分隔。

如果数组成员是undefined、null或空位，会被转成空字符串。

通过call方法，这方法也可用于字符串或类似数组的对象。

```js
var arr = [1,2,3];
console.log(arr.join('|'));//1|2|3
console.log([undefined,null].join(','));//','
console.log(Array.prototype.join.call('abc',','));//'a,b,c'
```

concat()：多个数组合并，将新数组成员添加到原数组成员后部，返回新数组，原数组不变。参数也可以是其他类型值。

🔺如果数组成员包括对象，该方法返回当前数组的一个浅拷贝。新数组拷贝的是对象的引用。

```js
var arr = [1,2,3];
console.log(arr.concat([1,2]));//[1,2,3,1,2]
console.log(arr.concat(5));//[1,2,3,5](原数组不变)

var oldArr = [{a:2}];
var newArr = oldArr.concat();
oldArr[0].a = 1;
console.log(newArr[0].a);//1
```

reverse()：颠倒排列数组元素，返回新数组，会改变原数组。

```js
var arr = [1,2,3];
console.log(arr.reverse());//[3,2,1]
console.log(arr);//[3,2,1]
```

slice()：提取目标数组一部分，返回新数组，原数组不变。重要应用是将类似数组的对象转为真正数组。

```js
var arr = [1,2,3];
console.log(arr.slice(1));//[2,3]
console.log(arr.slice(0,2));//[1,2]
console.log(arr.slice(-2,-1));//[2]
console.log(arr.slice(5));//[]
```

splice()：删除原数组一部分成员，并可在删除位置添加新的数组成员，返回值是被删除元素，会改变原数组。

`arr.splice(start,count,addElement1,addElement2)`

如果单纯插入元素，第二个参数可设为0。如果只提供第一个参数，等同于将原数组在指定位置拆分成两个数组。

```js
var arr = [1,2,3];
arr.splice(1,0,5,6,7);
console.log(arr);//[1,5,6,7,2,3]
arr.splice(2,2);
console.log(arr);//[1,5,2,3]
arr.splice(-3,1);
console.log(arr);//[1,2,3]

console.log(arr.splice(2));//[3]
console.log(arr);//[1,2]
```

sort()：排序数组，原数组会改变。默认按照字典序排列。可自定义排序方法。

```js
console.log([2,4,3].sort());//[2,3,4]
console.log([2,4,3].sort((a,b) => b-a));//[4,3,2]
console.log([{a:1,b:2},{a:3,b:1},{a:2,b:3}].sort((a,b) => a.b - b.b));
//[ { a: 3, b: 1 }, { a: 1, b: 2 }, { a: 2, b: 3 } ]
```

map()：将数组所有成员自此传入参数函数，把每一次执行结果组成一个新数组返回。map方法接受一个函数作为参数，传入三个参数：当前成员、当前位置和数组本身。map方法还可接受第二个参数，用来绑定回调函数内部的this变量。如果数组有空位，map方法回调函数不会在这个位置执行，会跳过空位。

```js
console.log([1,2].map((elem,index,arr) => {
    return elem * index *2;
}));//[0,4]

var arr = ['a','b','c'];
console.log([1,2].map(function (e) {
    return this[e];
},arr));//['b','c']

console.log([1,,2].map(item => 'a'));//['a',,'a']
```

forEach()：参数是一个函数，接受三个参数：当前值、当前位置、整个数组。可接受第二个参数，绑定参数函数的this变量。会跳过数组空位。

```js
[2,4,6].forEach((elem,index,arr) => {
    console.log(elem * (index+1));//2 8 18
});

[1,,2].forEach(item => console.log(item+1));//2 3

var output = [];
[1,2].forEach(function(e) {
    this.push(e**e);
},output);
console.log(output);//[1,4]
```

🔺如果数组遍历目的是为了得到返回值，就使用map方法，否则使用forEach方法。

filter()：过滤数组成员，满足条件的成员组成一个新数组返回。参数是一个函数，返回结果为true的成员组成新数组返回，不改变原数组。可接受三个参数：当前成员、当前位置和整个数组。可接受第二个参数，绑定参数函数内部this变量。

```js
console.log([1,3,5,7].filter(item => item>4));//[5,7]

var flag = {num:5};
var arr = [1,3,7,9];
console.log(arr.filter(function(item) {
    if (item > this.num) return item;
},flag));//[7,9]

console.log([1,2,3].filter((elem,index,arr) => {
    return index === 1;//[2]
}));
```

some()，every()：返回布尔值，表示判断数组成员是否符合某种条件。接受函数参数，该函数接受三个参数：当前成员、当前位置和整个数组。some方法只要一个成员返回值true，整个方法返回true。every方法所有成员返回值true才返回true。还可接受第二个参数，绑定参数函数内部this变量。

🔺空数组，some方法返回false，every方法返回true，回调函数都不会执行。

```js
console.log([1,2,3].some(item => item > 4));//false
console.log([].some(item => item < 2));//false
console.log([1,2,3].every(item => item < 4));//true
console.log([].every(item => item < 2));//true
```

reduce()、reduceRight()：依次处理数组每个成员，最终累计为一个值。reduce()从左到右处理，reduceRight()从右到左处理。第一个参数都是函数，函数可接受4个参数：累积变量，默认为数组第一个成员；当前变量，默认为数组第二个成员；当前位置（从0开始）；原数组。前两个参数必选，后两个参数可选。

如果要对累积变量设初值，可放在方法的第二个参数。

```js
console.log([1,2,3,4].reduce((a,b) => {
    console.log(a,b);
    return a+b;
}));
//1 2
// 3 3
// 6 4
// 10

console.log([1,2,3].reduce((a,b) => a-b));//-4
console.log([1,2,3].reduceRight((a,b) => a-b));//0

console.log([1,2,3].reduce((a,b) => {
    return a+b;
},20));//26
```

indexOf()、lastIndexOf()：返回给定元素在数组第一次/最后一次出现位置，找不到返回-1。indexOf()还可接受第二个参数，表示搜索开始位置。

```js
console.log([1,2,3,1].indexOf(1));//0
console.log([1,2,3,1].lastIndexOf(1));//3
console.log([NaN].indexOf(NaN));//-1
console.log([NaN].lastIndexOf(NaN));//-1
```

🔺不能搜索NaN的位置，无法确定数组成员是否包含NaN。廷尉方法内部使用严格相等运算符作比较。

⭐链式调用，当上述方法返回的还是数组可链式调用。

```js
var stu = [{name:'a',age:23},{name:'c',age:25}];
stu.map(s => {
    return s.age;
}).forEach(s => console.log(s));//23 25
```

**9.7 JSON对象**

- JSON格式

JSON（JavaScript Object Notation）是一种用于数据交换的文本格式，目的是取代繁琐笨重的XML格式。

两大优点：书写简单，一目了然；符合JavaScript原生语法，可由解释引擎直接处理，不用另外添加解析代码。

每个JSON对象就是一个值，可以是一个数组或对象，也可以是一个原始类型的值。总之，只能是一个值，不能是两个或更多的值。

🔺JSON对值的类型和格式有严格规定：

①复合类型值只能数组或对象，不能是函数、正则表达式对象、日期对象

②原始类型值只有四种：字符串、数值（十进制）、布尔值和null

③字符串必须使用双引号表示

④对象的键名必须在双引号里

⑤数组或对象最后一个成员的后面，不能加逗号

- JSON对象

处理JSON格式数据。有两个静态方法：

①JSON.stringify()：将一个值转为JSON字符串，该字符串符合JSON格式，并可被JSON.parse()还原。

```js
console.log(JSON.stringify(false));//"false"
console.log(JSON.stringify('false'));//""false""
```

如果对象的属性是undefined、函数或XML对象，该属性会被JSON.stringify()过滤。

```js
var obj = {a: undefined};
console.log(JSON.stringify(obj));//"{}"
```

如果数组成员是undefined、函数或XML对象，这些值被转为null。

```js
var arr = [undefined];
console.log(JSON.stringify(arr));//"[null]"
```

正则对象会被转为空对象。

JSON.stringify()方法会忽略对象的不可遍历的属性。

该方法还可接受一个数组作为第二个参数，指定参数对象的哪些属性需要转成字符串。

```js
var obj = { 'a': 1, 'b':2};
console.log(JSON.stringify(obj,['a']));//"{"a":1}"
```

第二个参数还可以是一个函数，更改JSON.stringify()的返回值。这个处理函数是递归处理所有的键。递归处理中，每一次处理的对象都是前一次返回的值。如果处理函数返回undefined或没有返回值，该属性会被忽略。

```js
var obj = {1: {2: 3}};
function f(k,v) {
    console.log(k,v);
    return v;
}
console.log(JSON.stringify(obj,f));
// { '1': { '2': 3 } }
// 1 { '2': 3 }
// 2 3
// {"1":{"2":3}}
```

该方法还可接受第三个参数，增加返回的JSON字符串可读性。第三个参数使得每个属性单独占据一行，并且将每个属性前面添加指定前缀（不超过10个字符）：

```js
console.log(JSON.stringify({a:2,b:3},null,'\t'));
//{
//	"a": 2,
//	"b": 3
//}
```

如果参数对象有自定义toJSON方法，那么JSON.stringify()会使用这个方法的返回值作为参数，而忽略原对象其他属性。

toJSON方法一个应用是，将正则对象转为字符串，因为JSON.stringify()方法默认不能转换正则对象。

```js
var obj = {
    1:123,
    2:234,
    toJSON:function() {
        return {
            3:this['1']+this['2'],
        };
    }
};
console.log(JSON.stringify(obj));//"{"3":357}"
```

```js
RegExp.prototype.toJSON = RegExp.prototype.toString;
console.log(JSON.stringify(/f/));//""/f/""
```

②JSON.parse()：将JSON字符串转为对应的值。该方法可接受一个处理函数作为第二个参数，用法与JSON.stringify()类似

```js
console.log(JSON.parse('"abc"'));//"abc"
console.log(JSON.parse('abc'));//error
```

```js
function f(k,v) {
    if (k === "1"){
        return v+2;
    }
    return v;
}
console.log(JSON.parse('{"1":2,"2":2}',f));//{'1':4,'2':2}
```



**9.8 Object对象**

JavaScript所有其他对象都继承自Object对象，即那些对象都是Object的实例。

Object对象的原生方法包括：①Object对象本身的方法（直接定义在Object对象的方法）、②Object的实例方法（定义在Object原型对象Object.prototype上的方法，可被Object实例直接使用）

- Object()

可将任意值转为对象。如果参数为空（或undefined和null），Object()返回一个空对象。

instanceof运算符用于验证一个对象是否为指定的构造函数的实例。

🔺如果参数是原始类型值，Object方法将其转为对应的包装对象的实例。如果参数是一个对象，它总是返回该对象，即不用转换。

```js
var obj = Object('abc');
console.log(obj instanceof Object);//true
console.log(obj instanceof String);//true
var arr = [];
var obj1 = Object(arr);
console.log(obj1 instanceof Array);//true
console.log(obj1 instanceof Object);//true
console.log(obj1 === arr);//true
```

- Object构造函数

用途是直接生成一个新对象。

```js
var obj = new Object();//等价于 var obj = {};
```

构造函数用法与上面的工具方法用法相似。不同在于Object(value)表示将value转成一个对象，new Object(value)表示新生成一个对象。

- Object静态方法

Object.keys()返回对象可枚举属性。Object.getOwnPropertyNames()还返回对象不可枚举属性。

```js
var arr = [1,2];
console.log(Object.keys(arr));//[ '0', '1' ]
console.log(Object.getOwnPropertyNames(arr));//[ '0', '1', 'length' ]
```

对象属性模型相关方法：

Object.getOwnPropertyDescriptor()：获取某个属性描述对象

Object.defineProperty()：通过描述对象，定义某个属性

Object.defineProperties()：通过描述对象，定义多个属性

控制对象状态的方法：

Object.preventExtensions()：防止对象扩展

Object.isExtensible()：判断对象是否可扩展

Object.seal()：禁止对象配置

Object.isSealed()：判断一个对象是否可配置

Object.freeze()：冻结一个对象

Object.isFrozen()：判断一个对象是否被冻结

原型链相关方法：

Object.create()：指定原型对象和属性，返回一个新对象

Object.getPrototypeOf()：获取对象的Prototype对象

- Object实例方法

Object.prototype.valueOf()：返回当前对象对应的值。默认情况下返回对象本身。可自定义valueOf方法覆盖Object.prototype.valueOf.

```js
var obj = new Object();
console.log(obj === obj.valueOf());//true
obj.valueOf = function() {
    return 3;
};
console.log(1+obj);//4
```

Object.prototype.toString()：返回一个对象的字符串形式。默认情况返回类型字符串。可自定义toString方法。

```js
var o = {a:2,b:2};
console.log(o.toString());//"[object Object]"
var obj = new Object();
obj.toString = function () {
    return '123';
};
console.log(obj.toString());//'123'
```

🔺利用toString方法判断数据类型：

```js
var type = function(o){
    var s = Object.prototype.toString.call(o);
    console.log(s.match(/\[object (.*?)\]/)[1].toLowerCase());
};
type([]);//"array"
type(function(){});//"function"
```

Object.prototype.toLocaleString()：返回一个值的字符串形式。作用是留出一个接口，让各种不同对象实现自己版本的toLocaleString，返回针对某些地域的特定的值。

主要有三个对象自定义了toLocaleString方法：Array、Number、Date

Object.prototype.hasOwnProperty()：接受字符串作为参数，返回布尔值，表示该实例对象自身是否有该属性。

```js
var obj = {a:1};
console.log(obj.hasOwnProperty('a'));//true
console.log(obj.hasOwnProperty('toString'));//false，继承属性
```

Object.prototype.isPrototypeOf()：判断当前对象是否为另一个对象的原型。

Object.prototype.propertyIsEnumerable()：判断某个属性是否可枚举。

**9.9 属性描述对象**

每个属性都有自己对应的属性描述对象，保存该属性的一些元信息。

```js
{
  value: 123,
  writable: false,
  enumerable: true,
  configurable: false,
  get: undefined,
  set: undefined
}
```

如上例，属性描述对象提供6个元属性。

value：属性值，默认undefined。

writable：布尔值，属性值是否可改变（是否可写），默认true

enumerable：布尔值，属性是否可遍历，默认true。如设为false，使得如for...in循环、Object.keys()跳过该属性。

configurable：布尔值，可配置属性，默认true。如设为false，将阻止某些操作改写该属性，如无法删除该属性，也不得改变该属性的属性描述对象（value属性除外）。configurable属性控制了属性描述对象可见性。

get：函数，该属性取值函数（getter），默认undefined

set：函数，该属性存值函数（setter），默认undefined

- Object.getOwnPropertyDescriptor()

获取属性描述对象。第一个参数是目标对象，第二个参数是一个字符串，对应目标对象某个属性名。

🔺只能用于对象自身属性，不能用于继承属性。

```js
var obj = {a:1};
console.log(Object.getOwnPropertyDescriptor(obj,'a'));
//{ value: 1, writable: true, enumerable: true, configurable: true }
```

- Object.getOwnPropertyNames()

返回数组，成员是参数对象自身的全部属性的属性名，不管该属性是否可遍历。

Object.keys只返回对象自身可遍历属性的全部属性名。

```js
var obj = Object.defineProperties({},{
    a1:{value:2,enumerable:true},
    a2:{value:3,enumerable:false},
});
console.log(Object.keys(obj));//['a1']
console.log(Object.getOwnPropertyNames(obj));//['a1','a2']
```

- Object.defineProperty()、Object.defineProperties()

允许通过属性描述对象，定义或修改一个属性，然后返回修改后的对象。

如果一次性定义或修改多个属性，可使用Object.defineProperties()。

🔺一旦定义了取值函数get或存值函数set，就不能将writable属性设为true，或同时定义value属性，否则报错。

🔺这两个方法参数里的属性描述对象，writable、configurable、enumerable属性默认值都是false

```js
Object.defineProperty(object,propertyName,attributesObject)
```

```js
var obj1 = Object.defineProperty({},'a',{
    value:111,
    writable:false,
});
console.log(obj1.a);//111
obj1.a = 234;
console.log(obj1.a);//111,不可写
```

```js
var obj1 = Object.defineProperties({},{
    a1: { value:1,enumerable:true},
    a2: { get: function () {
            return this.a1 + 2;
        },
        enumerable: true
    }
});
console.log(obj1.a2);//3
```

- Object.prototype.propertyIsEnumerable()

返回布尔值，判断某个属性是否可遍历。只能用于判断对象自身属性，对于继承的属性一律返回false。

```js
var obj = {p:1};
console.log(obj.propertyIsEnumerable('p'));//true
```

- 元属性

属性描述对象的各个属性称为“元属性”，可看作控制属性的属性。

value：目标属性的值

writable：决定了目标属性的值是否可以被改变

正常模式下，对writable为false的属性赋值不报错，只是失败。严格模式下会报错，即使对属性重新赋予一个同样的值。

如果原型对象某个属性writable为false，子对象无法自定义这个属性。规避方法如下：通过覆盖属性描述对象绕过限制。因为原型链会被完全忽视。

```js
var proto = Object.defineProperty({}, 'foo', {
  value: 'a',
  writable: false
});

var obj = Object.create(proto);
Object.defineProperty(obj, 'foo', {
  value: 'b'
});

obj.foo // "b"
```

enumerable：表示目标属性是否可遍历

如果一个属性的enumerable为false，for...in循环、Object.keys()方法和JSON.stringify()方法不会取到该属性。

configurable：决定了是否可以修改属性描述对象

configurable为false时，value、writable、enumerable、configurable都不能被修改

可配置性决定了目标属性是否可以被删除。

🔺①writable只有在false改为true会报错，true改为false是允许的。②只要writable和configurable其中一个为true，value就允许改动。③writable为false时，直接目标属性赋值不会报错，但不会成功。

- 存取器

```js
// 写法1
var obj = Object.defineProperty({},'a',{
    get:function () {
        return 'get';
    },
    set:function (v) {
        console.log('set: '+v);
    }
});
console.log(obj.a);//'get'
obj.a = 111;//'set: 111'
```

```js
// 写法2
var obj = {
    get a() {
        return 'get';
    },
    set a(v) {
        console.log('set: '+v);
    }
};
console.log(obj.a);//'get'
obj.a = 111;//'set: 111'
```

写法1：属性a的configurable和enumerable都为false，a不可遍历。

写法2：属性a的configurable和enumerable都为true，a可遍历。更常用。

🔺取值函数get不能接受参数，存值函数set只能接受一个参数（属性值）

🔺存取器一般用于属性值依赖对象内部数据的场合：

```js
var obj ={
    $n : 5,
    get next() { return this.$n++ },
    set next(n) {
        if (n >= this.$n) this.$n = n;
        else throw new Error('新的值必须大于当前值');
    }
};
obj.next = 3;//error
```

- 对象的拷贝

需要将一个对象所有属性拷贝到另一对象：

```js
var extend = function (to, from) {
    for (var property in from) {
        if (!from.hasOwnProperty(property)) continue;
        Object.defineProperty(
            to,
            property,
            Object.getOwnPropertyDescriptor(from, property)
        );
    }

    return to;
};
```

- 控制对象状态

三种冻结方法，由弱到强：Object.preventExtensions、Object.seal、Object.freeze。

Object.preventExtensions()使得一个对象无法添加新属性。

```js
var obj = {a:1};
Object.preventExtensions(obj);
console.log(Object.isExtensible(obj));//false
```

Object.seal()使得一个对象无法添加新属性，也无法删除旧属性。实质是把属性描述对象的configurable属性设为false。

该方法只是禁止新增或删除属性，并不影响修改某个属性值。

```js
var obj = {a:1};
Object.seal(obj);
delete obj.a;
console.log(obj.a);//1
obj.b = 2;
console.log(obj.b);//undefined
console.log(Object.isSealed(obj));//true
```

Object.freeze()：使得一个对象无法添加新属性、无法删除旧属性、也无法改变属性值，使得这个对象实际上变成了常量。

```js
var obj = {a:1};
Object.freeze(obj);
delete obj.a;
console.log(obj.a);//1
obj.b = 2;
console.log(obj.b);//undefined
obj.a = 3;
console.log(obj.a);//1
console.log(Object.isFrozen(obj));//true
```

🔺局限性

可通过改变原型对象，来为对象增加属性。一种解决方案是把obj原型也冻结住。

```js
var obj = {};
Object.preventExtensions(obj);
var proto = Object.getPrototypeOf(obj);
proto.a = 2;
console.log(obj.a);//2

Object.preventExtensions(proto);
proto.b = 2;
console.log(obj.b);//undefined
```

如果属性值是对象，上面这些方法只能冻结属性指向的对象，而不能冻结对象本身的内容。

```js
var obj = {arr:[1,2,3]};
Object.freeze(obj);
obj.arr.push(4);
console.log(obj.arr);//[1,2,3,4]
```

**9.10 包装对象**

三种原始类型值——数值、字符串、布尔值，在一定条件下可自动转为对象，即原始类型的“包装对象”。

包装对象，使得“对象”这种类型可覆盖JavaScript所有值，整门语言有一个通用数据模型，其次是使得原始类型值也有办法调用自己的方法。

🔺这三个对象作为构造函数使用（带new）时，可将原始类型值转为对象；作为普通函数使用（不带new）时，可将任意类型值转为原始类型值。

三种包装对象共同具有、从Object对象继承的方法：valueOf()、toString()。

🔺某些场景，原始类型值会自动当作包装对象调用，JavaScript引擎会自动将原始类型值转为包装对象实例，并在使用后立刻销毁实例。自动转换生成的包装对象是只读的，无法修改。

- 自定义方法

包装对象可自定义方法和属性，供原始类型值直接调用。

```js
String.prototype.user = function() {
    return this.toString()+'1';
};
console.log('abc'.user());//"abc1"
```

**9.11 RegExp对象**

正则表达式是一种表达文本模式（字符串结构）的方法。JavaScript的正则表达式体系参照Peri 5建立。

新建正则表达式两种方法：

①字面量，以斜杠表示开始和结束。在引擎编译代码时就新建，便利直观，效率更高。

```js
var regex = /xyz/;
```

②RegExp构造函数，运行时新建。

```js
var regex = new RegExp('xyz');
```

构造函数还可接受第二个参数，表示修饰符：

```js
var regex = new RegExp('xyz','i');
//等价于
var regex = /xyz/i;
```

- 实例属性

与修饰符相关：

RegExp.prototype.ignoreCase：返回布尔值，表示是否设置了i修饰符

RegExp.prototype.global：返回布尔值，表示是否设置了g修饰符

RegExp.prototype.multiline：返回布尔值，表示是否设置了m修饰符

RegExp.prototype.flags：返回字符串，包含已经设置的所有修饰符，按字母排序

这四个属性都是只读的。

与修饰符无关：

RegExp.prototype.lastIndex：返回整数，表示下一次开始搜索的位置。属性可读写，但只在进行连续搜素时有意义。

RegExp.prototype.source：返回正则表达式的字符串形式（不包含反斜杠），该属性只读。

- 实例方法

RegExp.prototype.test()：返回布尔值，表示当前模式是否能匹配参数字符串。

如果正则表达式带g修饰符，每一次test方法都从上一次结束位置开始向后匹配。带g修饰符时，可通过lastIndex属性指定开始搜索位置。带g修饰符时，正则表达式内部会记住上一次lastIndex属性，不应更换所要匹配的字符串。

🔺如果正则模式是空字符串，则匹配所有字符串。

```js
var r = /x/g;
var s = '_x_x';
console.log(r.lastIndex);//0
console.log(r.test(s));//true
console.log(r.lastIndex);//2
console.log(r.test(s));//true
console.log(r.lastIndex);//4
console.log(r.test(s));//false
```

RegExp.prototype.exec()：返回匹配结果。如果发现匹配就返回数组，成员是匹配成功的子字符串，否则返回null。

如果正则表达式包含圆括号（即含有“组匹配”），返回数组包括多个成员。第一个成员是整个匹配成功的结果，后面成员是圆括号对应的匹配成功的组。即第二个成员对应第一个括号，第三个成员对应第二个括号，依此类推。整个数组length属性等于组匹配数量+1。

exec()返回数组包含以下两个属性：input，整个字符串。index：模式匹配成功的开始位置。

```js
var reg = /a/g;
var str = 'abc_abc_abc_ab';
while(true) {
    var match = reg.exec(str);
    if (!match) break;
    console.log(match.index+':'+match+':'+reg.lastIndex);
}
// 0:a:1
// 4:a:5
// 8:a:9
// 12:a:13
```

- 字符串的实例方法

String.prototype.match()：返回数组，成员是所有匹配的子字符串。

match方法和正则对象的exec方法很类似。但如果正则表达上带g修饰符，match方法会一次性返回所有匹配成功结果。设置正则表达式lastIndex属性，对match方法无效，匹配总是从字符串第一个字符开始。

```js
var s = 'abbba';
var r = /a/g;
console.log(s.match(r));//[ 'a', 'a' ]
console.log(r.exec(s));//[ 'a', index: 0, input: 'abbba', groups: undefined ]
```

```js
var r = /a/g;
r.lastIndex = 3;
console.log('cacb'.match(r));//['a']
```

String.prototype.search()：返回第一个满足条件的匹配结果在整个字符串中位置。如没有匹配，返回-1。

```js
console.log('_x_x'.search(/x/));//1
```

String.prototype.replace()：替换匹配的值。接受两个参数，第一个正则表达式，第二个替换的内容。

正则表达式如不带g修饰符，替换第一个匹配成功的值，否则替换所有匹配成功的值。

```js
console.log('aaa'.replace(/a/,'b'));//"baa"
console.log('aaa'.replace(/a/g,'b'));//"bbb"
```

replace方法一个应用是消除字符串首尾两端空格：

```js
var str = '   abc  ';
console.log(str.replace(/^\s+|\s+$/g,''));//"abc"
```

replace方法第二个参数可使用美元符号$，指代所替换的内容。

$&：匹配的子字符串

$`：匹配结果前面的文本

$'：匹配结果后面的文本

$n：匹配成功的第n组内容，n从1开始

$$：指代美元符号$

```js
console.log('abcb'.replace('b','[$` - $& - $\']'));//'a[a - b - cb]cb'
```

replace第二个参数还可是一个函数，将每一个匹配内容替换为函数返回值。

replace方法第二个参数的替换函数可接受多个参数：第一个参数是捕捉到的内容，第二个参数是捕捉到的组匹配（有多少个组匹配，就有多少个对应参数）。此外，最后还可添加两个参数，倒数第二个参数是捕捉到的内容在整个字符串中位置（如从第五个位置开始），最后一个参数是原字符串。

```js
var prices = {
    'p1': '$1.99',
    'p2': '$9.99',
    'p3': '$5.00'
};

var template = '<span id="p1"></span>'
    + '<span id="p2"></span>'
    + '<span id="p3"></span>';

console.log(template.replace(
    /(<span id=")(.*?)(">)(<\/span>)/g,
    function(match, $1, $2, $3, $4){
        return $1 + $2 + $3 + prices[$2] + $4;
    }
));
//<span id="p1">$1.99</span><span id="p2">$9.99</span><span id="p3">$5.00</span>
```

四个括号，产生四个组匹配，用$1 - $4表示。

- String.prototype.split()：按照正则规则分割字符串，返回一个由分割后的各个部分组成的数组。

接受两个参数，第一个参数正则表达式，表示分割规则，第二个参数返回数组最大成员数。

```js
console.log('aaa*a*'.split(/a*/));//[ '', '*', '*' ]
console.log('aaa**a*'.split(/a*/));//[ '', '*', '*', '*' ]
```

分割规则是0次或多次a，正则默认是贪婪匹配，第一行分隔符分别是aaa、a，包含开始处空字符串。第二行分隔符是aaa、0个a（空字符串）、a。

如果正则表达式带括号，则括号匹配部分也作为数组成员返回。

```js
console.log('aaa*a*'.split(/(a*)/));//[ '', 'aaa', '*', 'a', '*' ]
```

- 匹配规则

①字面量字符与元字符

字面量：

```js
console.log(/abc/.test('bad abc'));//true
```

点字符：匹配除回车、换行、行分隔符、段分隔符以外所有字符。

```js
console.log(/a.t/.test('a3t'));//true
```

匹配a和t之间包含任意一个字符情况

位置字符：^表示字符串开始位置，$表示字符串结束位置。

```js
console.log(/^a/.test('a3t'));//true
console.log(/bc$/.test('abc'));//true
console.log(/^bc$/.test('bcabc'));//false，从开始位置到结束位置只有bc
```

选择符：竖线符号 | 表示或关系OR，多个选择符可联合使用

```js
console.log(/a(b|c)|cd/.test('zac'));//true
```

②转义符

12个字符(^、.、[、$、(、)、|、*、+、?、{、和\)需要反斜杠转义。如果使用RegExp方法生成正则对象，转义需要用两个斜杠，因为字符串内先转义一次。

```js
console.log(/2\+2/.test('2+2'));//true
console.log(new RegExp('2\\+2').test('2+2'));//true
```

③特殊字符

\cX：表示Ctrl-[X]，X是指A-Z之中任一英文字母，用来匹配控制字符

[\b]：匹配退格键

\n：匹配换行键

\r：匹配回车键

\t：匹配制表符

\v：匹配垂直制表符

\f：匹配换页符

\0：匹配null字符

\xhh：匹配一个以两位十六进制数表示的字符

\uhhhh：匹配一个以四位十六进制数表示的Unicode字符

④字符类

[abc]表示a、b、c、之中任选一个匹配。

脱字符^：除了字符类之中字符，其他字符都可以匹配。脱字符只有在字符类第一个位置才有含义。

方括号内没有其他字符（[^]）表示匹配一切字符，包括换行符。相比之下，点号作为元字符不包括换行符。

```js
console.log(/[abc]/.test('ad'));//true
console.log(/[^abc]/.test('abc'));//false
console.log(/[^]/.test('\n'));//true
```

连字符-：连续序列字符，表示字符连续范围。只有当连字号用在方括号中才表示连续字符序列。

```js
console.log(/[0-9]/.test('2'));//true,表示0到9范围
```

⑤预定义模式：

\d：匹配0到9之间任一数字，相当于[0-9]

\D：匹配0到9以外字符

\w：匹配任意字母、数组和下划线，相当于[A-Za-z0-9_]

\W：除所有字母、数字和下划线以外字符

\s：匹配空格（包含换行符、制表符、空格符），相当于[ \t\r\n\v\f]

\S：匹配非空格的字符

\b：匹配词的边界，词首必须独立

\B：匹配非词边界，即在词的内部

```js
console.log(/\s\w*/.exec('abc def'));//[ ' def', index: 3, input: 'abc def', groups: undefined ]
console.log(/\btest/.test('a test'));//true
console.log(/\btest/.test('atest'));//false
console.log(/\btest/.test('a-test'));//true
```

⑥重复类

模式的精确匹配次数，使用大括号表示。{n}表示恰好重复n次,{n.}表示至少重复m次，{n,m}表示重复不少于n次，不多于m次。

```js
console.log(/ab{3}c/.test('abbbc'));//true
console.log(/a{2,4}b/.test('ab'));//false
```

⑦量词符

设定某个模式出现次数

?问号表示某个模式出现0次或1次，相当于{0,1}。

*星号表示某个模式出现0次或多次，相当于{0,}。

+加号表示某个模式出现1次或多次，相当于{1,}。

```js
console.log(/a?bc/.test('abc'));//true
console.log(/a*bc/.test('bc'));//true
console.log(/a+bc/.test('bc'));//false
```

⑧贪婪模式

上面三个量词符，默认情况都是最大可能匹配，即匹配到下一个字符不满足匹配规则为止。

还有非贪婪模式，最小可能匹配，只要一发现匹配，就返回结果。可以在量词符后面加一个?问号表示非贪婪模式。

```js
console.log('abbbb'.match(/ab+/));//['abbbb']
console.log('abbbb'.match(/ab+?/));//['ab']
```

⑨修饰符

模式的附加规则，修饰符可单个使用，也可多个一起使用

g修饰符：全局匹配，正则对象将匹配全部符合条件的结果，主要用于搜索和替换。每次都从上一次匹配成功处开始向后匹配。不含g修饰符时，每次都从字符串头部开始匹配。

```js
var regex = /b/g;
var str = 'abba';
console.log(regex.test(str));//true
console.log(regex.test(str));//true
console.log(regex.test(str));//false
```

i修饰符：默认情况下正则对象区分大小写，i修饰符表示忽略大小写

```js
console.log(/abc/i.test('ABC'));//true
```

m修饰符：多行模式，会修改^和$行为。加上m修饰符,^和$还会匹配行首和行尾，即^和$会识别换行符\n

```js
console.log(/ab$/m.test('ab\n'));//true
console.log(/^b/m.test('a\nb'));//true
```

⑩组匹配

正则表达式的括号表示分组匹配。

```js
console.log(/(ab)+/.test('abab'));//true
```

匹配ab整个词语

另一个分组捕获例子：

```js
console.log('abcabc'.match(/(.)b(.)/));//['abc','a','c']
```

🔺使用组匹配时，不建议同时使用g修饰符

另外例子：

```js
console.log(/y(..)(.)\2\1/.test('yabccab'));//true
console.log(/y((..)\2)\1/.test('yabababab'));//true
```

\1指向外层括号，\2指向内层括号

⭐捕获带有属性的标签实例：

```js
var html = '<b class="hello">Hello</b><i>world</i>';
var tag = /<(\w+)([^>]*)>(.*?)<\/\1>/g;

var match = tag.exec(html);
console.log(match[1],match[2],match[3]);//"b" "class="hello"" "Hello"
var match = tag.exec(html);
console.log(match[1],match[2],match[3]);//"i" :world"
```

非捕获组：(?:x)，不返回该组匹配的内容，即匹配的结果中不计入这个括号

```js
console.log('abc'.match(/(?:.)b(.)/));//['abc','c']
```

⭐分解网址实例（采用非捕获组，不返回网络协议）：

```js
var url = /(?:http|ftp):\/\/([^/\r\n]+)(\/[^\r\n]*)?/;
console.log(url.exec('http://google.com/'));//["http://google.com/","google.com","/"]
```

先行断言：x(?=y)，x只有在y前面才匹配，y不会被计入返回结果。括号里内容不会返回。

如匹配后面跟着百分号的数字： `/\d+(?=%)/`

```js
console.log('abc'.match(/b(?=c)/));//['b']
```

先行否定断言：x(?!y)，x只有不在y前面才匹配，y不会被计入返回结果。括号里内容不会返回。

```js
console.log(/\d+(?!\.)/.exec('3.141'));//["141"]
console.log('abd'.match(/b(?!c)/));//['b']
console.log('abc'.match(/b(?!c)/));//null
```

