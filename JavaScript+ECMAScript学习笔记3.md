# JavaScript/ECMAScript学习笔记

### 17. 事件

**17.1 EventTarget接口**

事件本质是程序各个组成部分之间的一种通信方法，也是异步编程的一种实现。

DOM的事件操作（监听和触发）都定义在EventTarget接口。所有节点对象都部署了这个接口，其他一些需要事件通信的浏览器内置对象（如XMLHttpRequest、AudioNode、AudioContext等）也部署了这个接口。

有以下三个实例方法：

①EventTarget.addEventListener()

在当前节点或对象上，定义一个特定事件的监听函数。一旦事件发生，就执行监听函数。没有返回值。

三个参数：type为事件名称，大小写敏感。listener为监听函数，事件发生时调用该函数。useCapture为布尔值，表示监听函数是否在捕获阶段触发，默认false（监听函数只在冒泡阶段被触发），该参数可选。

```js
target.addEventListener(type, listener[, useCapture]);
```

🔺第二个参数除了监听函数，还可以是一个具有handleEvent方法的对象。第三个参数除了布尔值useCapture，还可以是一个属性配置对象：capture：布尔值，表示该事件是否在捕获阶段监听函数。once：布尔值，表示监听函数是否只触发一次，然后自动移除。passive：布尔值，表示监听函数不会调用事件的preventDefault方法。如果监听函数调用了，浏览器忽略这个要求，并在监控台输出一行警告。

🔺①addEventListener方法可针对当前对象同一个事件添加多个不同监听函数。这些函数按照添加顺序触发，先添加先触发。如为同一事件多次添加同一函数，只会执行一次，多余的添加将自动被去除。②如果希望向监听函数传递参数，可用匿名函数包装一下监听函数。③监听函数内部this，指向当前事件所在的那个对象。

```js
var p = document.querySelector('p');
function test(x) {
    console.log(x);
}
p.addEventListener('click',function () {test('testing');},false);
```

```js
var p = document.querySelector('p');
p.addEventListener('click', {
    handleEvent(evt) {
        console.log('click');
    }
},{once:true});
```

②EventTarget.removeEventListener()

移除addEventListener方法添加的事件监听函数。没有返回值。

该方法参数与addEventListener方法完全一致。

🔺该方法移除的监听函数，必须是addEventListener方法添加的那个监听函数，而且必须在同一个元素节点，否则无效。

③EventTarget.dispatchEvent()

在当前节点上触发指定事件，从而触发监听函数执行。返回布尔值，只要有一个监听函数调用了Event.preventDefault()，则返回值为false，否则为true。

参数是一个Event对象实例。如果参数为空或不是一个有效的事件对象，将报错。

```js
var p = document.querySelector('p');
p.addEventListener('click', {
    handleEvent(evt) {
        console.log('click');
    }
});
p.dispatchEvent(new Event('click'));
p.dispatchEvent(new Event('click'));//触发了2次点击事件
```

**17.2 事件模型**

①监听函数

浏览器事件模型，通过监听函数对事件做出反应。事件发生后，浏览器监听到这个事件，会执行对应监听函数，这时事件驱动编程模式主要编程方式。有三种方法为事件绑定监听函数：

👉HTML的on-属性

这些属性值是将会执行的代码而不是一个函数。

一旦指定事件发生，on-属性的值是原样传入JavaScript引擎执行。因此如果执行函数，不要忘记加上圆括号。

使用这个方法指定的监听代码，只会在冒泡阶段触发。事件从子元素开始冒泡到父元素：

```html
<div onclick="console.log(2)">
    <button onclick="console.log(1)">click</button>
</div>
<!--先输出1，再输出2-->
```

直接设置on-属性，与通过元素节点的setAttribute方法设置on-属性效果一样。

👉元素节点的事件属性

这个方法指定的监听函数只会在冒泡阶段触发。

🔺与HTML的on-属性不同的是，它的值是函数名，而后者必须给出完整监听代码。

```js
window.onload = doSomething;

div.onclick = function () {
    console.log(2);
}
```

👉EventTarget.addEventListener()

```js
window.addEventListener('load', doSomething, false);
```

⭐第一种方法不利于代码分工，不推荐使用。第二种方法，如果定义两次事件属性，后一次定义会覆盖前一次，不推荐使用。第三种有如下优点：同一事件可添加多个监听函数。能指定哪个阶段（捕获阶段还是冒泡阶段）触发监听函数。除了DOM节点，其他对象（如window，XMLHttpRequest等）也有这个接口，等于是整个JavaScript统一的监听函数接口。

- this的指向

三种写法里，this指向触发事件的那个元素节点。

- 事件的传播

第一阶段：从window对象传导到目标节点（上层传到底层），称为“捕获阶段”

第二阶段：在目标节点上触发，称为“目标阶段”

第三阶段：从目标节点传导回window对象（底层传到上层），称为“冒泡阶段”

使得同一个事件会在多个节点上触发。

```js
// <div>
//    <p>点击</p>
// </div>

var phases = {1:'capture',2:'target',3:'bubble'};
var div = document.querySelector('div');
var p = document.querySelector('p');
div.addEventListener('click',callback,true);
p.addEventListener('click',callback,true);
div.addEventListener('click',callback,false);
p.addEventListener('click',callback,false);

function callback(event) {
    var tag = event.currentTarget.tagName;
    var phase = phases[event.eventPhase];
    console.log(tag,phase);
}
// DIV capture
// P target
// P target
// DIV bubble
```

捕获阶段，事件从`<div>`向`<p>`传播，触发`<div>`的click事件。

目标阶段，事件从`<div>`到达`<p>`时，触发`<p>`的click事件。

冒泡阶段：事件从`<p>`传回`<div>`时，再次触发`<div>`的click事件。

🔺浏览器总是假定click事件目标节点是点击位置嵌套最深的那个节点。

事件传播顺序：window、document、html、body、div、p。（冒泡阶段逆序）

- 事件的代理

由于事件在冒泡阶段向上传播到父节点，因此可把子节点监听函数定义在父节点上，由父节点监听函数统一处理多个子元素的事件。叫做事件代理（delegation）。

```js
var ul = document.querySelector('ul');

ul.addEventListener('click', function (event) {
  if (event.target.tagName.toLowerCase() === 'li') {
    // some code
  }
});
```

如果希望事件到某个节点为止不再传播，可使用事件对象的stopPropagation方法：

```js
// 事件传播到 p 元素后，就不再向下传播了
p.addEventListener('click', function (event) {
  event.stopPropagation();
}, true);

// 事件冒泡到 p 元素后，就不再向上冒泡了
p.addEventListener('click', function (event) {
  event.stopPropagation();
}, false);
```

但该方法只阻止事件传播，不阻止该事件触发`<p>`节点的其他click事件监听函数。即不是彻底取消click事件。

```js
p.addEventListener('click', function (event) {
  event.stopPropagation();
  console.log(1);
});

p.addEventListener('click', function(event) {
  // 会触发
  console.log(2);
});
// 输出结果先是1，后是2
```

如要彻底取消该事件，不再触发后面所有click监听函数，可使用stopImmediatePropagation方法：

```js
p.addEventListener('click', function (event) {
  event.stopImmediatePropagation();
  console.log(1);
});

p.addEventListener('click', function(event) {
  // 不会被触发
  console.log(2);
});
// 只会输出1，不会输出2
```

**17.3 Event对象**

事件发生后会产生一个事件对象，作为参数传给监听函数。浏览器原生提供一个Event对象，所有事件都是这个对象实例，或者说继承了Event.prototype对象。

Event对象本身是一个构造函数，用来生成实例：

```js
event = new Event(type, options);
```

第一个参数是字符串，表示事件名称。第二个参数是对象，表示事件对象配置。主要有以下两个属性：①bubbles：布尔值，默认false，表示事件对象是否冒泡。②cancelable：布尔值，默认false，表示事件是否可被取消，即能否用Event.preventDefault()取消这个事件。一旦事件被取消，就好像从来没发生过，不会触发浏览器对该事件默认行为。

🔺如果不显式指定bubbles属性为true，生成的事件只能在“捕获阶段”触发监听函数。

①实例属性

- Event.bubbles，Event.eventPhase

Event.bubbles表示事件是否会冒泡，只读。

Event.eventPhase：返回整数常量，表示事件目前所处的阶段，只读。

四种可能返回值：0表示事件目前没发生。1表示事件目前处于捕获阶段。2表示事件到达目标节点，即Event.target属性指向的那个节点。3表示事件处于冒泡阶段。

- Event.cancelable，Event.cancalBubble，Event.defaultPrevented

Event.cancalable返回布尔值，表示事件是否可被取消，只读。大多数浏览器原生事件可以取消，但除非显式声明，Event构造函数生成的事件默认不可取消。

Event.cancalBubble是布尔值，如设为true，相当于执行Event.stopPropagation()，可阻止事件传播。

Event.defaultPrevented返回布尔值，表示该事件是否调用过Event.preventDefault方法，只读。

- Event.currentTarget，Event.target

Event.currentTarget返回事件当前所在节点，即事件当前正在通过的节点，即当前正在执行的监听函数所在的那个节点。随着事件传播，这个属性值会变化。

Event.target返回原始触发事件的那个节点，即事件最初发生的节点。不会随着事件传播而改变。

```js
// <p>123<em>456</em></p>
function test(e) {
    // 不管点击123还是456，总是返回true
    console.log(this === e.currentTarget);
    // 点击123，返回true；点击456，返回false
    console.log(this === e.target);
}
document.querySelector('p').addEventListener('click',test,false);
```

🔺监听函数只有事件经过时才触发

- Event.type

返回字符串，表示事件类型。事件的类型是在生成事件的时候指定的。只读。

```js
var evt = new Event('aaa');
console.log(evt.type);//"aaa"
```

- Event.timeStamp

返回一个毫秒时间戳，表示事件发生的时间。相对于网页加载成功开始计算。

- Event.isTrusted

返回布尔值，表示事件是否由真实的用户行为产生。比如用户点击链接产生click事件，该事件是用户产生的；Event构造函数生成的事件则是脚本产生的。

- Event.detail

只有浏览器UI事件才具有。返回数值，表示事件某种信息。如click和dbclick事件，该属性是鼠标按下的次数；对于鼠标滚轮事件，返回滚轮正向滚动距离，负值就是负向滚动距离，返回值总是3的倍数。

②实例方法

- Event.preventDefault()

取消浏览器对当前事件默认行为。该方法生效前提是事件对象的cancalable属性为true。

应用：为文本输入框设置校验条件。如果用户输入不合条件就无法将字符输入文本框。

```js
// <input type="checkbox" id="checkbox">
var input = document.getElementById('checkbox');
input.addEventListener('click',function(e) {e.preventDefault();},false);
// 无法选中单选框
```

- Event.stopPropagation()

阻止事件在DOM中继续传播，防止再触发定义在别的节点上的监听函数，但不包括在当前节点上其他的事件监听函数。

- Event.stopImmediatePropagation()

阻止同一个事件的其他监听函数被调用，不管监听函数定义在当前节点还是其他节点。

如果同一个节点对于同一个事件指定了多个监听函数，这些函数会根据添加顺序依次调用。只要其中有一个监听函数调用了Event.stopImmediatePropagation()，其他监听函数不会再执行。

```js
// <p>123</p>
function f1(e) {
    e.stopImmediatePropagation();
}
function f2(e) {
    console.log('ok');
}
var p = document.querySelector('p');
p.addEventListener('click',f1,false);
p.addEventListener('click',f2,false);//无输出
```

- Event.composedPath()

返回数组，成员是事件的最底层节点和依次冒泡经过的所有上层节点。

```js
// <div>
//    <p>click</p>
// </div>

var div = document.querySelector('div');
var p = document.querySelector('p');
p.addEventListener('click',function (e) {
    console.log(e.composedPath());//[p,div,body,html,document,Window]
},false);
```

**17.4 鼠标事件**

- 鼠标事件种类

继承了MouseEvent接口。具体事件：

click：按下鼠标时候触发

dbclick：在同一个元素上双击鼠标触发

mousedown：按下鼠标键时触发

mouseup：释放按下的鼠标键时触发

mousemove：当鼠标在一个节点内部移动时触发。当鼠标持续移动，该事件连续触发。为避免性能问题，可限定一段时间内只能运行一次。

mouseenter：鼠标进入一个节点时触发，进入子节点不会触发。

mouseover：鼠标进入一个节点时触发，进入子节点再一次触发。

mouseout：鼠标离开一个节点时触发，离开父节点也会触发。

mouseleave：鼠标离开一个节点时触发，离开父节点不会触发。

contextmenu：按下鼠标右键（上下文菜单出现前）触发，或按下“上下文菜单键”时触发。

wheel：滚动鼠标滚轮时触发，继承WheelEvent接口。

🔺click事件触发顺序：mousedown首先触发，mouseup接着触发，click最后触发。dbclick事件则会在mousedown、mouseup、click之后触发。

- MouseEvent接口

滚轮事件和拖拉事件也是MouseEvent实例。MouseEvent接口继承了Event接口。

浏览器原生提供一个MouseEvent构造函数：

```js
var event = new MouseEvent(type, options);
```

第一个参数是字符串，表示事件名称。第二个参数是事件配置对象，可选，除了Event接口的实例配置属性，还可配置如下属性：

screenX，screenY：数值，鼠标相对于屏幕水平（垂直）位置（单位像素）。默认值0，设置后不会移动鼠标。

clientX，clientY：数值，鼠标相对于程序窗口的水平（垂直）位置（单位像素），默认值0，设置后不会移动鼠标

ctrlKey，shiftKey，altKey，metaKey：布尔值，是否同时按下了Ctrl键/Shift键/Alt键/Meta键，默认false。

button：数值，表示按下哪个鼠标按键。默认值0，表示按下主键（左键）或当前事件没定义这个属性；1表示按下辅助键（中间键），2表示按下次要键（右键）

buttons：数值，表示按下了鼠标哪些键。三个比特位的二进制值，默认0（没按下任何键）。1（001）表示按下主键（左键），2（010）表示按下次要键（右键），4（100）表示按下辅助键（中间键）。即如果返回3（011）表示同时按下了左键和右键。

relatedTarget：节点对象，事件相关节点，默认null。mouseenter和mouseover事件时，表示鼠标刚刚离开的那个元素节点；mouseout和mouseleave事件时，表示鼠标正在进入的那个元素节点。

- MouseEvent接口实例属性

MouseEvent.ctrlKey，MouseEvent.altKey，MouseEvent.metaKey，MouseEvent.shiftKey

具体用法见上。

```js
// <p onclick="test(event)">123</p>
function test(e) {
    console.log(e.altKey,e.metaKey,e.ctrlKey,e.shiftKey);
}
// 点击网页输出是否同时按下对应键
```

MouseEvent.button，MouseEvent.buttons

具体用法见上。

```js
// <p onmouseup="test(event)">123</p>
function test(e) {
    console.log(e.button);
}
```

MouseEvent.clientX，MouseEvent.clientY

具体用法见上。只读。

还有别名MouseEvent.x和MouseEvent.y。

MouseEvent.movementX，MouseEvent.movementY

返回当前位置与上一个mousemove事件之间水平（垂直）距离（单位像素）。只读。

```js
currentEvent.movementX = currentEvent.screenX - previousEvent.screenX
currentEvent.movementY = currentEvent.screenY - previousEvent.screenY
```

MouseEvent.screenX，MouseEvent.screenY

具体用法见上。只读。

MouseEvent.offsetX，MouseEvent.offsetY

返回鼠标位置与目标节点左侧（上方）的padding边缘的水平距离（单位像素）。只读。

MouseEvent.pageX，MouseEvent.pageY

返回鼠标位置与文档左侧（上侧）边缘距离（单位像素）。返回值包括文档不可见部分。只读。

MouseEvent.relatedTarget

只读。下面是不同事件target属性值和relatedTarget属性值：

| 事件名称   | target属性     | relatedTarget属性 |
| ---------- | -------------- | ----------------- |
| focusin    | 接受焦点的节点 | 丧失焦点的节点    |
| focusout   | 丧失焦点的节点 | 接受焦点的节点    |
| mouseenter | 将要进入的节点 | 将要离开的节点    |
| mouseleave | 将要离开的节点 | 将要进入的节点    |
| mouseout   | 将要离开的节点 | 将要进入的节点    |
| mouseover  | 将要进入的节点 | 将要离开的节点    |
| dragenter  | 将要进入的节点 | 将要离开的节点    |
| dragexit   | 将要离开的节点 | 将要进入的节点    |

- MouseEvent接口实例方法

MouseEvent.getModifierState()

返回布尔值，表示有没有按下特定功能键，参数是一个表示功能键的字符串。

```js
// <p onclick="test(event)">123</p>
function test(e) {
    console.log(e.getModifierState('CapsLock'));//是否按下大写键
}
```

- WheelEvent接口

继承了MouseEvent实例，代表鼠标滚轮事件实例对象。

浏览器原生提供WheelEvent()构造函数：

```js
var wheelEvent = new WheelEvent(type, options);
```

第一个参数是字符串，表示事件类型，目前只有wheel。第二个参数表示事件配置对象，该对象属性除了Event、UIEvent的配置属性外，还可接受如下属性：

deltaX，deltaY，deltaZ：数值，滚轮的水平/垂直/Z轴滚动量，默认0.0

deltaMode：数值，相关滚动事件单位。0表示滚动单位为像素，1表示单位为行，2表示单位为页，默认0。

👉实例属性

除了具有Event和MouseEvent实例属性和实例方法，还具有以下实例属性，但没有自己的实例方法。

WheelEvent.deltaX，WheelEvent.deltaY，WheelEvent.deltaZ，WheelEvent.deltaMode

具体用法见上。都是只读。

**17.5 键盘事件**

- 键盘事件种类

键盘事件由用户击打键盘触发，主要有keydown、keypress、keyup三个事件，都继承了KeyboardEvent接口。

keydown：按下键盘时触发

keypress：按下有值的键时触发，即按下Ctrl、Alt、Shift、Meta这样无值的键这个事件不触发。对于有值的键，按下时先触发keydown事件，再触发这个事件。

keyup：松开键盘时触发该事件

🔺如果用户一直按键不松开，就连续触发键盘事件，触发顺序如下：

keydown、keypress、keydown、keypress......（重复以上过程）、keyup

- KeyboardEvent接口

描述用户与键盘的互动。继承了Event接口。

浏览器原生提供KeyboardEvent构造函数：

```js
new KeyboardEvent(type, options)
```

第一个参数是字符串，表示事件类型。第二个参数是事件配置对象，除了Event接口提供的属性，还可配置如下属性：

key：字符串，当前按下的键，默认空字符串。

code：字符串，当前按下的键的字符串形式，默认空字符串。

location：整数，当前按下的键的位置，默认0.

ctrlKey，shiftKey，altKey，metaKey：布尔值，是否按下Ctrl/Shift/Alt/Meta键，默认false。

repeat：布尔值，是否重复按键，默认false。

- KeyboardEvent实例属性

KeyboardEvent.altKey，KeyboardEvent.ctrlKey，KeyboardEvent.metaKey，KeyboardEvent.shiftKey。

具体用法见上。只读。

```js
// <input id="input">
var input = document.getElementById('input');
input.addEventListener('keydown',function (e) {
    console.log(e.altKey,e.ctrlKey);
},false);
```

KeyboardEvent.code

只读。

数字键0-9：返回digit0 - digit9

字母键A-z：返回KeyA - KeyZ

功能键F1-F12：返回F1 - F12

方向键：返回ArrowDown、ArrowUp、ArrowLeft、ArrowRight

Alt键：返回AltLeft或AltRight

Shift键：返回ShiftLeft或ShiftRight

Ctrl键：返回ControlLeft或ControlRight

KeyboardEvent.key

只读。

如果按下的键代表可打印字符，返回这个字符，如数字、字母。

如果按下的键代表不可打印的特殊字符，返回预定义的键值，如Backspace、Tab、Enter、CapsLock等。

如果同时按下一个控制键和一个符号键，返回符号键键名。如按下Ctrl + a，返回a；按下Shift + a，返回A。

如果无法识别键名，返回字符串Unidentified。

KeyboardEvent.location

0：键盘主区域，或无法判断处于哪个区域

1：键盘左侧，只适用于那些有两个位置的键。（如Ctrl和Shift）

2：键盘右侧，只适用于那些有两个位置的键。（如Ctrl和Shift）

3：数字小键盘

```js
// <input id="input">
var input = document.getElementById('input');
input.addEventListener('keydown',function (e) {
    console.log(e.location,e.key,e.code);//按下左边shift键，输出 1 “Shift" "ShiftLeft"
},false);
```

KeyboardEvent.repeat

该键是否被按着不放。

- KeyboardEvent实例方法

KeyboardEvent.getModifierState()

返回布尔值，表示是否按下或激活指定的功能键。常用参数如下：

Alt、CapsLock、Control、Meta、NumLock、Shift

```js
// <input id="input">
var input = document.getElementById('input');
input.addEventListener('keydown',function (e) {
    console.log(e.getModifierState('Shift'));
},false);
```

**17.6 进度事件**

进度事件描述资源加载进度，主要由AJAX请求、`<img>`、`<audio>`、`<video>`、`<style>`、`<link>`等外部资源的加载触发，继承了ProgressEvent接口。主要包含以下事件：

abort：外部资源中止加载时（如用户取消）触发。如果发生错误导致中止不会触发。

error：由于错误导致外部资源无法加载时触发。

load：外部资源加载成功时触发

loadstart：外部资源开始加载时触发

loadend：外部资源停止加载时触发，发生顺序排在error、abort、load等事件后面

progress：外部资源加载过程中不断触发。

timeout：加载超时时触发

🔺除了资源下载，文件上传也存在这些事件。

有时图片加载会在脚本运行前完成，尤其是当脚本放置在网页底部时。可用complete属性先判断一下是否加载完成。由于DOM元素节点没提供是否加载错误的属性，所以error事件的监听函数最好放在`<img>`元素的HTML代码中（onerror），才能保证发生加载错误时肯定会执行。

🔺error事件不会冒泡。子元素的error事件，不会触发父元素error事件监听函数。

- ProgressEvent接口

主要用来描述外部资源加载进度，如AJAX加载，`<img>`、`<video>`、`<style>`、`<link>`等外部资源加载。进度相关的事件都继承了接口。

浏览器原生提供了ProgressEvent()构造函数：

```js
new ProgressEvent(type, options)
```

第一个参数是字符串，表示事件类型。第二个参数是事件配置对象，除了用Event接口配置属性，还可使用如下属性：

lengthComputable：布尔值，表示加载的总量是否可以计算，默认false。

loaded：整数，表示已经加载过的量，默认0

total：整数，表示需要加载的总量，默认0

👉ProgressEvent具有对应的实例属性：

ProgressEvent.lengthComputable

ProgressEvent.loaded

PrpgressEvent.total

如果ProgressEvent.lengthComputable为false，ProgressEvent.total实际没意义。

```js
var p = new ProgressEvent('load', {lengthComputable:true,loaded:20,total:100});
document.body.addEventListener('load', function (e) {
    console.log(e.loaded/e.total);//0.2
},false);
document.body.dispatchEvent(p);
```

⭐下载过程和上传过程实例：

```js
// 下载过程
var xhr = new XMLHttpRequest();
xhr.addEventListener('progress', updateProgress, false);
xhr.addEventListener('load', transferComplete, false);
xhr.addEventListener('error', transferFailed, false);
xhr.addEventListener('abort', transferCanceled, false);
xhr.open();

//上传过程
var xhr = new XMLHttpRequest();
xhr.upload.addEventListener('progress', updateProgress, false);
xhr.upload.addEventListener('load', transferComplete, false);
xhr.upload.addEventListener('error', transferFailed, false);
xhr.upload.addEventListener('abort', transferCanceled, false);
xhr.open();
```

**17.7 表单事件**

- 表单事件种类

①input事件

当`<input>`、`<select>`、`<textarea>`值发生变化时触发。对于复选框（`<input type=checkbox>`）或单选框（`<input type=radio>`），用户改变选项时也触发。对于打开contenteditable属性的元素，只要值发生变化，也触发。

input事件会连续触发，用户每按下一次按键，就触发一次。input事件对象继承了InputEvent接口。

input事件和change事件很像，不同之处在于input事件在元素值发生变化后立即发生；而change事件在元素失去焦点时发生，而内容此时可能已变化多次。即如有连续变化，input事件触发多次，change事件只在失去焦点时触发一次。

```js
//<select id="select">
//    <option value="1">1</option>
//    <option value="2">2</option>
//</select>
var input = document.getElementById('select');
input.addEventListener('input', function (e) {
    console.log(e.target.value);//改变下拉框选项触发input事件
},false);
```

②select事件

在`<input>`、`<textarea>`里选中文本时触发。

选中的文本可通过event.target元素的selectionDirection、selectionEnd、selectionStart和value属性得到。

```js
// <input id="input" type="text" value="select">
var input = document.getElementById('input');
input.addEventListener('select', function (e) {
    console.log(e.type,e.target.selectionStart,e.target.selectionDirection,e.target.value);
    // 选中"lec"，输出 "select" "2" "forward" "select"
},false);
```

③change事件

change事件不会连续触发，input事件必然伴随change事件。

激活单选框（radio）或者复选框（checkbox）时触发。

用户提交时触发。如从下拉列表（select）完成选择，在日期或文件输入框完成选择。

当文本框或`<textarea>`元素值发生改变，并且丧失焦点时触发。

```js
// <input type="radio" id="radio">
var radio = document.getElementById('radio');
radio.addEventListener('change', function (e) {
    console.log(e.target.nodeName);//"INPUT"
},false);
```

④invalid事件

用户提交表单时，如果表单元素值不满足校验条件，触发invalid事件。

```js
// <form>
//    <input type="text" id="input" required>
//    <button type="submit">submit</button>
//</form>
var input = document.getElementById('input');
input.addEventListener('invalid', function (e) {
    console.log('invalid');//输入框必填项，不填时点击submit按钮输出"invalid"
},false);
```

⑤reset事件、submit事件

发生在表单对象`<form>`上，而不是发生在表单的成员上。

reset事件当表单重置（所有表单成员变回默认值）时触发。

submit事件当表单数据向服务器提交时触发。

- InputEvent接口

描述input事件的实例。继承了Event接口。

浏览器原生提供InputEvent()构造函数：

```js
new InputEvent(type, options)
```

第一个参数是字符串，表示事件名称。第二个参数是事件配置对象，除了Event构造函数的配置属性，还可设置如下字段：

inputType：字符串，表示发生变更的类型

data：字符串，表示插入的字符串。如没有插入的字符串（如删除操作），返回null或空字符串。

dataTransfer：返回一个DataTransfer对象实例，通常只在输入框接受富文本时输入有效

这三个属性都是只读。

InputEvent.data

返回字符串，表示变动内容。

InputEvent.inputType

返回字符串，表示字符串发生变更的类型。Chrome浏览器返回值：

手动插入文本：insertText。粘贴插入文本：insertFromPaste。向后删除：deleteContentBackward。向前删除：deleteContentForward。

InputEvent.dataTransfer

只在文本框接受粘贴内容（insertFromPaste）或拖拽内容（insertFromDrop）时才有效。

```js
// <input type="text" id="input">
var input = document.getElementById('input');
input.addEventListener('input', function (e) {
    console.log(e.data,e.inputType);//data属性：依次按下123字符，先输出1，在下一行输出2，在下一行输出3，然后一次性删除三个字符，在下一行输出null。
},false);
```

**17.8 触摸事件**

- 触摸操作

浏览器触摸API由三部分组成：

Touch：一个触摸点

TouchList：多个触摸点的集合

TouchEvent：触摸引发的事件实例

Touch接口的实例对象用来表示触摸点（一根手指或一根触摸笔），包括位置、大小、形状、压力、目标元素等属性。触摸动作由多个触摸点（多根手指）组成的集合由TouchList接口实例对象表示。只有触摸屏才会引发TouchEvent事件。

很多时候触摸事件和鼠标事件同时触发，即使这时并没有用到鼠标。这是为了让只定义鼠标事件没有定义触摸事件的代码，在触摸屏情况下仍然适用。如要避免这种情况，可用event.preventDefault方法阻止发出鼠标事件。

- Touch接口

浏览器原生提供Touch构造函数：

```js
var touch = new Touch(touchOptions);
```

接受配置对象作参数：

identifier：必需，整数，触摸点唯一ID

target：必需，元素节点，触摸点开始时所在的网页元素

clientX，clientY：可选，数值，触摸点相对于浏览器窗口左上角的水平（垂直）距离，默认0

screenX，screenY：可选，数值，触摸点相对于屏幕左上角的水平（垂直）距离，默认0

pageX，pageY：可选，数值，触摸点相对于网页左上角的水平（垂直）位置，（包括页面滚动距离），默认0

radiusX，radiusY：可选，数值，触摸点周围受到影响的椭圆范围的X轴（Y轴）半径，默认0

rotationAngle：可选，数值，触摸区域的椭圆的旋转角度，单位为度数，0到90度之间，默认0

force：可选，数值，在0到1之间，触摸压力。0代表没有压力，1代表硬件所能识别的最大压力，默认0

👉实例属性

Touch.identifier

该值在整个触摸过程中保持不变，直到触摸事件结束。

Touch.screenX，Touch.screenY，Touch.clientX，Touch.clientY，Touch.pageX，Touch.pageY

pageX和pageY包含了页面滚动带来的位移，其他与页面滚动无关

Touch.radiusX，Touch.radiusY，Touch.rotationAngle

radiusX和radiusY单位为像素，乘以2就得到触摸范围的宽度和高度。这三个属性共同定义了用户与屏幕接触的区域，指尖接触屏幕，触摸范围形成一个椭圆，这三个属性用来描述这个椭圆区域。

Touch.force

触摸压力

Touch.target

即使触摸点已经离开这个节点，属性依然不变

- TouchList接口

实例是一个类似数组的对象，成员是Touch的实例对象，表示所有触摸点。用户用三根手指触摸，产生的TouchList实例会包含三个成员，每根手指触摸点对应一个Touch实例对象。

它的实例主要通过触摸事件的TouchEvent.touches，TouchEvent.changedTouches，TouchEvent.targetTouches这几个属性获取。

它的实例属性和实例方法只有两个：

TouchList.length：成员数量，触摸点的数量

TouchList.item()：返回指定位置成员，参数是该成员位置编号

- TouchEvent接口

继承了Event接口，表示由触摸引发的事件实例，通常来自触摸屏或轨迹板。

浏览器原生提供TouchEvent()构造函数：

```js
new TouchEvent(type, options)
```

第一个参数是字符串，表示事件类型。第二个参数是事件配置对象，除了Event接口配置属性，还有以下属性：

touches：TouchList实例，代表所有当前处于活跃状态触摸点，默认是空数组。

targetTouches：TouchList实例。代表所有处在触摸的目标元素节点内部、且仍然处于活动状态的触摸点，默认空数组。

changedTouches：TouchList实例，代表本次触摸事件的相关触摸点，默认空数组。

ctrlKey，shiftKey，altKey，metaKey：布尔值，表示Ctrl/Shift/Alt/Meta键是否同时按下，默认false。

👉实例属性

除了具有Event实例的所有属性和方法，还有自己的实例属性，都是只读。

TouchEvent.altKey，TouchEvent.ctrlKey，TouchEvent.shiftKey，TouchEvent.metaKey

是否按下某个键

TouchEvent.changedTouches

返回TouchList实例，成员是一组Touch实例对象，表示本次触摸事件的相关触摸点。

对于不同时间，该属性含义不同：

touchstart事件：被激活的触摸点

touchmove事件：发生变化的触摸点

touchend事件：消失的触摸点（不再被触碰的点）

TouchEvent.touches

返回TouchList实例，成员是所有仍处于活动状态（触摸中）的触摸点。一般，一个手指就是一个触摸点。

TouchEvent.targetTouches

具体用法见上。

- 触摸事件种类

可通过TouchEvent.type属性查看到底发生哪一个事件：

touchstart：用户开始触摸时触发，target属性返回发生触摸的元素节点

touchend：用户不再接触触摸屏时（或移出屏幕边缘时）触发，target属性与touchstart事件一致，即开始触摸时所在元素节点。changedTouches属性返回一个TouchList实例，包含所有不再触摸的触摸点。

touchmove：用户移动触摸点时触发，target属性与touchstart事件一致。如果触摸半径、力度、角度发生变化，也会触发。

touchcancel：触摸点取消时触发，如在触摸区域跳出一个模态窗口、触摸点离开了文档区域（进入浏览器菜单栏）、用户触摸点太多，超过了支持的上限（自动取消早先的触摸点）。

```js
var el = document.getElementsByTagName('canvas')[0];
el.addEventListener('touchstart', handleStart, false);
el.addEventListener('touchmove', handleMove, false);

function handleStart(evt) {
  evt.preventDefault();
  var touches = evt.changedTouches;
  for (var i = 0; i < touches.length; i++) {
    console.log(touches[i].pageX, touches[i].pageY);
  }
}

function handleMove(evt) {
  evt.preventDefault();
  var touches = evt.changedTouches;
  for (var i = 0; i < touches.length; i++) {
    var touch = touches[i];
    console.log(touch.pageX, touch.pageY);
  }
}
```

**17.9 拖拉事件**

- 拖拉事件种类

拖拉，指用户在某个对象上按下鼠标键不放，拖动它到另一位置，然后释放鼠标键，将该对象放在那里。

拖拉的对象，包括元素节点、图片、链接、选中的文字等。网页中，除了元素节点默认不可拖拉，其他都可直接拖拉。为了让元素节点可拖拉，可将该节点draggable属性设为true：

```html
<div draggable="true">
    123
</div>
```

上例，松开鼠标键后拖动效果消失，div还在原来位置。

draggable属性可用于任何元素节点，但`<img>`和`<a>`不加这个属性也可以拖拉。

🔺一旦某个元素节点draggable属性设为true，就无法再用鼠标选中该节点内部的文字或子节点。

当元素节点或选中的文本被拖拉时，会持续触发拖拉事件：

drag：拖拉过程中，在被拖拉的节点上持续触发（相隔几百毫秒）

dragstart：用户开始拖拉时，在被拖拉节点上触发，target属性是被拖拉的节点。通常应在该事件监听函数中指定拖拉的数据。

dragend：拖拉结束时（释放鼠标键或按下ESC键），在被拖拉节点上触发，target属性是被拖拉的节点。与dragstart事件在同一节点触发。不管拖拉是否跨窗口，或中途被取消，dragend事件总是会触发。

dragenter：拖拉进入当前节点时，在当前节点触发一次，target属性是当前节点。通常该事件的监听函数中，指定是否允许在当前节点放下（drop）拖拉的数据。如当前节点没有该事件监听函数，或监听函数不执行任何操作，即不允许在当前节点放下数据。在视觉上显示拖拉进入当前节点，也是在这个事件监听函数设置。

dragover：拖拉到当前节点上方时，在当前节点上持续触发（相隔几百毫秒），target属性是当前节点。与dragenter事件区别是：dragenter事件在进入该节点时触发，只要没有离开这个节点，dragover事件持续触发。

dragleave：拖拉操作离开当前节点范围时，在当前节点触发，target属性是当前节点。在视觉上显示拖拉离开操作当前节点，在该事件监听函数中设置。

drop：被拖拉的节点或选中的文本，释放到目标节点时，在目标节点上触发。如当前节点不允许drop，即使在该节点上松开鼠标键，也不会触发。如用户按下ESC键，取消这个操作，也不会触发。该事件监听函数负责取出拖拉数据并进行处理。

```js
//<div id="div1" style="width:200px;height:200px;border: 1px solid black">
//    <p draggable="true" id="p">123</p>
//</div>
//<div id="div2" style="width:200px;height:200px;border:1px solid black" draggable="true"></div>
var div1 = document.getElementById('div1');
var div2 = document.getElementById('div2');
var p = document.getElementById('p');
p.addEventListener('dragstart',function (e) {
    console.log('start');
},false);
p.addEventListener('dragend',function (e) {
    console.log('end');
},false);
div2.addEventListener('dragenter',function (e) {
    // e.preventDefault();
    console.log('enter');
},false);
div2.addEventListener('dragover',function (e) {
    e.preventDefault();
    console.log('over');
},false);
div2.addEventListener('drop',function (e) {
    div1.removeChild(p);
    div2.appendChild(p);
    console.log(e);
},false);
```

🔺①拖拉过程只触发以上这些拖拉事，尽管鼠标在移动，但鼠标事件不触发。②将文件从操作系统拖拉进浏览器，不会触发dragstart和dragend事件。③dragenter和dragover事件监听函数，用来取出拖拉的数据（允许放下被拖拉的元素）。这两个事件默认设置为当前节点不允许接受被拖拉数据。想要在目标节点上放下数据，首先必须阻止这两个事件默认行为：

```html
<div ondragover="return false">
<div ondragover="event.preventDefault()">
```

- DragEvent接口

拖拉事件都继承了DragEvent接口，这个接口又继承了MouseEvent接口和Event接口。

浏览器原生提供一个DragEvent()构造函数：

```js
new DragEvent(type, options)
```

第一个参数是字符串，表示事件类型。第二个参数是事件配置对象，除了接受MouseEvent接口和Event接口配置属性，还可设置dataTransfer属性，要么是null，要么是一个DataTransfer接口实例。DataTransfer实例对象用来读写拖拉事件中传输的数据。

- DataTransfer接口

浏览器原生提供一个DataTransfer()构造函数：

```js
var dataTrans = new DataTransfer();
```

不接受参数。

拖拉的数据分成两方面：数据的种类（格式）和数据的值。数据的种类是一个MIME字符串（text/plain，image/jpeg等），数据的值是一个字符串。一般，如果拖拉一段文本，数据就是那段文本；如果拖拉一个链接，数据就是链接URL。

拖拉事件开始时，开发者可提供数据类型和数据值。拖拉过程中，开发者通过dragenter和dragover事件监听函数，检查数据类型，以确定是否允许放下（drop）被拖拉的对象。如只允许放下链接的区域，检查拖拉的数据类型是否为text/uri-list

发生drop事件时，监听函数取出拖拉的数据，进行处理。

- DataTransfer实例属性

DataTransfer.dropEffect

设置放下（drop）被拖拉节点时的效果，会影响到拖拉经过相关区域时鼠标形状，取决于：

copy：复制被拖拉的节点

move：移动被拖拉的节点

link：创建指向被拖拉的节点的链接

none：无法放下被拖拉的节点

dropEffect属性一般在dragenter和dragover事件的监听函数中设置，对于dragstart、drag、dragleave这三个事件，该属性不起作用。该属性只对接受被拖拉的节点的区域有效，对被拖拉的节点本身无效。进入目标区域后，拖拉行为会初始化成设定效果。

DataTransfer.effectAllowed

设置本次拖拉中允许效果：

copy：复制被拖拉的节点

move：移动被拖拉的节点

link：创建指向被拖拉节点的链接

copyLink：允许copy或link

copyMove：允许copy或move

linkMove：允许link或move

all：允许所有效果

none：无法放下被拖拉的节点

uninitialized：默认值，等同all

如某种效果不允许，用户无法在目标节点中达成这种效果。

dragstart事件监听函数，可用来设置该属性，其他事件监听函数里设置这个无效。

DataTransfer.files

一个FileList对象，包含一组本地文件，可用来在拖拉操作中传送。如本次拖拉不涉及文件，则属性为空的FileList对象。

DataTransfer.types

只读的数组，每个成员是一个字符串，里面是拖拉的数据格式（通常MIME值）。

DataTransfer.items

返回一个类似数组的只读对象（DataTransferItemList实例），每个成员就是本次拖拉的一个对象（DataTransferItem实例）。如果本次拖拉不包含对象，返回空对象。

DataTransferItemList实例有以下属性和方法：

length：返回成员数量

add(data,type)：增加一个指定内容和类型的字符串作为成员

add(file)：增加一个文件作为成员

remove(index)：移除指定位置成员

clear()：移除所有成员

DataTransferItem实例有以下属性和方法：

kind：返回成员种类（string还是file）

type：返回成员类型（通常MIME值）

getAsFile()：如果被拖拉的是文件，返回该文件，否则返回null

getAsString(callback)：如果被拖拉的是字符串，将该字符传入指定的回调函数处理。该方法异步，所以需要传入回调函数。

- DataTransfer的实例方法

DataTransfer.setData()

设置拖拉事件所带有的数据。没有返回值。

接受两个参数，都是字符串。第一个参数表示数据类，第二个参数是具体数据。如果指定类型的数据在dataTransfer属性不存在，就加入这些数据，否则原有的数据将被新数据替换。

🔺如果是拖拉文本框或者拖拉选中的文本，会默认将对应的文本数据，添加到dataTransfer属性，不用手动指定。

DataTransfer.getData()

接受一个字符串（表示数据类型）为参数，返回事件所带的指定类型的数据（通常是用setData方法添加的数据）。如果指定类型数据不存在，返回空字符串。通常只有drop事件触发后，才能取出数据。

类型值指定为URL，可取出第一个有效链接：

```js
var link = event.dataTransfer.getData('URL');
```

DataTransfer.clearData()

接受一个字符串（表示数据类型）为参数，删除事件所带的指定类型的数据。如没有指定类型，删除所有数据。如指定类型不存在，调用该方法不会产生任何效果。

🔺①该方法不会移除被拖拉的文件，因此调用该方法后，DataTransfer.types属性可能依然返回Files类型。②该方法只能在dragstart事件监听函数中使用，因为这是拖拉操作的数据唯一可写的时机。

DataTransfer.setDragImage()

拖动过程中（dragstart事件触发后），浏览器会显示一张图片跟随鼠标一起移动，表示被拖动的节点。这张图片是自动创造的，通常显示为被拖动节点的外观，无需手动设置。

该方法可自定义这张图片，接受三个参数。第一个参数是`<img>`节点或`<canvas>`节点，如果省略或为null，则使用被拖动的节点的外观；第二个参数和第三个参数为鼠标相对于该图片左上角的横坐标和纵坐标。

**17.10 其他常见事件**

- 资源事件

①beforeunload事件

在窗口、文档、各种资源将要卸载前触发。可用来防止用户不小心卸载资源。

如果该事件对象的returnValue属性是一个非空字符串，浏览器会弹出一个对话框，询问用户是否要卸载该资源。但用户指定的字符串可能无法显示，浏览器会展示预定义的字符串。如果用户点击“取消”按钮，资源不会卸载。

浏览器对这个事件的行为很不一致。不能依赖它来阻止用户关闭浏览器窗口，最好不要使用这个事件。

一旦使用了该事件，浏览器不会缓存当前网页，使用“回退”按钮将重新向服务器请求网页。因为监听这个事件的目的，一般是为了网页状态，这时缓存页面的初始状态就没意义了。

②unload事件

在窗口关闭或者document对象将要卸载时触发。触发顺序排在beforeunload、pagehide事件后面。

该事件发生时，文档处于一个特殊状态。所有资源依然存在，但对用户来说不可见，UI互动全部无效。这个事件无法取消，即使在监听函数里抛出错误，也不能停止文档的卸载。

手机上，浏览器或系统可能直接丢弃网页，这时该事件不会发生。跟beforeunload事件一样，一旦使用unload事件，浏览器不会缓存当前网页，理由同上。任何情况下都不应依赖这个事件，指定网页卸载时要执行的代码，可考虑完全不用该事件。

该事件可用pagehide代替。

③load事件、error事件

load事件在页面或某个资源加载成功时触发。页面或资源从浏览器缓存加载时不会触发。

error事件在页面或资源加载失败时触发。

abort事件在用户取消加载时触发。

这三个事件实际上属于进度事件，不仅发生在document对象，还发生在各种外部资源上。浏览网页是一个加载各种资源过程，图像、样式表、脚本、视频、音频、Ajax请求等，这些资源和document对象、window对象、XMLHttpRequestUpload对象，都会触发load事件和error事件。

页面load事件也可用pageshow事件代替。

- session历史事件

①pageshow事件、pagehide事件

默认，浏览器会在当前会话缓存页面，当用户点击“前进/后退”按钮时，浏览器就会从缓存中加载页面。

pageshow事件在页面加载时触发，包括第一次加载和从缓存加载两种情况。如要指定页面每次加载（不管是否从浏览器缓存）时都运行的代码，可放在这个事件监听函数。

第一次加载，触发顺序排在load事件后。从缓存加载时，load事件不触发，因为网页在缓存中的样子通常是load事件监听函数运行后样子，所以不必重复执行。同理，如从缓存中加载页面，网页内初始化的JavaScript脚本（如DOMContentLoaded事件监听函数）也不会执行。

pageshow事件有一个persisted属性，返回布尔值。页面第一次加载时是false；页面从缓存加载时是true。

pagehide事件与pageshow事件类似，当用户通过“前进/后退”按钮，离开当前页面时触发。与unload事件区别在于，如在window对象上定义了unload事件监听函数后，页面不会保存在缓存中，而使用pagehide事件，页面会保存在缓存中。

pagehide事件也有一个persisted属性，将其设为true表示页面要保存在缓存中；设为false表示网页不保存在缓存中，这时如果设置了unload事件监听函数，该函数将在pahehide事件后立即运行。

如页面包含 `<frame>`或`<iframe>`元素，则`<frame>`页面的pageshow事件和pagehide事件，都会在主页面之前触发。

🔺这两个事件只在浏览器的history对象发生变化时触发，与网页是否可见没关系。

②popstate事件

在浏览器history对象的当前记录发生显式切换时触发。调用history.pushState()或history.replaceState()不会触发该事件。该事件只在用户在history记录之间显式切换时触发，如鼠标点击“后退/前进”按钮，或在脚本中调用history.back()、history.forward()、history.go()时触发。

该事件对象有一个state属性，保存history.pushState()和history.replaceState()为当前记录添加的state对象。

浏览器对于页面首次加载，是否触发该事件，处理不一致。

③hashchange事件

在URL的hash部分（#号后面的部分，包括#号）发生变化时触发。一般在window对象上监听。

有两个特有属性，oldURL属性newURL属性，分别表示变化前后的完整URL。

- 网页状态事件

①DOMContentLoaded事件

网页下载并解析完成后，浏览器就会在document对象上触发该事件。仅仅完成了网页的解析（整张页面DOM生成了），所有外部资源（样式表、脚本、iframe等）可能还没有下载结束。即这个事件比load事件发生时间早得多。

🔺网页的JavaScript脚本是同步执行的，脚本一旦发生堵塞，会推迟触发该事件。

②readystatechange事件

当Document对象和XMLHttpRequest对象的readyState属性发生变化时触发。document.readyState有三个可能值：loading（网页正在加载）、interactive（网页已经解析完成，但外部资源仍处于加载状态）和complete（网页和所有外部资源已结束加载，load事件即将触发）。

可看作是DOMContentLoaded事件另一种实现方法。

- 窗口事件

①scroll事件

在文档或文档元素滚动时触发，主要出现在用户拖动滚动条。

连续大量触发，它的监听函数中不应有非常耗费计算的操作。推荐做法是用requestAnimationFrame或setTimeout控制该事件触发频率，可结合customEvent抛出一个新事件。

🔺throttle函数

```
function throttle(fn, wait) {
  var time = Date.now();
  return function() {
    if ((time + wait - Date.now()) < 0) {
      fn();
      time = Date.now();
    }
  }
}

window.addEventListener('scroll', throttle(callback, 1000));
```

上例可将scroll事件触发频率，限制在一秒一次。

lodash函数库提供了现成throttle函数，可直接使用：

```js
window.addEventListener('scroll', _.throttle(callback, 1000));
```

🔺debounce是防抖，要连续操作结束后再执行。throttle是节流，确保一段时间内只执行一次。以网页滚动为例，debounce是要等到用户停止滚动后才执行，throttle则是如果用户一直在滚动网页，那么在滚动过程中还是会执行。

②resize事件

改变浏览器窗口大小时触发，主要发生在window对象上。

连续大量触发，最好通过throttle函数控制事件触发频率。

③fullscreenchange事件、fullscreenerror事件

fullscreenchange事件在进入或退出全屏状态时触发，该事件发生在document对象上。

fullscreenerror事件在浏览器无法切换到全屏状态时触发。

- 剪贴板事件

cut：将选中的内容从文档中移除，加入剪贴板时触发

copy：进行复制动作时触发

paste：剪贴板内容粘贴到文档后触发

这三个事件的事件对象都是ClipboardEvent接口实例。ClipboardEvent有一个实例属性clipboardData，是一个DataTransfer对象，存放剪贴的数据。

🔺禁止输入框粘贴事件：

```js
inputElement.addEventListener('paste', e => e.preventDefault());
```

- 焦点事件

发生在元素节点和document对象上，与获得或失去焦点相关。主要包括以下四个事件：

focus：元素节点获得焦点后触发，该事件不会冒泡

blur：元素节点失去焦点后触发，该事件不会冒泡

focusin：元素节点将要获得焦点时触发，发生在focus事件之前，该事件会冒泡

focusout：元素节点将要失去焦点时触发，发生在blur事件之前，该事件会冒泡

这四个事件的事件对象都继承了FocusEvent接口。FocusEvent实例有以下属性：

FocusEvent.target：事件的目标节点

FocusEvent.relatedTarget：对于focusin事件，返回失去焦点的节点；对于focusout事件，返回将要接受焦点的节点；对于focus和blur事件，返回null。

🔺由于focus和blur事件不会冒泡，只能在捕获阶段触发，所以addEventListener方法第三个参数设为true。

```js
//<input id="input" type="text">    
var input = document.getElementById('input');
input.addEventListener('focus',function (e) {
    e.target.style.border = '3px solid blue';
},true);
input.addEventListener('blur',function (e) {
    e.target.style = '';
},true);
```

- CustomEvent接口

生成自定义的事件实例。那些浏览器预定义的事件虽可以手动生成，但往往不能在事件上绑定数据。如需要在触发事件同时，传入指定数据，就可使用CustomEvent接口生产的自定义事件对象。

浏览器原生提供CustomEvent()构造函数：

```js
new CustomEvent(type, options)
```

第一个参数是字符串，表示事件名字。第二个参数是事件配置对象，除了接受Event事件的配置属性，只有一个自己的属性：

detail：表示事件的附带数据，默认null。

```js
var event = new CustomEvent('my',{'detail':1});
document.body.addEventListener('my',function (e) {
    console.log(e.detail);//1
},false);
document.body.dispatchEvent(event);
```

**17.11 GlobalEventHandlers接口**

```js
div.onclick = clickHandler;
```

这种指定事件的回调函数方式，这个接口是由GlobalEventHandlers接口提供。优点是使用比较方便，缺点是只能为每个事件指定一个回调函数，并且无法指定事件触发的阶段。

HTMLElement、Document和Window都继承了这个接口，即各种HTML元素、document对象、window对象上都可用这个接口。下面是一些属性：

- GlobalEventHandlers.onabort

某个对象的abort事件（停止加载）发生时，就调用onabort属性指定的回调函数。实际上这个属性一般只用在`<img>`元素上。

- GlobalEventHandlers.onerror

error事件发生时，会调用onerror属性指定的回调函数。

error事件分两种：①JavaScript运行时错误，会传到window对象，导致window.onerror()。该处理函数接受五个参数：message为错误信息字符串、source为报错脚本的URL、lineno为报错的行号（整数）、colno为报错的列号（整数）、error为错误对象。②资源加载错误，如 `<img>`或`<script>`加载的资源出现加载错误，Error对象会传到对应元素，导致该元素onerror属性开始执行。一般资源加载错误不会触发window.onerror

- GlobalEventHandlers.onload，GlobalEventHandlers.onloadstart

元素完成加载时，触发load事件，执行onload()，典型应用场景是window对象和`<img>`元素。对于window对象来说，只有页面所有资源加载完成（包括图片、脚本、样式表、字体等所有外部资源），才会触发load事件。

对于`<img>`和`<video>`等元素，加载开始时还会触发loadstart事件，导致执行onloadstart。

- GlobalEventHandlers.onfocus，GlobalEventHandlers.onblur

当前元素获得焦点时，触发onfocus；失去焦点时，触发onblur。

🔺如果不是可以接受用户输入的元素，要触发onfocus，该元素必须有tabindex属性。

- GlobalEventHandlers.onscroll

页面或元素滚动时，触发scroll事件，执行onscroll()

- GlobalEventHandlers.oncontextmenu，GlobalEventHandlers.onshow

用户在页面上按下鼠标右键，触发contextmenu事件，执行oncontextmenu()。如该属性执行后返回false，等于禁止了右键菜单。document.oncontextmenu和window.oncontextmenu效果一样。

元素右键菜单显示时，触发该元素onshow监听函数。

```js
document.oncontextmenu = function (e) {
    return false;
}
```

- 其他事件属性

①鼠标事件属性：onclick、ondbclick、onmousedown、onmouseenter、onmouseleave、onmouseout、onmouseover、onmouseup、onwheel。

②键盘事件属性：onkeydown、onkeypress、onkeyup。

③焦点事件属性：onblur、onfocus

④表单事件属性：oninput、onchange、onsubmit、onreset、oninvalid、onselect

⑤触摸事件属性：ontouchcancel、ontouchend、ontouchmove、ontouchstart

⑥（拖拉事件）被拖动元素事件属性：ondragstart，ondrag、ondragend

⑦（拖拉事件）接收被拖动元素的容器元素事件属性：ondragenter、ondragleave、ondragover、ondrop

⑧`<dialog>`对话框元素事件属性：oncancel、onclose

### 18. 浏览器模型

**18.1 浏览器环境概述**

JavaScript是浏览器的内置脚本语言。即浏览器内置了JavaScript引擎，并提供各种接口，让JavaScript脚本可控制浏览器各种功能。一旦网页内嵌了JavaScript脚本，浏览器加载网页就会去执行脚本，从而达到操作浏览器目的，实现网页各种动态效果。

- 代码嵌入网页方法

①script元素直接嵌入代码

`<script>`标签有一个type属性，用来指定脚本类型。有两种取值：text/javascript：默认值，对老式浏览器设这个值比较好。application/javascript：对于较新浏览器，建议这个值。

如果type属性值，浏览器不认识，就不会执行其中代码。但`<script>`节点依然存在于DOM中。

②script元素加载外部脚本

如果脚本文件使用了非英语字符，还应该注明字符编码：

```js
<script charset="utf-8" src="https://www.example.com/script.js"></script>
```

所加载脚本必须是纯JavaScript代码，不能有HTML代码和`<script>`标签。

加载外部脚本和直接添加代码块，两种方法不能混用。

🔺为防止攻击者篡改外部脚本，script标签允许设置一个integrity属性，写入该外部脚本Hash签名，验证脚本一致性。

③事件属性

onclick、onmouseover等

④URL协议

URL支持javascript: 协议，即在URL位置写入代码，使用这个URL时就会执行JavaScript代码。

如果代码返回一个字符串，浏览器会新建一个文档，展示这个字符串内容，原有文档内容都会消失。如果返回不是字符串，浏览器不会新建文档，也不会跳转。

javascript: 协议的常见用途是书签脚本Bookmarklet。由于浏览器书签保存的是一个网址，所以javascript: 网址也可保存在里面，用户选择这个书签时，就会在当前页面执行这个脚本。为防止书签替换掉当前文档，可在脚本前加上void，或在脚本最后加上void 0。

```html
<a href="javascript: void new Date().toLocaleTimeString();">点击</a>
<a href="javascript: new Date().toLocaleTimeString();void 0;">点击</a>
```

- script元素

①工作原理

浏览器一边下载HTML网页，一边开始解析。即不等到下载完，就开始解析。👉解析过程中，浏览器发现`<script>`元素，就暂停解析，把网页渲染的控制权转交给JavaScript引擎。👉如果`<script>`元素引用了外部脚本，就下载该脚本再执行，否则直接执行代码。👉JavaScript引擎执行完毕，控制权交还渲染引擎，恢复往下解析HTML网页。

加载外部脚本时，浏览器暂停页面渲染，等待脚本下载并执行完成后再继续渲染。原因是JavaScript代码可修改DOM，所以必须把控制权让给它，否则会导致复杂的线程竞赛的问题。

如果外部脚本加载时间很长（一直无法完成下载），浏览器会一直等待脚本下载完成，造成网页长时间失去响应，浏览器呈现“假死”状态，称为“阻塞效应”。

为避免这种情况，最好将`<script>`标签都放在页面底部，而不是头部。这样即使遇到脚本失去响应，网页主体的渲染也已完成，用户至少可看到内容而不是空白页面。如果某些脚本代码一定要放在页面头部，最好直接将代码写入页面，而不是连接外部脚本文件，缩短加载时间。

脚本文件都放在网页尾部加载，另一好处是调用DOM节点时保证DOM已经生成。

🔺放在头部解决方案：①设定DOMContentLoaded事件回调函数。②使用`<script>`标签的onload属性。（指定外部脚本文件下载和解析完成触发load事件）

```html
<script src="a.js"></script>
<script src="b.js"></script>
```

如上，多个script标签，浏览器同时并行下载a.js和b.js。但执行时保证先执行a.js，然后再执行b.js，即使后者先下载完成。脚本执行顺序由它们在页面出现顺序决定，保证脚本之间依赖关系不受到破坏。必须等这两个脚本都加载完成，浏览器才继续页面渲染。

解析和执行CSS，也会产生阻塞。

对于来自同一个域名的资源，浏览器一般有限制，同时最多下载6-20个资源，即最多同时打开的TCP连接有限制，这是为了防止对服务器造成太大压力。如是来自不同域名的资源就没有限制，所以通常把静态文件放在不同的域名下，加快下载速度。

②defer属性

作用是延迟脚本的执行，等到DOM加载完成后，再执行脚本。

```html
<script src="a.js" defer></script>
<script src="b.js" defer></script>
```

defer属性运行流程：

浏览器开始解析HTML网页👉解析过程中，发现带有defer属性的`<script>`元素👉浏览器继续往下解析HTML网页，同时并行下载`<script>`元素加载的外部脚本👉浏览器完成解析HTML网页，此时再回过头执行已经下载完成的脚本。

有了defer属性，浏览器下载脚本文件时，不会阻塞页面渲染。下载的脚本文件在DOMContentLoaded事件触发前执行，而且可以保证执行顺序就是它们在页面出现顺序。

对内置脚本的`<script>`标签，以及动态生成的script标签，defer属性不起作用。使用defer加载的外部脚本不应使用document.write方法

③async属性

作用是使用另一个进程下载脚本，下载时不会阻塞渲染。

```html
<script src="a.js" async></script>
<script src="b.js" async></script>
```

async属性运行流程：

浏览器开始解析HTML网页👉解析过程中，发现有async属性的script标签👉浏览器继续往下解析HTML网页，同时并行下载`<script>`标签中外部脚本👉脚本下载完成，浏览器暂停解析HTML网页，开始执行下载的脚本👉脚本执行完毕，浏览器恢复解析HTML网页

async属性可保证脚本下载的同时，浏览器继续渲染。但无法保证脚本执行顺序，哪个脚本先下载结束就先执行那个脚本。使用async属性的脚本文件不应使用document.write方法。

⭐如果脚本间没有依赖关系，使用async属性，如果脚本间有依赖关系，使用defer属性。如果同时使用async和defer属性，后者不起作用，浏览器行为由async属性决定。

④脚本的动态加载

`<script>`元素可动态生成，生成后再插入页面，从而实现脚本动态加载。

```js
['a.js', 'b.js'].forEach(function(src) {
  var script = document.createElement('script');
  script.src = src;
  document.head.appendChild(script);
});
```

动态生成的script标签不会阻塞页面渲染，也就不会造成浏览器假死。但无法保证脚本执行顺序，哪个脚本先下载完成就先执行哪个。

```js
['a.js', 'b.js'].forEach(function(src) {
  var script = document.createElement('script');
  script.src = src;
  script.async = false;
  document.head.appendChild(script);
});
```

该写法不会阻塞页面渲染，可保证脚本执行顺序。但这段代码后面加载的脚本文件，会因此都等待b.js执行完成后再执行。

⑤加载使用的协议

如不指定协议，默认采用HTTP协议下载。如采用HTTPS协议下载必需写明。

根据页面本身协议决定加载协议：

```js
<script src="//example.js"></script>
```

- 浏览器的组成

①渲染引擎

将网页代码渲染为用户视觉可以感知的平面文档。

不同浏览器有不同渲染引擎。Chrome浏览器是Blink引擎。

渲染引擎处理网页，主要分四阶段：

解析代码：HTML代码解析成DOM，CSS代码解析成CSSOM👉对象合成：将DOM和CSSOM合成一棵渲染树👉布局：计算出渲染树的布局👉绘制：将渲染树绘制到屏幕

四阶段并非严格顺序执行，往往第一步还没完成，第二步和第三步就已开始。

②重流和重绘

渲染树转换为网页布局，称为“布局流”；布局显示到页面这个过程，称为“绘制”。都具有阻塞效应，并耗费很多时间和计算资源。

页面生成后，脚本操作和样式表操作，都会触发“重流”和“重绘”。用户的互动也会触发重流和重绘，如设置了鼠标悬停效果、页面滚动、在输入框输入文本、改变窗口大小等。

🔺重流必然导致重绘，重绘不一定需要重流。如改变元素颜色，只会重绘不会重流；改变元素布局会导致重绘和重流。

大多数情况下，浏览器智能判断，将重流和重绘只限制到相关的子树上，最小化所耗费的代价，而不会全局重新生成网页。

作为开发者，应尽量设法降低重绘次数和成本。如尽量不要变动高层DOM元素，而以底层DOM元素变动代替；比如重绘table布局和flex布局，开销会比较大。

⭐优化技巧

-读取DOM或写入DOM，尽量在一起，不要混杂。不要读取一个DOM节点，然后立即写入，接着再读取一个DOM节点。

-缓存DOM信息

-不要一项项改变样式，而是使用CSS class一次性改变样式

-使用documentFragment操作DOM

-动画使用absolute定位或fixed定位，可减少对其他元素影响

-只在必要时显示隐藏元素

-使用window.requestAnimationFrame()，因为它可把代码推迟到下一次重绘前执行，而不是立即要求页面重绘

-使用虚拟DOM（virtual DOM库）

③JavaScript引擎

主要作用：读取网页中的JavaScript代码，对其处理后运行。

JavaScript是一种解释型语言，即不需要编译，由解释器实时运行。好处是运行和修改都方便，刷新页面就可以重新解释；缺点是每次运行都要调用解释器，系统开销较大，运行速度慢于编译型语言。

为提高运行速度，目前浏览器对JavaScript进行一定程度编译，生成类似字节码的中间代码。

早期，浏览器内部对JavaScript处理过程如下：

读取代码，进行词法分析，将代码分解成词元👉对词元进行语法分析，将代码整理成“语法树”👉使用“翻译器”，将代码转为字节码👉使用“字节码解释器”，将字节码转为机器码

逐行解释将字节码转为机器码是很低效的，为提高运行速度，现代浏览器改为采用即时编译（JIT），即字节码只在运行时编译，用到哪一行就编译哪一行，并把编译结果缓存。

字节码不能直接运行，而是运行在一个虚拟机上，一般也把虚拟机称为JavaScript引擎。并非所有JavaScript虚拟机运行时都有字节码，有的JavaScript虚拟机基于源码，即只要有可能，就通过JIT编译器直接把源码编译成机器码运行。这是为了尽可能优化代码、提高性能。Chrome浏览器使用的JavaScript虚拟机是V8。

**18.2 window对象**

浏览器里，window对象指当前的浏览器窗口。也是当前页面顶层对象，一个变量如未声明，默认是顶层对象属性。

```js
a = 1;
window.a //1
```

- window对象的属性

window.name

字符串，当前浏览器窗口名字。主要配合超链接和表单target属性使用。

只能保存字符串，如果写入不是字符串，会自动转为字符串。

只要浏览器窗口不关闭，这个属性不会消失。一旦浏览器窗口关闭后，该属性保存的值就会消失。

window.closed、window.opener

window.closed：布尔值，窗口是否关闭。一般用来检查，使用脚本打开的新窗口是否关闭。

window.opener：打开当前窗口的父窗口。如当前窗口没有父窗口（直接地址栏输入打开），返回null。如果两个窗口间不需要通信，建议将子窗口的opener属性显式设为null。

通过opener属性，可获得父窗口全局属性和方法，但只限于两个窗口同源情况，且其中一个窗口由另一个打开。`<a>`元素添加 `rel="noopener"`属性，可防止新打开窗口获取父窗口，减轻被恶意网站修改父窗口URL的风险。

```js
window.open().opener === window //true，并打开新窗口
```

window.self、window.window

指向窗口本身。只读。

```js
window.self === window; // true
window.window === window; // true
```

window.frames、window.length

window.frames：返回类似数组的对象，成员是页面内所有框架窗口，包括frame元素和iframe元素。

如果iframe元素设置了id或name属性，可用属性值引用iframe窗口。如`<iframe name="aaa">`，可用frames['aaa']或frames.aaa引用。

frames属性实际上是window对象的别名。

因此frames[0]也可用window[0]表示，但建议使用frames表述。

```js
// <iframe name="iframe" src="a.html" id="iframe"></iframe>
// <iframe name="iframe2" src="b.html" id="iframe2"></iframe>
console.log(window.frames[1] === window.iframe2);//true
console.log(window.frames[0] === window['iframe']);//true
console.log(window.frames.length);//2
```

window.length：当前网页包含的框架总数。如当前网页不包含frame和iframe元素，该值为0。

```js
window.frames.length === window.length // true
```

window.frameElement

主要用于当前窗口嵌在另一网页情况（嵌入`<object>`、`<iframe>`、或`<embed>`元素），返回当前窗口所在的那个元素节点。如当前窗口是顶层窗口，或所嵌入的那个网页不是同源的，返回null。

```js
// HTML
// <iframe src="about.html"></iframe>

// about.html里面
var frameEl = window.frameElement;// iframe元素
if (frameEl) {
  frameEl.src = 'other.html';
}
```

window.top、window.parent

window.top：最顶层窗口，主要用于在框架窗口（frame）里获取顶层窗口

window.parent：指向父窗口。如当前窗口没有父窗口，window.parent指向自身。

对不包含框架的网页，这两个属性等同于window对象

window.status

读写浏览器状态栏的文本。不一定有效。

window.devicePixelRatio

数值，表示一个CSS像素的大小与一个物理像素的大小之间比率。即它表示一个CSS像素由多少个物理像素组成。可用于判断用户的显示环境，如这个比率较大，表示用户正在使用高清屏幕，因此可显示较大像素图片。

```js
window.devicePixelRatio // 1.25
```

位置大小属性

window.screenX、window.screenY：浏览器窗口左上角相对于当前屏幕左上角的水平（垂直）距离（单位像素）。只读。

window.innerHeight、window.innerWidth：网页在当前窗口中可见部分高度（宽度），即视口大小（单位像素）。只读。用户放大网页时，这两个属性会变小。这两个属性值包括滚动条高度和宽度。

window.outerHeight、window.outerWidth：浏览器窗口的高度（宽度），包括浏览器菜单和边框（单位像素）。只读。

window.scrollX、window.scrollY：页面水平（垂直）滚动距离。只读。返回值是双精度浮点数。如页面没滚动，返回0。

window.pageXOffset、window.pageYOffset：window.scrollX和window.scrollY别名。

组件属性

返回浏览器组件对象。

window.locationbar：地址栏对象

window.menubar：菜单栏对象

window.scrollbars：窗口滚动条对象

window.toolbar：工具栏对象

window.statusbar：状态栏对象

window.personalbar：用户安装的个人工具栏对象

这些对象的visible属性是布尔值，表示组件是否可见，只读。

全局对象属性

指向一些浏览器原生的全局对象

window.document：指向document对象，有同源限制，只有来自同源的脚本才能读取这个属性。

window.location：指向Location对象，用于获得当前窗口URL信息，等同于document.location属性。

window.navigator：指向Navigator对象，获取环境信息。

window.history：指向History对象，浏览器的浏览历史

window.localStorage：指向本地储存的localStorage数据。

window.sessionStorage：指向本地储存的sessionStorage数据。

window.console：指向console对象，操作控制台。

window.screen：指向Screen对象，屏幕信息。

window.isSecureContext

布尔值，当前窗口是否处在加密环境。如是HTTPS协议，返回true，否则就是false。

- window对象的方法

window.alert()、window.prompt()、window.confirm()

浏览器与用户互动的全局方法。弹出不同的对话框，要求用户做出回应。

window.alert()：只有一个“确定”按钮。用户只有点击按钮，对话框才消失。对话框弹出期间浏览器窗口处于冻结状态，如果不点按钮，用户什么也干不了。参数只能是字符串，没法使用CSS样式。

window.prompt()：提示文字的下方还有一个输入框，要求用户输入信息，并有“确定”和“取消”两个按钮。用来获取用户输入的数据。返回值有两种情况：①字符串（也可能空字符串）、②null。

用户输入信息，并点击“确定”，用户输入的信息就是返回值。

用户没输入信息，直接点击“确定”，输入框默认值就是返回值。

用户点击“取消”，或按下ESC按钮，返回值null。

window.confirm()：除提示信息外，只有“确定”和“取消”两个按钮，征询用户是否同意。返回布尔值。

🔺三个方法都具有堵塞效应，一旦弹出对话框，整个页面暂停执行，等待用户做出反应。

window.open()、window.close()、window.stop()

window.open()：新建另一个浏览器窗口，返回新窗口的引用，如无法新建窗口，则返回null。

```js
var windowB = window.open('a.html','a');
windowB.name // "a"
```

可接受三个参数：

```js
window.open(url, windowName, [windowFeatures])
```

url：字符串，新窗口地址。

windowName：字符串，新窗口名字。可指定已经存在的窗口，但不等于可任意控制其他窗口。为防止被不相干的窗口控制，浏览器只有在两个窗口同源，或目标窗口被当前网页打开情况下，才允许open方法指向该窗口。

windowFeatures：字符串，内容为逗号分隔的键值对，新窗口参数，如有没有提示栏、工具条等。

```js
var popup = window.open(
  'somepage.html',
  'DefinitionsWindows',
  'height=200,width=200,location=no,status=yes,resizable=yes,scrollbars=yes'
);
```

🔺如果新窗口和父窗口不是同源的，它们彼此不能获得对方窗口对象内部属性。

window.close()：关闭当前窗口，一般只用来关闭window.open方法新建的窗口。只对顶层窗口有效。

window.stop()：等同于单击浏览器的停止按钮，停止加载图像、视频等正在或等待加载的对象。

window.moveTo()、window.moveBy()

window.moveTo()：移动浏览器窗口到指定位置。接受两个参数，分别是窗口左上角距离屏幕左上角水平距离和垂直距离。

window.moveBy()：将窗口移动到一个相对位置。接受两个参数，分别是窗口左上角向右移动的水平距离和向下移动的垂直距离。

window.resizeTo()、window.resizeBy()

window.resizeTo()：缩放窗口到指定大小。接受两个参数，第一个是缩放后的窗口宽度，第二个是缩放后的窗口高度。

window.resizeBy()：按照相对量缩放窗口。接受两个参数，第一个是水平缩放的量，第二个是垂直缩放的量，单位都是像素。

```js
window.resizeBy(-200, -200)// 将当前窗口宽度和高度都缩小200像素
```

window.scrollTo()、window.scroll()、window.scrollBy()

window.scrollTo()：将文档滚动到指定位置。接受两个参数，表示滚动后位于窗口左上角的页面坐标。也可以接受一个配置对象作为参数。配置对象options有三个属性：top为滚动后页面左上角垂直坐标，即y坐标。left为滚动后页面左上角水平坐标，即x坐标。behavior为字符串，表示滚动方式，可能取值是smooth、instant、auto（默认）。

window.scroll()：window.scrollTo()别名

window.scrollBy()：将网页滚动指定距离（单位像素）。接受两个参数，水平向右的像素，垂直向下滚动的像素。

如果不是滚动整个文档，而是滚动某个元素，可采用：Element.scrollTop、Element.scrollLeft、Element.scrollIntoView()。

window.print()

跳出打印对话框，与用户点击菜单里“打印”命令效果相同。

window.focus()、window.blur()

window.focus()：激活窗口，使其获得焦点，出现在其他窗口前面。

window.blur()：将焦点从窗口移除。

window.getSelection()

返回Selection对象，表示用户现在选中的文本。使用Selection对象的toString()可得到选中文本。

window.getComputedStyle()、window.matchMedia()

window.getComputedStyle()：接受一个元素节点作参数，返回一个包含该元素的最终样式信息的对象。

window.matchMedia()：检查CSS的mediaQuery语句。

⭐window.requestAnimationFrame()

推迟某个函数的执行。推迟到浏览器下一次重流时执行，执行完才进行下一次重绘。如果某个函数会改变网页布局，一般放在该函数里执行，可节省系统资源，使网页效果更平滑。

该方法接受一个回调函数作为参数。

```js
window.requestAnimationFrame(callback)
```

callback执行时，参数就是系统传入的一个高精度时间戳，单位是ms，表示距离网页加载的时间。

返回值是整数，该返回值可传入window.cancelAnimationFrame()用来取消回调函数的执行。

🔺网页动画例子：

```js
var element = document.getElementById('animate');
element.style.position = 'absolute';

var start = null;

function step(timestamp) {
  if (!start) start = timestamp;
  var progress = timestamp - start;
  // 元素不断向左移，最大不超过200像素
  element.style.left = Math.min(progress / 10, 200) + 'px';
  // 如果距离第一次执行不超过 2000 毫秒，
  // 就继续执行动画
  if (progress < 2000) {
    window.requestAnimationFrame(step);
  }
}

window.requestAnimationFrame(step);
```

⭐window.requestIdleCallback()

将某个函数推迟执行，但它保证将回调函数推迟到系统资源空闲时执行。即如果某个任务不是很关键，就可使用该函数将其推迟执行，以保证网页性能。

和window.requestAnimationFrame()区别在于，后者指定回调函数在下一次浏览器重排时执行，问题在于下一次重排时，系统资源未必空闲，不一定能保证在16毫秒之内完成；window.requestIdleCallback()可保证回调函数在系统资源空闲时执行。

接受一个回调函数和一个配置对象作参数。配置对象可指定一个推迟执行的最长时间，如果过了这个事件，回调函数不管系统资源有无空闲，都会执行。

```js
window.requestIdleCallback(callback[, options])
```

callback参数是一个回调函数。该回调函数执行时，系统传入一个IdleDeadline对象作参数。IdleDeadline对象有一个didTimeout属性（布尔值，表示是否为超时调用）和一个timeRemaining()方法（返回该空闲时段剩余毫秒数）。

options参数是一个配置对象，只有timeout属性，指定回调函数推迟执行的最大毫秒数。

该方法返回整数，该整数可传入window.cancelIdleCallback()取消回调函数。

🔺实例：

```js
// 应用1
requestIdleCallback(myNonEssentialWork);

function myNonEssentialWork(deadline) {
  while (deadline.timeRemaining() > 0) {
    doWorkIfNeeded();
  }
}
// 应用2
requestIdleCallback(processPendingAnalyticsEvents, { timeout: 2000 });
```

如果由于超时导致回调函数执行，deadline.timeRemaining()返回0，deadline.didTimeout返回true。

如果多次执行window.requestIdleCallback()，指定多个回调函数，那么这些回调函数将排成一个队列，按照先进先出顺序执行。

- 事件

①load事件和onload属性

load事件发生在文档在浏览器窗口加载完毕时。window.onload属性可指定这个事件的回调函数。

②error事件和onerror属性

浏览器脚本发生错误时，触发window对象的error事件。可通过window.onerror属性对该事件指定回调函数。window的error事件的回调函数共接受五个参数：出错信息、出错脚本的网址、行号、列号、错误对象。

一般，只有JavaScript脚本的错误，才触发这个事件，而像资源文件不存在之类错误都不会触发。

🔺如果脚本网址与网页网址不在同一个域（如使用了CDN），浏览器根本不会提供详细的出错信息，只会提示除错，错误类型是“Script error."，行号为0，其他信息都没有。这是浏览器防止向外部脚本泄漏信息。一个解决方法是在脚本所在的服务器，设置HTTP头信息：

```js
Access-Control-Allow-Origin: *
```

然后，在网页的`<script>`标签中设置crossorigin属性：

```html
<script crossorigin="anonymous" src="//example.com/file.js"></script>
```

crossorigin=”anonymous“表示读取文件不需要身份信息，即不需要cookie和HTTP认证信息。

如设为crossorigin=”use-credentials“，表示浏览器会上传cookie和HTTP认证信息，同时还需要服务器端打开HTTP头信息Access-Control-Allow-Credentials。

③window对象的事件监听属性

除了具备元素节点都有的GlobalEventHandlers接口，window对象还具有以下事件监听函数属性：

window.onpagehide、window.onpageshow、window.onunload、window.onpopstate等

- 多窗口操作
- 网页可使用iframe元素嵌入其他网页，因此一个网页中会形成多个窗口。如果子窗口之中又嵌入别的网页，会形成多级窗口。

①窗口的引用

各个窗口之中的脚本，可引用其他窗口。浏览器提供了一些特殊变量，用来返回其他窗口。

top：顶层窗口，最上层那个窗口。

parent：父窗口。

self：当前窗口，自身。

与这些变量对应，浏览器还提供一些特殊窗口名，供window.open()方法、`<a>`标签、`<form>`标签等引用：

_top：顶层窗口、 _parent：父窗口、 _blank：新窗口。

```html
<a href="somepage.html" target="_top">Link</a>
<!--在顶层窗口打开链接-->
```

②iframe元素

使用contentWindow属性获得iframe节点包含的window对象。

contentDocument属性可得到子窗口的document对象。

`<iframe>`元素遵守同源政策，只有当父窗口与子窗口在同一个域时，两者之间才可用脚本通信，否则是由使用widnow.postMessage方法。

`<iframe>`窗口内部，使用window.parent引用父窗口。如当前页面没有父窗口，则window.parent属性返回自身。因此，可用window.parent是否等于window.self，判断当前窗口是否为iframe窗口。

`<iframe>`窗口的window对象，有一个frameElement属性，返回`<iframe>`在父窗口中的DOM节点。对于非嵌入的窗口，该属性为null。

```js
// <iframe name="iframe" src="a.html" id="iframe"></iframe>
var iframe = document.getElementById('iframe');
console.log(iframe.contentWindow.frameElement.nodeName);//IFRAME
```

③window.frames属性

可实现窗口之间的相互引用。如frames[1].frames[2]返回第二个子窗口内部的第三个子窗口。

每个成员的值是框架内的窗口（window对象），而不是iframe标签在父窗口的DOM节点。

如果`<iframe>`元素设置了name属性，name属性的值会自动称为子窗口名称，可用在window.open()的第二个参数，或者`<a>`和`<frame>`标签的target属性。

**18.3 Navigator对象、Screen对象**

window.navigator属性指向一个包含浏览器和系统信息的Navigator对象。脚本通过这个属性了解用户环境信息。

- Navigator对象的属性

Navigator.userAgent

返回浏览器User Agent字符串，表示浏览器厂商和版本信息。

一般不通过它识别浏览器，而是使用“功能识别”方法，即逐一测试当前浏览器是否支持要用到的JavaScript功能。

```js
navigator.userAgent
// "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"
```

识别所有移动设备的浏览器：

```js
/mobi|android|touch|mini/i.test(ua)
```

Navigator.plugins

返回类似数组的对象，成员是Plugin实例对象，表示浏览器安装的插件，如Flash、ActiveX等

Navigator.platform

返回用户操作系统信息，如MacIntel、Win32、Linux x86_64等

Navigator.onLine

布尔值，表示用户当前在线还是离线（浏览器断线）

有时，浏览器可连接局域网，但局域网不能连通外网。这时onLine属性返回true。所以不能假定只要是true，用户就一定能访问互联网。但如果是false，可断定用户一定离线。

用户变成在线触发online事件，变成离线触发offline事件。（window.online/window.offline)

Navigator.language，Navigator.languages

language属性返回字符串，浏览器首选语言。只读。

languages返回数组，用户可接受的语言。HTTP请求头信息的Accept-Language字段就来自这个数组。

如这个属性变化，会在window对象上触发languagechange事件。

Navigator.geolocation

返回一个Geolocation对象，包含用户地理位置信息，该API只有在HTTPS协议下可用，否则调用下面方法会报错。

Geolocation.getCurrentPosition()：得到用户当前位置

Geolocation.watchPosition()：监听用户位置变化

Geolocation.clearWatch()：取消watchPosition()指定的监听函数

调用这三个方法时，浏览器跳出一个对话框，要求用户给予授权。

Navigator.cookieEnabled

布尔值，表示浏览器的Cookie功能是否打开。

反映的是浏览器总的特性，与是否储存某个具体网站Cookie无关。用户可设置某个网站不得储存Cookie，cookieEnabled返回还是true。

- Navigator对象的方法

Navigator.javaEnabled()

布尔值，浏览器是否能运行Java Applet小程序。

Navigator.sendBeacon()

向服务器异步发送数据。

- Navigator的实验性属性

在部分浏览器可用

Navigator.deviceMemory

当前计算机内存数量（单位GB)。只读。只在HTTPS环境下可用。

返回值是近似值，四舍五入到最接近的2的幂，通常是0.25、0.5、1、2、4、8.实际内存超过8GB，也返回8。

Navigator.hardwareConcurrency

返回用户计算机上可用的逻辑处理器数量。只读。

现代计算机的CPU有多个物理核心，每个物理核心有时支持一次运行多个线程。因此，四核CPU可提供八个逻辑处理器核心。

该属性通过用于创建Web Worker，每个可用的逻辑处理器都创建一个Worker。

Navigator.connection

返回对象，包含当前网络连接的相关信息。

downlink：有效带宽估计值（单位：Mbps），四舍五入到每秒25kB的最接近倍数。

downlinkMax：当前连接的最大下行链路速度（单位：Mbps）

effectiveType：返回连接的等效类型，可能值为slow-2g、2g、3g、4g。

rtt：当前连接的估计有效往返时间，四舍五入到最接近的25毫秒的倍数。

saveData：用户是否设置了浏览器的减少数据使用量选项（如不加载图片），返回true或false。

type：当前连接的介质类型，可能值为bluetooth、cellular、ethernet、none、wifi、wimax、other、unknown。

- Screen对象

当前窗口所在的屏幕，提供显示设备的信息。window.screen指向这个对象。

有下面属性：

Screen.height，Screen.width：浏览器窗口所在的屏幕的高度（宽度）。

Screen.availHeight，Screen.availWidth：浏览器窗口可用的屏幕高度（宽度）。

Screen.pixelDepth：整数，屏幕的色彩位数，如24表示屏幕提供24位色彩。

Screen.colorDepth：Screen.pixelDepth别名。colorDepth表示应用程序颜色深度，pixelDepth表示屏幕颜色深度。

Screen.orientation：返回一个对象，表示屏幕方向。该对象的type属性是一个字符串，表示屏幕具体方向，landscape-primary表示横放，landscape-secondary表示颠倒的横放、portrait-primary表示竖放、portrait-secondary表示颠倒的竖放。

```js
screen.height //864
screen.pixelDepth //24
screen.width //1536
```

**18.4 Cookie**

Cookie是服务器保存在浏览器的一小段文本信息，一般大小不超4kB。浏览器每次向服务器发出请求，就会自动附上这段信息。

主要保存状态信息：

对话（session）管理：保存登录、购物车等需要记录的信息。

个性化信息：保存用户的偏好，如网页字体大小、背景色等

追踪用户：记录和分析用户行为

🔺客户端储存应该使用Web Storage API和IndexedDB，只有那些每次请求都需要让服务器知道的信息，才应该放在Cookie里。

每个Cookie都有以下几方面元数据：Cookie的名字、Cookie的值（真正的数据写在里面）、到期时间（超时会失效）、所属域名（默认当前域名）、生效的路径（默认当前网址）。

实例：用户访问网站www.a.com，服务器在浏览器写入一个Cookie。这个Cookie所属域名为www.a.com，生效路径为根路径/。如Cookie的生效路径设为/a，那么这个Cookie只有在访问www.a.com/a及其子路径才有效。以后，浏览器访问某个路径前，会找出对该域名和路径有效，并且还没到期的Cookie，一起发送给服务器。

用户可设置浏览器不接受Cookie，也可设置不向服务器发送Cookie。

不同浏览器对Cookie的数量和大小的限制不一样，一般单个域名设置的Cookie不超过30个，每个Cookie大小不超过4kB。

🔺浏览器同源政策规定，两个网址只要域名相同，就可共享Cookie，不要求协议相同。

- Cookie与HTTP协议

Cookie由HTTP协议生成，也主要供HTTP协议使用。

①HTTP回应：Cookie的生成

服务器如果希望希望在浏览器保存Cookie，就要在HTTP回应的头信息里，放置一个Set-Cookie字段。

HTTP回应可包含多个Set-Cookie字段，即在浏览器生成多个Cookie。

除了Cookie的值，Set-Cookie字段还可附加Cookie的属性。

```http
...
Set-Cookie: <cookie-name>=<cookie-value>; Expires=<date>
Set-Cookie: <cookie-name>=<cookie-value>; Max-Age=<non-zero-digit>
Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>
Set-Cookie: <cookie-name>=<cookie-value>; Path=<path-value>
Set-Cookie: <cookie-name>=<cookie-value>; Secure
Set-Cookie: <cookie-name>=<cookie-value>; HttpOnly
```

如果服务器想改变一个早先设置的Cookie，必须同时满足四个条件：Cookie的key、domain、path和secure都匹配。

例子：

```http
...
Set-Cookie: key1=value1; domain=example.com; path=/blog  （改变前）
Set-Cookie: key1=value2; domain=example.com; path=/  （改变后）
```

只要有一个属性不同，就会生成一个全新的Cookie，而不是替换原来Cookie

②HTTP请求：Cookie的发送

浏览器向服务器发送HTTP请求时，每个请求都会带上相应的Cookie。即把服务器早前保存在浏览器的这段信息，再发回服务器。这时要使用HTTP头信息的Cookie字段。

```http
。。。
Cookie: name=value; name2=value2; name3=value3
```

服务器收到浏览器发来的Cookie时，有两点无法知道：

Cookie各种属性，如何时过期。哪个域名设置的Cookie，到底是一级域名设的还是某一个二级域名设的。

- Cookie的属性

①Expires，Max-Age

Expires：指定一个具体的到期时间，到了指定时间后，浏览器不再保留这个Cookie。它的值是UTC格式。如不设置该属性，或设为null，Cookie只在当前会话（session）有效，浏览器窗口一旦关闭，当前Session结束，该Cookie就会被删除。浏览器根据本地时间决定Cookie是否过期，没办法保证Cookie一定会在服务器指定时间过期。

Max-Age：指定从现在开始Cookie存在的秒数。过了这个时间后浏览器不再保留这个Cookie。

🔺①如同时指定Expires和Max-Age，Max-Age的值优先生效。②如没有指定这两个属性，这个Cookie就是Session Cookie，即它只在本次对话存在，一旦用户关闭浏览器，就不再保留这个Cookie。

②Domain，Path

Domain：指定浏览器发出HTTP请求时，哪些域名要附带这个Cookie。如没有指定该属性，浏览器会默认将其设为当前域名，这时子域名将不会附带这个Cookie。

如a.com不设置Cookie的domain属性，那么b.a.com将不会附带这个Cookie。如指定了domain属性，子域名也会附带这个Cookie。如果服务器指定域名不属于当前域名，浏览器拒绝这个Cookie。

Path：指定浏览器发出HTTP请求时，哪些路径要附带这个Cookie。只要浏览器发现，Path属性是HTTP请求路径的开头一部分，就会在头信息里带上这个Cookie。如Path属性是/，请求/docs路径也会包含该Cookie。前提是域名必须一致。

③Secure，HttpOnly

Secure：指定浏览器只有在加密协议HTTPS下，才能将这个Cookie发送到服务器。如当前协议是HTTP，浏览器自动忽略服务器发来的Secure属性。该属性只是一个开关，不需要指定值。如通信HTTPS协议，开关自动打开。

HttpOnly：指定该Cookie无法通过JavaScript脚本拿到，主要是document.cookie属性、XMLHttpRequest对象和Request API都拿不到该属性。防止了该Cookie被脚本读到，只有浏览器发出HTTP请求时，才带上该Cookie。

④SameSite

防止CSRF攻击和用户追踪。

恶意网站可设法伪造带有正确Cookie的HTTP请求，这就是CSRF攻击。

SameSite用来限制第三方Cookie，减少安全风险。可设置三个值：Strict、Lax、None。

Strict：完全禁止第三方Cookie，跨站点时，任何情况下都不会发送Cookie。只有当前网页URL与请求目标一致，才带上Cookie。过于严格，可能造成非常不好的用户体验。

Lax：大多数情况下不发送第三方Cookie，但导航到目标网址的Get请求除外。

导航到目标网址的GET请求，只包括三种情况：连接，预加载请求，GET表单。

None：网站可显式关闭SameSite属性，将其设为None。前提是必须同时设置Secure属性（Cookie只能通过HTTPS协议发送），否则无效。

```http
...
Set-Cookie: widget_session=abc123; SameSite=None; Secure  （有效）
```

- document.cookie

读写当前网页Cookie。

读取时，返回当前网页所有Cookie，前提是该Cookie不能有HttpOnly属性。

写入时，Cookie值必须写成key=value形式。等号两边不能有空格。必须对分号、逗号和空格进行转义（它们都不允许作为Cookie的值），这可以用encodeURIComponent方法达到。

一次只能写入一个Cookie，而且写入不是覆盖而是添加。

```js
document.cookie = "key1=1";
document.cookie = "key2=2";
document.cookie // "key1=1;key2=2" 
```

🔺读写行为的差异，与HTTP协议的Cookie通信格式有关。浏览器向服务器发送Cookie时，Cookie字段使用一行将所有Cookie全部发送；服务器向浏览器设置Cookie时，Set-Cookie字段是一行设置一个Cookie。

写入时，可一起写入Cookie的属性，属性值等号两边一样不能有空格。

🔺①path属性必须为绝对路径，默认当前路径。②domain属性必须是当前发送Cookie的域名的一部分。如当前域名为a.com，不可设为b.com。默认为当前的一级域名（不含二级域名）。

Cookie属性一旦设置完成，没有办法读取这些属性值。

删除一个现存Cookie唯一方法：设置它的Expires属性为一个过去的日期。

**18.5 XMLHttpRequest对象**

AJAX包括以下步骤：创建XMLHttpRequest实例👉发出HTTP请求👉接收服务器传回的数据👉更新网页数据

XMLHttpRequest对象是AJAX主要接口，用于浏览器和服务器间通信。可使用多种协议，发送任何格式数据。

XMLHttpRequest本身是构造函数，可使用new命令生成实例，没有任何参数：

```js
var xhr = new XMLHttpRequest();
```

一旦新建实例，就可使用open()指定建立HTTP连接一些细节。

然后指定回调函数，监听通信状态（readyState属性）变化。

最后使用send()方法，实际发出请求。

一旦拿到服务器返回的数据，AJAX不会刷新整个网页，而是只更新网页里相关部分，从而不打断用户正在做的事情。

🔺AJAX只能向同源网址（协议、域名、端口都相同）发出HTTP请求。

```js
var xhr = new XMLHttpRequest();

xhr.onreadystatechange = function(){
  // 通信成功时，状态值为4
  if (xhr.readyState === 4){
    if (xhr.status === 200){
      console.log(xhr.responseText);
    } else {
      console.error(xhr.statusText);
    }
  }
};

xhr.onerror = function (e) {
  console.error(xhr.statusText);
};

xhr.open('GET', '/endpoint', true);
xhr.send(null);
```

- XMLHttpRequest实例属性

XMLHttpRequest.readyState

返回整数，表示实例对象当前状态，只读。

0：XMLHttpRequest实例已生成，但实例open()方法还没调用

1：open()已经调用，但send()还没调用，仍可使用实例的setRequestHeader()设定HTTP请求头信息

2：send()已经调用，服务器返回的头信息和状态码已收到

3：正在接收服务器传来的数据体（body部分）。如实例responseType属性等于text或者空字符串，responseText属性会包含已经收到的部分信息。

4：服务器返回的数据已经完全接收，或本次接收已经失败。

🔺这个值每一次变化，都会触发readyStateChange事件。

XMLHttpRequest.onreadystatechange

readystatechange事件发生时，会执行这个属性。

如使用实例的abort()，终止XMLHttpRequest请求，也会造成readyState属性变化。

XMLHttpRequest.response

服务器返回的数据体（HTTP回应的body部分）。只读。

如本次请求没成功或数据不完整，该属性等于null。但如果responseType属性等于text或空字符串，再请求没结束前，response属性包含服务器已经返回的部分数据。

XMLHttpRequest.responseType

字符串，服务器返回数据的类型。可写。可在调用open()之后，send()之前，设置这个属性值，告诉服务器返回指定类型数据。如responseType设为空字符串，等同于默认值text。

该属性可取以下值：

“”（空字符串）：等同于text，服务器返回文本呢数据

“arraybuffer”：ArrayBuffer对象，返回二进制数组

“blob”：Blob对象，返回二进制对象

“document”：Document对象，返回文档对象

“json”：JSON对象

“text”：字符串

🔺text类型适合大多数情况，直接处理文本较方便。document类型适合返回HTML/XML文档情况。对于那些打开CORS的网站，可直接用Ajax抓取网页，然后不用解析HTML字符串，直接对抓取回来的数据进行DOM操作。blob类型适合二进制数据，如图片文件。

XMLHttpRequest.responseText

返回从服务器接收到的字符串，只读。只有HTTP请求完成接收后，该属性才会包含完整数据。

XMLHttpRequest.responseXML

返回从服务器接收到的HTML或XML文档对象，只读。如本次请求没成功，或收到的数据不能被解析成XML或HTML，该属性为null。

该属性生效前提是HTTP回应的Content-Type头信息等于text/xml或application/xml。要求再发送请求前，responseType要设为document。如HTTP回应的Content-Type头信息不等于text/xml和application/xml，但想从responseXML拿到数据（即把数据按照DOM格式解析），需要手动调用XMLHttpRequest.overrideMimeType()，强制进行XML解析。

该属性得到的数据，是直接解析后的文档DOM树。

XMLHttpRequest.responseURL

字符串，发送数据的服务器网址

🔺这个值与open()指定的请求网址不一定相同。如服务器端发生跳转，这个属性返回最后实际返回数据的网址。如原始URL包括锚点，该属性会把锚点剥离。

XMLHttpRequest.status、XMLHttpRequest.statusText

status返回整数，服务器回应的HTTP状态码。如通信成功，状态码是200；如服务器没返回状态码，这个属性默认200.请求发出前，该属性为0，只读。

基本上，只有2xx和304的状态码，表示服务器返回是正常状态。

statusText返回字符串，服务器发送的状态提示，该属性包含整个状态信息。请求发送前，该属性值为空字符串；如服务器没返回状态提示，默认为“OK”，只读。

XMLHttpRequest.timeout、XMLHttpRequestEventTarget.ontimeout

timeout返回整数，表示多少毫秒后，如请求仍没有得到结果，就自动终止。

ontimeout用于设置一个监听函数，如果发生timeout事件，就执行这个监听函数

事件监听属性

XMLHttpRequest.onloadstart：loadstart事件（HTTP请求发出）的监听函数

XMLHttpRequest.onprogress：progress事件（正在发送和加载数据）的监听函数

XMLHttpRequest.onabort：abort事件（请求中止，如调用abort()）的监听函数

XMLHttpRequest.onerror：error事件（请求失败）的监听函数

XMLHttpRequest.onload：load事件（请求成功完成）的监听函数

XMLHttpRequest.ontimeout：timeout事件（用户指定时限已过，请求还未完成）的监听函数

XMLHttpRequest.onloadend：loadend事件（请求完成，不管成功或失败）的监听函数

🔺progress事件监听函数有一个事件对象参数，该对象有三个属性：loaded返回已经传输数据量、total返回总的数据量、lengthComputable返回布尔值，表示加载进度是否可计算。只有progress事件的监听函数有参数，其他函数都无参数。

XMLHttpRequest.withCredentials

布尔值，表示跨域请求时，用户信息（如Cookie和认证的HTTP头信息）是否会包含在请求中，默认false。

如需要跨域AJAX请求发送Cookie，需要设为true。同源请求不需要设置该属性。

为使属性生效，服务器必须显式返回Access-Control-Allow-Credentials头信息。

withCredentials属性打开，跨域请求不仅会发送Cookie，还会设置远程主机指定的Cookie，反之亦然。如withCredentials属性没有打开，跨域AJAX请求即使明确要求浏览器设置Cookie，浏览器也会忽略。

XMLHttpRequest.upload

通过AJAX文件上传，发送文件后，通过该属性得到一个对象，通过观察这个对象可得知上传进展。主要方法是监听这个对象各种事件：loadstart、loadend、load、error、abort、progress、timeout。

- 实例方法

XMLHttpRequest.open()

指定HTTP请求的参数，或说初始化XMLHttpRequest实例对象。可接受五个参数：

method：HTTP动词方法，GET、POST等

url：请求发送目标URL

async：布尔值，请求是否为异步，默认true。如设为false，send()只有等到收到服务器返回路结果才进行下一步操作。可选。

user：用于认证的用户名，默认空字符串。可选。

password：用于认证的密码，默认空字符串。可选。

🔺如对使用过open()的AJAX请求，再次使用该方法，等同于调用abort()，终止请求。

```js
var xhr = new XMLHttpRequest();
xhr.open('POST', encodeURI('someURL'));
```

XMLHttpRequest.send()

实际发出HTTP请求。如不带参数，表示HTTP请求只有一个URL，没有数据体，如GET请求。如带参数，表示除了头信息，还带有包含具体数据的信息体，如POST请求。

🔺所有XMLHttpRequest的监听事件，都必须在send()调用前设定。

send方法的参数就是发送的数据。多种格式数据都可作为参数。如果send()发送DOM对象，在发送前，数据会先被串行化。如发送二进制数据，最好是发送ArrayBufferView或Blob对象，这使得通过Ajax上传文件成为可能。

XMLHttpRequest.setRequestHeader()

设置浏览器发送的HTTP请求的头信息。必须在open()之后，send()之前调用。

如该方法多次调用，设定同一个字段，则每一次调用的值会被合并成一个单一值发送。

接受两个参数，第一个参数是字符串，表示头信息的字段名，第二个参数是字段值。

```js
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.setRequestHeader('Content-Length', JSON.stringify(data).length);
xhr.send(JSON.stringify(data));
```

XMLHttpRequest.overrideMimeType()

指定MIME类型，覆盖服务器返回的真正MIME类型，从而让浏览器进行不一样的处理。

🔺该方法必须在send()之前调用

如果希望服务器返回指定数据类型，可用responseType属性告诉服务器。只有在服务器无法返回某种数据类型时，才使用该方法。

XMLHttpRequest.getResponseHeader()

返回HTTP头信息指定字段值，如还没有收到服务器回应或者指定字段不存在，返回null。参数不区分大小写。

XMLHttpRequest.getAllResponseHeaders()

返回字符串，表示服务器发来的所有HTTP头信息。格式为字符串，每个头信息间使用CRLF分隔（回车+换行），如没有收到服务器回应，该属性为null。如发送网络错误，该属性为空字符串。

XMLHttpRequest.abort()

终止已经发出的HTTP请求。调用该方法后，readyState属性变，status属性变为0。

- 实例事件

readyStateChange事件

readyState的值发生改变。通过onReadyStateChange属性，指定这个事件监听函数。

progress事件

上传文件时，XMLHttpRequest实例对象本身和实例upload属性，都有一个progress事件，不断返回上传进度。

load事件、error事件、abort事件

load事件表示服务器传来的数据接收完毕，error事件表示请求出错，abort事件表示请求被中断（如用户取消请求）。

loadend事件

abort、load和error这三个事件，会伴随一个loadend事件，表示请求结束，但不知道其是否成功

timeout事件

服务器超过指定时间还没有返回结果，就会触发timeout事件。

- Navigator.sendBeacon()

用户卸载网页时，有时需要向服务器发一些数据。使用Navigator.sendBeacon()，这个方法还是异步发出请求，但是请求与当前页面线程脱钩，作为浏览器进程的任务，因此可保证会把数据发出去，不拖延卸载流程。

接受两个参数，第一个参数是目标服务器URL，第二个参数是所要发送的数据（可选），可是任意类型（字符串、表单对象、二进制对象等）。

返回值是一个布尔值，成功发送数据为true，否则为false。

该方法发送数据的HTTP方法是POST，可跨域，类似于表单提交数据。不能指定回调函数。

```js
// HTML 代码如下
// <body onload="analytics('start')" onunload="analytics('end')">

function analytics(state) {
  if (!navigator.sendBeacon) return;

  var URL = 'http://example.com/analytics';
  var data = 'state=' + state + '&location=' + window.location;
  navigator.sendBeacon(URL, data);
}
```

**18.6 同源限制**

最初含义是，A网页设置的Cookie，B网页不能打开，除非这两个网页”同源“。

”同源“指：协议相同、域名相同、端口相同。

实例：http://www.a.com/dir/a.html。协议是http://，域名是www.a.com，端口是80（默认省略）。

标准规定端口不同的网址不是同源，但浏览器没有遵守这条规定。同一网域的不同端口，是可以互相读取Cookie的。

同源政策的目的：保证用户信息安全，防止恶意网站窃取数据。

同源政策的限制：如果非同源，三种行为受限制，无法读取非同源网页的Cookie、LocalStorage和IndexedDB；无法接触非同源网页的DOM；无法向非同源地址发送AJAX请求（可发送但浏览器拒绝接受响应）。

- Cookie

Cookie是服务器写入浏览器的一小段信息，只有同源网页才能共享。如两个网页一级域名相同，只是二级域名不同，浏览器允许设置document.domain共享Cookie。

实例：A网页网址为：http://w1.example.com/a.html，B网页网址为：http://w2.example.com/b.html。只要设置相同的document.domain，两个网页就可共享Cookie。因为浏览器通过document.domain属性检查是否同源。

```js
// 两个网页都需要设置，因为设置document.domain同时，端口重置为null
document.domain = 'example.com';
```

只适用于Cookie和iframe窗口，LocalStorage和IndexedDB无法通过这种方法。

🔺服务器也可以在设置Cookie时，指定Cookie的所属域名为一级域名：

```js
Set-Cookie: key=value; domain=.example.com; path=/
```

这样，二级域名和三级域名不用做任何设置，都可读取这个Cookie。

- iframe和多窗口通信

只有在同源情况下，父窗口和子窗口才能通信；如果跨域，无法拿到对方DOM

如果两个窗口一级域名相同，s只是二级域名不同，设置document.domain属性可规避同源政策，拿到DOM。

对完全不同源网站，目前两种方法解决跨域窗口通信问题：

片段识别符、跨文档通信API。

①片段识别符

指URL的#号后面部分。如果只是改变片段标识符，页面不会重新刷新。

父窗口可把信息写入子窗口片段标识符，子窗口通过hashchange时间得到通知。子窗口也可改变父窗口片段标识符：

```js
// 1
var src = originURL + '#' + data;
document.getElementById('myIFrame').src = src;
// 2
parent.location.href = target + '#' + hash;
```

②window.postMessage()

方法第一个参数是具体的信息内容，第二个参数是接收消息的窗口的源（origin），即”协议+域名+端口“，也可设为*，表示不限制域名，向所有窗口发送。

父窗口和子窗口都可通过message事件，监听对方信息。

message事件的参数是事件对象event，提供三个属性：

event.source：发送消息的窗口

event.origin：消息发向的网址

event.data：消息内容

③LocalStorage

通过window.postMessage，读写其他窗口的LocalStorage也可以。

- AJAX

同源政策规定，AJAX请求只能发给同源网址，否则报错。

除了架设服务器代理（浏览器请求同源服务器，再由后者请求外部服务），有三种方法规避限制：

JSONP、WebSocket、CORS

①JSONP

服务器与客户端跨源通信常用方法。简单易用，没有兼容性问题。

第一步，网页添加一个`<script>`元素，向服务器请求一个脚本，这不受同源政策限制，可跨域请求。

第二步，服务器收到请求后，拼接一个字符串，将JSON数据放在函数名里，作为字符串返回。

第三步，客户端会将服务器返回的字符串，作为代码解析，因为浏览器认为，这是`<script>`标签请求的脚本内容。这时，客户端只要定义了某个函数，就能在函数体内，拿到服务器返回的JSON数据。

```js
function addScriptTag(src) {
  var script = document.createElement('script');
  script.setAttribute('type', 'text/javascript');
  script.src = src;
  document.body.appendChild(script);
}

window.onload = function () {
  addScriptTag('http://example.com/ip?callback=foo');
}

function foo(data) {
  console.log('Your public IP address is: ' + data.ip);
};
```

服务器收到这个请求后，将数据放在回调函数参数位置返回：

```js
foo({
  'ip': '8.8.8.8'
});
```

②WebSocket

一种通信协议，使用ws://（非加密）和wss://（加密）作为协议前缀。该协议不实行同源政策，只要服务器支持，就可通过它进行跨源通信。

```http
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Origin: http://example.com
```

Origin表示请求的请求源，即发自哪个域名。

服务器可根据这个字段，判断是否许可本次通信。如果该域名在白名单内，服务器做出如下回应：

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat
```

③CORS

跨源资源分享。是W3C标准，属于跨源AJAX请求的根本解决方法。相比JSONP只能发GET请求，CORS允许任何类型请求。

**18.7 CORS通信**

允许浏览器向跨域的服务器，发出XMLHttpRequest请求，克服了AJAX只能同源使用的限制。

CORS需要浏览器和服务器同时支持。

整个CORS通信过程，都是浏览器自动完成，不需要用户参与。对开发者来说，CORS通信与普通的AJAX通信没差别，代码完全一样。浏览器一旦发现AJAX请求跨域，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感知。实现CORS通信关键是服务器。只要服务器实现了CORS接口，就可跨域通信。

- 两种请求

CORS请求分成两类：简单请求和非简单请求。

只要同时满足以下两大条件，就属于简单请求：

请求方法是以下三种方法之一：HEAD、GET、POST

HTTP头信息不超出以下几种字段：Accept、Accept-Language、Content-Language、Last-Event-ID、Content-type：只限于三个值：application/x-www-form-urlencoded、multipart/form-data、text/plain

凡是不满足上面两个条件，就是非简单请求。

- 简单请求

①基本流程

浏览器直接发出CORS请求。在头信息中，增加一个Origin字段。

```http
GET /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```

Origin字段说明，本次请求来自哪个域（协议+域名+端口）。服务器根据这个值，决定是否同意这次请求。

如Origin指定的源，不在许可范围内，服务器返回一个正常HTTP回应。浏览器发现，这个回应的头信息没包含Access-Control-Allow-Origin字段，就知道出错了，从而抛出一个错误，被XMLHttpRequest的onerror回调函数捕获。

如果Origin指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段：

```http
...
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8
```

Access-Control-Allow-Origin：必须。值要么是请求时Origin字段值，要么是一个 * ，表示接受任意域名的请求。

Access-Control-Allow-Credentials：可选。布尔值，表示是否允许发送Cookie。默认，Cookie不包含在CORS请求中。设为true，表示浏览器可把Cookie包含在请求中一起发给服务器。这个值也只能设为true，如服务器不要浏览器发送Cookie，不发送该字段即可。

Access-Control-Expose-Headers：可选。CORS请求时，XMLHttpRequest对象的getResponseHeader()只能拿到6个服务器返回的基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。如想拿到其他字段，必须在Access-Control-Expose-Headers里指定。

②withCredentials属性

浏览器可以发送Cookie，除了服务器显式指定Access-Control-Allow-Credentials字段，开发者必须在AJAX请求中打开withCredentials属性。

如果服务器要求浏览器发送Cookie，Access-Control-Allow-Origin就不能设为星号，必须指定明确的、与请求网页一致的域名。同时，Cookie依然遵循同源政策，只有用服务器域名设置的Cookie才会上传，其他域名的Cookie并不会上传，且（跨域）原网页代码中的document.cookie也无法读取服务器域名下的Cookie。

- 非简单请求

①预检请求

非简单请求的CORS请求，会在正式通信前，增加一次HTTP查询请求，称为“预检”请求。浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可用哪些HTTP方法和头信息字段。只有得到肯定回复，浏览器才会发出正式的XMLHttpRequest请求，否则报错。

这是为了防止这些新增的请求，对传统的没有CORS支持的服务器形成压力，给服务器一个提前拒绝机会。

```http
OPTIONS /cors HTTP/1.1
Origin: http://api.bob.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```

”预检“请求用的请求方法是OPTIONS，表示这个请求是用来询问的。头信息里，关键字段是Origin，表示请求来自哪个源。”预检“请求的头信息包括两个特殊字段：

Access-Control-Request-Method：必须，列出浏览器的CORS请求会用到哪些HTTP方法。

Access-Control-Request-Headers：逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段。

②预检请求的响应

服务器收到”预检“请求后，检查了Origin、Access-Control-Request-Method和Access-Control-Request-Headers字段后，确认允许跨源请求，就可做出回应。

```http
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://api.bob.com  （关键）
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```

如果服务器否定了”预检“请求，返回正常的HTTP回应，但没有任何CORS相关的头信息字段，或明确表示请求不符合条件。这时，会触发一个错误，被XMLHttpRequest对象的onerror回调函数捕获。

服务器回应的其他CORS相关字段：

```http
...
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000
```

Access-Control-Allow-Methods：必须，逗号分隔的字符串。表明服务器支持的所有跨域请求的方法。返回的是所有支持的方法，而不单是浏览器请求的那个方法。

Access-Control-Allow-Headers：如果浏览器请求包括Access-Control-Request-Headers字段，则Access-Control-Allow-Headers字段必需。逗号分隔的字符串。表明服务器支持的所有头信息字段，不限于浏览器在”预检“中请求的字段。

Access-Control-Allow-Credentials：与简单请求时含义相同

Access-Control-Max-Age：可选，指定本次预检请求的有效期，单位为秒。

③浏览器的正常请求和回应

一旦服务器通过了”预检“请求，以后每次浏览器正常的CORS请求，就跟简单请求一样，会有一个Origin头信息字段。服务器的回应，也都有一个Access-Control-Allow-Origin头信息字段。

- 与JSONP比较

JSONP只支持GET请求，CORS支持所有类型HTTP请求。

JSONP优势在于支持老式浏览器，以及可向不支持CORS的网站请求数据。

**18.8 Storage接口**

脚本在浏览器保存数据。两个对象都部署了这个接口：window.sessionStorage和window.localStorage。

sessionStorage保存的数据用于浏览器的一次会话，当会话结束（窗口关闭），数据被清空。

localStorage保存的数据长期存在，下一次访问该网站时，网页直接读取以前保存的数据。

除了保存期限长短不同，这两个对象其他方面都一致。

保存的数据都以”键值对“形式存在，所有数据都是以文本格式保存。

这个接口很像Cookie强化版，能使用大得多的存储空间。与Cookie一样，也受同域限制。

- 属性和方法

只有一个属性：

Storage.length：保存的数据项个数。

方法：

Stoage.setItem()：存入数据。第一个参数是键名，第二个参数是保存的数据。如果键名已经存在，会更新已有键值。方法没有返回值。两个参数都是字符串。

写入不一定要用该方法，直接赋值也可以。

Storage.getItem()：读取数据，参数是键名。如键名不存在，返回null。

Storage.removeItem()：清除某个键名对应键值。参数是键名。如键名不存在，方法不做任何事情。

Storage.clear()：清除所有保存的数据。返回值是undefined。

Storage.key()：参数为整数（从0开始），返回该位置对应的键值。

```js
localStorage.setItem('aaa','1');
localStorage.setItem('bbb','2');
console.log(localStorage.getItem('aaa'),localStorage.key(1),localStorage.length);//'1' 'bbb' 2
```

- storage事件

Storage接口储存的数据发生变化时，触发storage事件。

监听函数接受一个event实例对象作为参数，这个实例对象继承了StorageEvent接口，有几个特有属性，都只读。

StorageEvent.key：字符串，发生变动的键名。如storage事件由clear()引起，返回null。

StorageEvent.newValue：字符串，新的键值。如storage事件由clear()引起，返回null。

StorageEvent.oldValue：字符串，旧的键值。如该键值对新增，返回null。

StorageEvent.storageArea：对象，返回键值对所在的整个对象。可从这个属性拿到当前域名储存的所有键值对。

StorageEvent.url：字符串，原始触发storage事件的那个网页网址。

🔺该事件不在导致数据变化的当前页面触发，而是在同一个域名下其他窗口触发。同时打开多个窗口时，当其中一个窗口导致储存的数据发送改变时，只有在其他窗口才能观察到监听函数执行。

**18.9 History对象**

window.history属性指向History对象，表示当前窗口浏览历史。

浏览器不允许脚本读取这些地址，但允许在地址之间导航。

浏览器工具栏的”前进“和”后退“按钮，就是对History对象进行操作。

- 属性

History.length：当前窗口访问过的网址数量（包括当前网页）

History.state：History堆栈最上层状态值。

- 方法

History.back()、History.forward()、History.go()

在历史之中移动。

History.back()：移动到上一个网址。

History.forward()：移动到下一个网址。

History.go()：接受一个整数作参数，以当前网址为基准，移动到参数指定网址。如go(1)相当于forward()、go(-1)相当于back()。如不指定参数，默认为0，相当于刷新当前页面。

🔺移动到以前访问过的页面时，页面通常是从浏览器缓存中加载，而不是重新要求服务器发送新的网页。

History.pushState()

在历史中添加一条记录。

```js
window.history.pushState(state, title, url)
```

三个参数：state为一个与添加的记录相关联的状态对象，主要用于popstate事件。该事件触发时，对象会传入回调函数。浏览器会将这个对象序列化后保留本地，重新载入这个页面时可拿到这个对象。如不需要这个对象，可填null。title为新页面标题，可填空字符串。url为新网址，必须与当前页面处在同一个域。防止恶意代码让用户以为他们是在另一个网站上，因为该方法不导致页面跳转。

```js
history.pushState({a:1},'','2.html')
history.state // {a:1}
```

🔺①pushState()不会触发页面刷新，只是导致History对象发生变化，地址栏会有反应。②如果url参数设置了一个新的锚点值（hash），并不会触发hashchange事件。如果url锚点值变了，则会在History对象创建一条浏览记录。

History.replaceState()

修改History对象当前记录，其他都与pushState()一致。

- popstate事件

每当同一个文档浏览历史出现变化时，触发事件。

仅仅调用pushState()或replaceState()方法，不会触发；只有用户点击浏览器倒退按钮和前进按钮，或使用JavaScript调用History.back()、History.forward()、History.go()才会触发。

回调函数的参数是一个event事件对象，state属性指向pushState和replaceState方法为当前URL所提供的状态对象。

🔺页面第一次加载时，浏览器不会触发popstate事件。

**18.10 Location对象、URL对象、URLSearchParams对象**

- Location对象

浏览器提供的原生对象，提供URL相关信息和操作方法。通过window.location和document.location属性，可拿到该对象。

①属性

Location.href：整个URL

Location.protocol：当前URL协议，包括冒号

Location.host：主机。如端口不是协议默认的80和433，还会包含冒号和端口

Location.hostname：主机名，不包括端口

Location.port：端口号

Location.pathname：URL路径部分，从根路径 / 开始

Location.search：查询字符串部分，从问号 ? 开始

Location.hash：片段字符串部分，从 # 号开始

Location.username：域名前面的用户名

Location.password：域名前面的密码

Location.origin：URL的协议、主机名和端口

```js
console.log(document.location.host,document.location.protocol,document.location.origin);
// "localhost:63342" "http:" "http://localhost:63342"
```

🔺①只有origin属性是只读，其他属性都可写。②如对Location.href写入新的URL地址，浏览器立刻跳转到新地址。③Location.href是浏览器唯一允许跨域写入的属性，即非同源窗口可改写另一窗口的Location.href属性，导致后者网址跳转。

②方法

Location.assign()

接受URL字符串为参数，使得浏览器立刻跳转到新URL。

Location.replace()

接受URL字符串为参数，使得浏览器立刻跳转到新的URL。与assign()区别是，replace会在浏览器浏览历史History里删除当前网址，即后退按钮无法回到当前网页。

Location.reload()

浏览器重新加载当前网址，相当于按下浏览器刷新按钮。接受布尔值为参数，如为true，浏览器向服务器重新请求这个网页，并且重新加载后，网页滚动到头部。如为false或空，浏览器从本地缓存重新加载网页，视口位置是重新加载前位置。

Location.toString()

返回整个URL字符串

- URL编码和解码

网页URL只能包含合法字符：

URL元字符：； ， / ? : @ = + $ #

语义字符：a-z、A-Z、0-9、- _ . ! ~ * ' ()

除了以上字符，其他字符出现在URL之中都必须转义，规则是根据操作系统默认编码，将每个字节转为百分号加上两个大写十六进制字母。

四个编码/解码方法：

①encodeURI()

转码整个URL，参数是字符串，代表整个URL。将元字符和语义字符外的字符都转义

②encodeURIComponent()

转码URL组成部分，转码除了语义字符外所有字符，参数是URL的片段

③decodeURI()

整个URL解码，参数是转码后URL

④decodeURIComponent()

URL片段的解码，参数是转码后的URL片段

- URL接口

浏览器原生提供URL()接口，构造、解析和编码URL。通过window.URL可拿到这个构造函数。

①构造函数

可生成URL实例，参数是URL字符串。

URL()参数也可是另一个URL实例，URL()自动读取该实例href属性作为实参。

如URL字符串是一个相对路径，需要表示绝对路径的第二个参数作为计算基准。

```js
var url2 = new URL('page2.html', 'http://example.com/page1.html');
url2.href
// "http://example.com/page2.html"

var url3 = new URL('..', 'http://example.com/a/b.html') // ..表示上层路径
url3.href
// "http://example.com/"
```

②实例属性

URL.href：返回整个URL

URL.protocol：返回协议，以冒号结尾

URL.hostname：返回域名

URL.host：返回域名与端口，包含冒号，默认80和443端口会省略

URL.port：返回端口

URL.origin：返回协议、域名和端口

URL.pathname：返回路径，以斜杠开头

URL.search：返回查询字符串，以问号开头

URL.searchParams：返回URLSearchParams实例

URL.hash：返回片段识别符，以井号开头

URL.password：返回域名前的密码

URL.username：返回域名前的用户名

🔺只有origin属性只读，其他属性都可写，并且立即生效。

③静态方法

URL.createObjectURL()

为上传/下载的文件、流媒体文件生成一个URL字符串，这个字符串代表了File对象或Blob对象的URL。

每次使用该方法，都会在内存里生成一个URL实例。如不再需要该方法生成的URL字符串，为节省内存，可使用URL.revokeObjectURL()释放这个实例。

URL.revokeObjectURL()

释放URL.createObjectURL()生成的URL实例，参数是URL.createObjectURL()返回的URL字符串。

- URLSearchParams对象

URLSearchParams对象是浏览器原生对象，用来构造、解析和处理URL的查询字符串

本身是一个构造函数，可生成实例。参数可为查询字符串，起首问号有没有都行，也可是对应查询字符串的数组或对象。

```js
// 方法一：传入字符串
var params = new URLSearchParams('?foo=1&bar=2');
// 等同于
var params = new URLSearchParams(document.location.search);

// 方法二：传入数组
var params = new URLSearchParams([['foo', 1], ['bar', 2]]);

// 方法三：传入对象
var params = new URLSearchParams({'foo' : 1 , 'bar' : 2});
```

URLSearchParams会对查询字符串自动编码

浏览器向服务器发送表单数据时，可直接使用URLSearchParams实例作为表单数据。

URLSearchParams实例有遍历器接口，可用for...of循环遍历

🔺URLSearchParams没有实例属性，只有实例方法

URLSearchParams.toString()

返回实例字符串形式

URLSearchParams.append()

追加一个查询参数，接受两个参数，第一个为键名，第二个为键值，没有返回值。

该方法不会识别是否键名已经存在。

```js
var params = new URLSearchParams({'foo': 1 , 'bar': 2});
params.append('foo', 3);
params.toString() // "foo=1&bar=2&foo=3"
```

URLSearchParams.delete()

删除指定的查询参数，接受键名为参数。

URLSearchParams.has()

返回布尔值，表示查询字符串是否包含指定键名。

URLSearchParams.set()

设置查询字符串键值，接受两个参数，第一个为键名，第二个为键值。如已经存在的键，键值会被改写，否则会被追加。

如有多个同名键，set方法会移除现存所有键。

```js
var params = new URLSearchParams('?foo=1&foo=2');
params.set('foo', 3);
params.toString() // "foo=3"
```

URLSearchParams.get()、URLSearchParams.getAll()

get()读取查询字符串里指定键，接受键名为参数。返回的是字符串，如果指定键名不存在则返回null。

如有多个同名键，get返回位置最前面那个键值：

```js
var params = new URLSearchParams('?foo=3&foo=2&foo=1');
params.get('foo') // "3"
```

getAll()返回数组，成员是指定键的所有键值，接受键名为参数。

URLSearchParams.sort()

对查询字符串里的键排序，按照Unicode码点从小到大排列。

URLSearchParams.keys()、URLSearchParams.values()、URLSearchParams.entries()

返回一个遍历器对象，供for...of循环遍历。keys方法返回键名遍历器，values方法返回键值遍历器，entries方法返回键值对遍历器。

🔺如对URLSearchParams进行遍历，内部调用是entries接口。

```js
var params = new URLSearchParams('a=1&b=2');
for (var p of params.values()) {
    console.log(p);// 1 2
}
for (var p of params.entries()) {
    console.log(p);// ["a","1"] ["b","2"]
}
```

**18.11 ArrayBuffer对象，Blob对象**

- ArrayBuffer对象

表示一段二进制数据，模拟内存里的数据。 通过这个对象，JavaScript可读写二进制数据。这个对象可看作内存数据的表达。这个对象是ES6才写入标准的。

浏览器原生提供ArrayBuffer()构造函数，用来生成实例。接受一个整数作参数，表示这段二进制数据占用多少个字节。

ArrayBuffer对象有实例属性byteLength，表示当前实例占用的内存长度（单位字节）

ArrayBuffer对象有实例方法slice()，用来复制一部分内存。接受两个整数参数，分别表示复制开始位置（从0开始）和结束位置（不包含），如省略第二个参数，表示一直复制到结束。

```js
var buffer = new ArrayBuffer(8);
console.log(buffer.byteLength);//8
var buffer2 = buffer.slice(0,4);//复制
console.log(buffer2.byteLength);//4
```

- Blob对象

表示一个二进制文件数据内容，如一个图片文件的内容可通过Blob对象读写。常用来读写文件，名字是Binary Large Object（二进制大型对象）缩写。与ArrayBuffer区别在于，它用于操作二进制文件，而ArrayBuffer用于操作内存。

浏览器原生提供Blob()构造函数：

```js
new Blob(array [, options])
```

第一个参数是数组，成员是字符串或二进制对象，表示新生成的Blob实例对象内容。第二个参数可选，配置对象，目前只有一个属性type，值是字符串，表示数据MIME类型，默认空字符串。

```js
var htmlFragment = ['<a id="a"><b id="b">hey!</b></a>'];
var myBlob = new Blob(htmlFragment, {type : 'text/html'});
var obj = { hello: 'world' };
var blob = new Blob([ JSON.stringify(obj) ], {type : 'application/json'});
```

①实例属性和实例方法

两个实例属性size和type，分别返回数据大小和类型。

有一个实例方法slice()，拷贝原来的数据。有三个参数，都是可选。依次是起始的字节位置（默认0）、结束的字节位置（不包含）、新实例数据类型（默认空字符串）。

```js
var htmlFragment = ['<a id="a"><b id="b">hey!</b></a>'];
var myBlob = new Blob(htmlFragment, {type : 'text/html'});
console.log(myBlob.size,myBlob.type);//32 "text/html"
```

②获取文件信息

文件选择器 `<input type="file">`用于让用户选取文件。浏览器不允许脚本自行设置这个控件的value属性，即文件必须是用户手动选取的，不能是脚本指定的。一旦用户选好了文件，脚本就可以读取这个文件。

文件选择器返回一个FileList对象，该对象是一个类似数组的成员，每个成员都是一个File实例对象。File实例对象是一个特殊Blob实例，增加了name和lastModifiedDate属性。

除了文件选择器，拖放API的dataTransfer.files返回也是FileList对象，成员因此也是File实例对象。

③下载文件

AJAX请求时，如指定responseType属性为blob，下载下来就是一个Blob对象。

④生成URL

浏览器允许使用URL.createObjectURL()，针对Blob对象生成一个临时URL，以便某些API使用。这个URL以blob://开头，表明对应一个Blob对象，协议头后面是一个识别符，用来唯一对应内存里的Blob对象。这一点与data://URL(URL包含实际数据)和file://URL(本地文件系统里文件)都不一样。

浏览器处理Blob URL和普通URL一样，如果Blob对象不存在，返回404状态码；如果跨域请求，返回403状态码。Blob URL只对GET请求有效，如请求成功，返回200状态码。由于Blob URL就是普通URL，因此可以下载。

⑤读取文件

取得Blob对象后，可通过FileReader对象，读取Blob对象内容，即文件内容。

FileReader对象提供四个方法，处理Blob对象。Blob对象作参数传入这些方法，然后以指定格式返回。

FileReader.readAsText()：返回文本，需要指定文本编码，默认UTF-8

FileReader.readAsArrayBuffer()：返回ArrayBuffer对象

FileReader.readAsDataURL()：返回Data URL

FileReader.readAsBinaryString()：返回原始二进制字符串

**18.12 File对象，FileList对象，FileReader对象**

- File对象

代表一个文件，读写文件信息。继承了Blob对象，或说是一种特殊的Blob对象，所有可使用Blob对象的场合都可以使用它。

最常见场景是表单的上传控件 `<input type="file">`。用户选中文件后，浏览器生成一个数组，里面是每一个用户选中的文件。

```js
// <input type="file" onchange="readFile(this.files)">
function readFile(f) {
    console.log(f instanceof FileList);//true
    console.log(f[0] instanceof File);//true
    console.log(f[0].name);//'a.txt'
    var reader = new FileReader();
    reader.readAsText(f[0]);
    reader.onload = function (e) {
        console.log(e.target.result);//123123
    }
}
```

①构造函数

浏览器原生提供一个File()构造函数：

```js
new File(array, name [, options])
```

第一个参数array是一个数组，成员是二进制对象或字符串，表示文件内容。

第二个参数是字符串，表示文件名或文件路径。

第三个参数options是配置对象，设置实例属性。可选。可设置两个属性：type为字符串，表示实例对象MIME类型，默认值为空字符串。lastModified为时间戳，表示上次修改的时间，默认Date.now()。

```js
var file = new File(
  ['foo'],
  'foo.txt',
  {
    type: 'text/plain',
  }
);
```

②实例属性和实例方法

File.lastModified：最后修改时间

File.name：文件名或文件路径

File.size：文件大小（单位字节）

File.type：文件MIME类型

File对象没有自己实例方法，由于继承了Blob对象，可使用Blob的实例方法slice()

- FileList对象

类似数组的对象，代表一组选中的文件，每个成员都是一个File实例。主要出现在两个场景：

文件控件节点（`<input type="file">`）的files属性。拖拉一组文件时，目标区的DataTransfer.files属性。

FileList的实例属性主要是length，表示包含多少个文件。

FileList实例方法主要是item()，返回指定位置实例。接受一个整数作参数，表示位置序号。也可直接用方括号运算符获取实例。

- FileReader对象

读取File对象或Blob对象所包含的文件内容

浏览器原生提供一个FileReader构造函数：

```js
var reader = new FileReader();
```

FileReader有以下实例属性：

FileReader.error：读取文件时产生的错误对象

FileReader.readyState：整数，读取文件时的当前状态。0表示尚未加载任何数据，1表示数据正在加载，2表示加载完成。

FileReader.result：读取完成后的文件内容，可能是字符串，可能是一个ArrayBuffer实例。

FileReader.onabort：abort事件（用户终止读取操作）的监听函数

FileReader.onerror：error事件（读取错误）的监听函数

FileReader.onload：load事件（读取操作完成）的监听函数，通常在这个函数里使用result属性，拿到文件内容

FileReader.onloadstart：loadstart事件（读取操作开始）的监听函数

FileReader.onloadend：loadend事件（读取操作结束）的监听函数

FileReader.onprogress：progress事件（读取操作进行中）的监听函数

FileReader有以下实例方法：

FileReader.abort()：终止读取操作，readyState属性变为2

FileReader.readAsArrayBuffer()：以ArrayBuffer的格式读取文件，读取完成后，result属性返回ArrayBuffer实例

FileReader.readAsBinaryString()：读取完成后，result属性返回原始二进制字符串。

FileReader.readAsDataURL()：读取完成后，result属性返回一个Data URL格式（Base64编码）字符串，代表文件内容。对于图片文件，这个字符串可用于`<img>`元素的src属性。这个字符串不能直接进行Base64解码，必须把前缀`data:*/*;base64,`从字符串里删除后，再解码。

FileReader.readAsText()：读取完成后，result属性返回文件内容的文本字符串。第一个参数是代表文件Blob实例，第二个参数可选，表示文本编码，默认UTF-8。

```js
var file = new File(['1,2,3'],'b.txt',{type:'text/plain'});
console.log(file.name,file.size,file.type);//'b.txt' 5 'text/plain'
var fileReader = new FileReader();
fileReader.readAsText(file);
fileReader.onload = function (e) {
    console.log(e.target.result);//'1,2,3'
}
```

**18.13 表单，FormData对象**

- 表单概述

表单是用来收集用户提交的数据，发送到服务器。

用户点击“提交”按钮，每一个控件都会生成一个键值对，键名是控件的name属性，键值是控件的value属性，键名和键值之间由等号连接。

所有的键值对都会提交到服务器。提交的数据格式跟`<form>`元素method属性有关。如是GET方法，所有键值对会以URL的查询字符串形式提交到服务器。如是POST方法，所有键值对会连接成一行，作为HTTP请求的数据体发送到服务器。

表单里的`<button>`元素如没有用type属性指定类型，默认是submit控件。除了点击submit控件提交表单，还可用表单元素的submit()方法，通过脚本提交表单。表单元素的reset()方法重置所有控件的值。

```html
<form action="/handling-page" method="post">
  <div>
    <label for="name">用户名：</label>
    <input type="text" id="name" name="user_name" />
  </div>
  <div>
    <label for="passwd">密码：</label>
    <input type="password" id="passwd" name="user_passwd" />
  </div>
  <div>
    <input type="submit" id="submit" name="submit_button" value="提交" />
  </div>
</form>
```

- FormData对象

FormData()是一个构造函数：

```js
var formdata = new FormData(form);
```

参数是一个DOM表单元素，构造函数会自动处理表单键值对。参数可选，如果省略，则为空表单。

①实例方法

FormData.get(key)：获取指定键名对应键值，如有多个同名键值对，返回第一个键值对的键值。

FormData.getAll(key)：返回数组，指定键名对应的所有键值。

FormData.set(key,value)：设置指定键名的键值。如果键名不存在，会添加这个键值对，否则更新指定键名的键值。如果第二个参数是文件，还可用第三个参数表示文件名。

FormData.delete(key)：删除一个键值对

FormData.append(key,value)：添加一个键值对。如果键名重复，会生成两个相同键名的键值对。如果第二个参数是文件，还可用第三个参数，表示文件名。

FormData.has(key)：返回布尔值，是否具有该键名的键值对

FormData.keys()：返回遍历器对象，用于for...of循环遍历所有键名。

FormData.values()：返回遍历器对象，用于for...of循环遍历所有键值。

FormData.entries()：返回遍历器对象，用于for...of循环遍历所有键值对。

```js
//<form id="myForm" name="myForm">
//    <div>
//        <label for="username">用户名：</label>
//        <input type="text" id="username" name="username">
//    </div>
//    <div>
//        <label for="useracc">账号：</label>
//        <input type="text" id="useracc" name="useracc">
//    </div>
//    <input type="submit" value="Submit!">
//</form>

var form = document.getElementById('myForm');
var formData = new FormData(form);
formData.set('username','aaa');
console.log(formData.get('username'));//'aaa'
formData.set('useracc',1234);
console.log(formData.has('useracc'));//true
for ( var value of formData.values()) {
    console.log(value);//'aaa' 1234
}
```

- 表单的内置验证

①自动校验

如果一个控件通过验证，会匹配:valid的CSS伪类，浏览器继续进行表单提交流程。如没有通过验证，该控件会匹配:invalid的CSS伪类，浏览器终止表单提交，显示一个错误信息。

②checkValidity()

手动触发校验。返回布尔值，true表示通过校验，false表示没通过校验。

③willValidate属性

布尔值，表示该控件是否会在提交时进行校验。

④validationMessage属性

返回字符串，表示控件不满足校验条件时，浏览器显示的提示文本。当该控件不会在提交时自动校验或该控件满足校验条件时，该属性返回空字符串。

⑤setCustomValidity()

定制校验失败时报错信息。接受字符串作参数，字符串就是定制的报错信息。如参数为空字符串，则上次设置的报错信息被清除。

该方法可替换浏览器内置的表单验证报错信息。

可直接调用，也可再invalid事件监听函数里调用。

⑥validity属性

返回一个ValidityState对象，包含当前校验状态信息。所有属性都只读：

ValidityState.badInput：布尔值，浏览器是否不能将用户的输入转换成正确类型。

ValidityState.customError：布尔值，表示是否已经调用setCustomValidity()方法，将校验信息设为一个非空字符串。

ValidityState.patternMismatch：布尔值，用户输入的值是否不满足模式要求

ValidityState.rangeOverflow：布尔值，用户输入的值是否大于最大范围

ValidityState.rangeUnderflow：布尔值，用户输入的值是否小于最小范围

ValidityState.stepMismatch：布尔值，用户输入的值不符合步长设置（不能被步长值整除）

ValidityState.tooLong：布尔值，用户输入字数超出最长字数

ValidityState.tooShort：布尔值，用户输入字数少于最短字数

ValidityState.typeMismatch：布尔值，用户填入值不符合类型要求（主要是类型为Email或URL情况）

ValidityState.valid：布尔值，用户是否满足所有校验条件

ValidityState.valueMissing：布尔值，用户没有填入必填值

🔺如果想禁止浏览器弹出表单验证报错信息，可监听invalid事件。

⑦表单的novalidate属性

关闭浏览器自动校验。也可在脚本里面设置。如果表单元素没设置novalidate属性，提交按钮的formnovalidate属性也有同样作用

```js
//<form id="myForm" name="myForm">
//    <input id="username" required minlength="2" maxlength="5" type="text">
//    <input type="submit" value="Submit!">
//</form>

var input = document.getElementById('username');
input.addEventListener('invalid',function (e) {
    console.log(e.target.validity.tooShort);//当输入一个字符时，返回true
},false);
```

- enctype属性

表单可用四种编码，向服务器发送数据。编码格式由表单enctype属性决定。

假定表单有两个字段，分别是foo和baz，其中foo字段值等于bar，baz字段值是一个分为两行字符串。

①GET方法

enctype属性无效。数据以URL查询字符串发出

②application/x-www-form-urlencoded

如表单用POST方法发送数据，并省略enctype属性，数据以application/x-www-form-urlencoded格式发送（默认值）

③text/plain

如表单用POST方法发送数据，enctype属性为text/plain，数据以纯文本格式发送。

④multipart/form-data

如表单用POST方法，enctype属性为multipart/form-data，数据以混合格式发送。这种格式也是文件上传格式。

- 文件上传

通过文件输入框选择本地文件。

将表单元素method属性设为POST，enctype属性设为multipart/form-data。enctype属性决定了HTTP头信息的Content-Type字段值，默认是application/x-www-form-urlencoded。

file控件的multiple属性，指定可一次选择多个文件；如没有这个属性，一次只能选择一个文件。

可以发送FormData实例，也可直接AJAX发送文件。

**18.14 IndexedDB API**

IndexedDB是浏览器提供的本地数据库，可被网页脚本创建和操作。允许储存大量数据，提供查找接口，还能建立索引。

特点：

①键值对储存：采用对象仓库存放数据，每一个数据记录都有对应主键，主键独一无二。

②异步：不会锁死浏览器，用户依然可进行其他操作，防止大量数据读写拖慢网页表现。

③支持事务：一系列操作步骤中，只要有一步失败，整个事务就都取消，数据库回滚到事务发生前状态，不存在只改写一部分数据情况。

④同源限制：每一个数据库对应创建它的域名。网页只能方问自身域名下数据库而不能访问跨域数据库。

⑤储存空间大：一般来说不少于250MB，甚至无上限

⑥支持二进制储存：还可储存二进制数据，如ArrayBuffer对象和Blob对象

- 基本概念

数据库：IDBDatabase对象

一系列相关数据的容器。每个域名（协议+域名+端口）都可新建任意多个数据库。

IndexedDB数据库有版本的概念。同一个时刻，只能有一个版本的数据库存在。如修改数据库结构，只能通过升级数据库版本完成。

对象仓库：IDBObjectStore对象

每个数据库包含若干对象仓库。类似于关系型数据库表格。

数据记录：

对象仓库保存的是数据集了。每条记录类似于关系型数据库的行，但只有主键和数据体两部分。主键用来建立默认索引，必须不同。主键可是数据记录里一个属性，也可指定为一个递增整数编号。数据体可是任意数据类型，不限于对象。

```js
{ id: 1, text: 'foo' }
```

索引：IDBIndex对象

可在对象仓库里为不同属性建立索引，加速数据检索

事务：IDBTransaction对象

数据记录的读写和删改，都要通过事务完成。事务对象提供error、abort和complete三个事件，监听操作结果。

操作请求：IDBRequest对象

指针：IDBCursor对象

主键集合：IDBKeyRange对象

- 操作流程

①打开数据库

使用indexedDB.open()。接受两个参数，第一个参数是字符串，表示数据库名，如指定数据库不存在则新建数据库。第二个参数是整数，表示数据库版本。如省略，打开已有数据库时，默认当前版本；新建数据库时，默认为1。

该方法返回一个IDBRequest对象。该对象通过三种事件error、success、upgradeneeded，处理打开数据库操作结果：

error事件表示打开数据库失败

success事件表示成功打开数据库，这时通过result属性拿到数据库对象

如果指定版本号，大于数据库实际版本号，就会发生数据库升级事件upgradeneeded。通过事件对象target.result属性拿到数据库实例。

②新建数据库

与打开数据库是同一操作。如指定数据库不存在，就会新建。不同在于后续操作主要在upgradeneeded事件监听函数里完成，因为这时版本从无到有。

新建数据库后，第一件事是新建对象仓库（新建表）。新建对象仓库后，下一步新建索引。

```js
request.onupgradeneeded = function (event) {
  db = event.target.result;
  var objectStore;
  if (!db.objectStoreNames.contains('person')) {
    objectStore = db.createObjectStore('person', { keyPath: 'id' });
  }
}
```

```js
request.onupgradeneeded = function(event) {
  db = event.target.result;
  var objectStore = db.createObjectStore('person', { keyPath: 'id' });
  objectStore.createIndex('name', 'name', { unique: false });
  objectStore.createIndex('email', 'email', { unique: true });
}
```

③新增数据

向对象仓库写入数据记录，通过事务完成。

```js
function add() {
  var request = db.transaction(['person'], 'readwrite')
    .objectStore('person')
    .add({ id: 1, name: '张三', age: 24, email: 'zhangsan@example.com' });

  request.onsuccess = function (event) {
    console.log('数据写入成功');
  };

  request.onerror = function (event) {
    console.log('数据写入失败');
  }
}

add();
```

④读取数据

通过事务完成

```js
function read() {
   var transaction = db.transaction(['person']);
   var objectStore = transaction.objectStore('person');
   var request = objectStore.get(1);

   request.onerror = function(event) {
     console.log('事务失败');
   };

   request.onsuccess = function( event) {
      if (request.result) {
        console.log('Name: ' + request.result.name);
        console.log('Age: ' + request.result.age);
        console.log('Email: ' + request.result.email);
      } else {
        console.log('未获得数据记录');
      }
   };
}

read();
```

⑤遍历数据

使用指针对象IDBCursor

```js
function readAll() {
  var objectStore = db.transaction('person').objectStore('person');

   objectStore.openCursor().onsuccess = function (event) {
     var cursor = event.target.result;

     if (cursor) {
       console.log('Id: ' + cursor.key);
       console.log('Name: ' + cursor.value.name);
       console.log('Age: ' + cursor.value.age);
       console.log('Email: ' + cursor.value.email);
       cursor.continue();
    } else {
      console.log('没有更多数据了！');
    }
  };
}

readAll();
```

⑥更新数据

IDBObject.put()

```js
function update() {
  var request = db.transaction(['person'], 'readwrite')
    .objectStore('person')
    .put({ id: 1, name: '李四', age: 35, email: 'lisi@example.com' });

  request.onsuccess = function (event) {
    console.log('数据更新成功');
  };

  request.onerror = function (event) {
    console.log('数据更新失败');
  }
}

update();
```

⑦删除数据

IDBObjectStore.delete()

```js
function remove() {
  var request = db.transaction(['person'], 'readwrite')
    .objectStore('person')
    .delete(1);

  request.onsuccess = function (event) {
    console.log('数据删除成功');
  };
}

remove();
```

⑧索引

从任意字段拿到数据记录。如不建立索引，默认只能从主键取值。

```js
objectStore.createIndex('name', 'name', { unique: false });

var transaction = db.transaction(['person'], 'readonly');
var store = transaction.objectStore('person');
var index = store.index('name');
var request = index.get('李四');

request.onsuccess = function (e) {
  var result = e.target.result;
  if (result) {
    // ...
  } else {
    // ...
  }
}
```

- indexedDB对象

①indexedDB.open()

异步，但立刻返回一个IDBOpenDBRequest对象

```js
var openRequest = window.indexedDB.open('test', 1);
```

第一个参数是数据库名，格式为字符串，必须。第二个参数是数据库版本，大于0的正整数，如该参数大于当前版本，触发数据库升级，可选。如数据库存在，打开当前版本数据库；如数据库不存在，创建该版本数据库，默认版本为1。

通过各种事件通知客户端：

success：打开成功

error：打开失败

upgradeneeded：第一次打开该数据库，或数据库版本发生变化

blocked：上一次数据库连接还未关闭

🔺①第一次打开数据库，触发upgradeneeded事件，然后触发success事件。②监听函数定义在IDBOpenDBRequest对象上，success事件发生后，从result属性可拿到已经打开的IndexedDB数据库对象。

②indexedDB.deleteDatabase()

删除一个数据库，参数为数据库名。立刻返回一个IDBOpenDBRequest对象，然后对数据库异步删除。

IDBOpenDBRequest对象可监听以下事件：

success：删除成功

error：删除报错

🔺①调用该方法后，当前数据库其他已经打开连接都会接收到versionchange事件。②删除不存在的数据库不会报错

③indexedDB.cmp()

比较两个值是否为indexedDB的相同的主键。返回整数，表示相同，1表示第一个主键大于第二个主键，-1表示第一个主键小于第二个主键。

🔺不能用于比较任意的JavaScript值

- IDBRequest对象

IDBRequest对象表示打开的数据库连接。数据库的操作都是通过这个对象完成。

所有操作都是异步操作，通过readyState属性判断是否完成，如为pending表示操作正在进行，如为done表示操作完成，可能成功可能失败。

操作完成后，触发success事件或error事件，通过result属性和error属性拿到操作结果。

IDBRequest.readyState：penging表示操作正在进行，done表示操作正在完成

IDBRequest.result：返回请求结果

IDBRequest.error：请求失败时，返回错误对象

IDBRequest.source：返回请求来源（如索引对象或ObjectStore）

IDBRequest.transaction：返回当前请求正在进行的事务，如不包含事务则返回null。

IDBRequest.onsuccess：success事件监听函数

IDBRequest.onerror：error事件监听函数

IDBOpenDBRequest对象继承了IDBRequest对象，提供了两个额外事件监听属性：

IDBOpenDBRequest.onblocked：blocked事件监听函数

IDBOpenDBRequest.onupgradeneeded：upgradeneeded事件监听函数

- IDBDatebase对象

从IDBOpenDBRequest对象的result属性上，拿到一个IDBDatabase对象，表示连接的数据库。后续操作都通过这个对象完成。

①属性

IDBDatabase.name：字符串，数据库名

IDBDatabase.version：整数，数据库版本。数据库第一次创建时，为空字符串

IDBDatabase.objectStoreNames：DOMStringList对象（字符串集合），包含当前数据库所有object store名

IDBDatabase.onabort：abort事件（事务中止）监听函数

IDBDatabase.onclose：close事件（数据库意外关闭）监听函数

IDBDatabase.onerror：error事件（访问数据库失败）监听函数

IDBDatabase.onversionchange：数据库版本变化时触发（发生upgradeneeded事件或调用indexedDB.deleteDatabase()）

🔺可使用DOMStringList对象的contains方法，检查数据库是否包含某个对象仓库

②方法

IDBDatabase.close()：关闭数据库连接，实际会等所有事务完成后关闭

IDBDatabase.createObjectStore()：创建存放数据的对象仓库，返回IDBObjectStore对象。该方法只能在versionchange事件监听函数里使用。

IDBDatabase.deleteObjectStore()：删除指定对象仓库，只能在versionchange事件监听函数中调用。

IDBDatabase.transaction()：返回IDBTransaction事务对象

🔺createObjectStore()可接受第二个对象参数，设置对象仓库属性：

```js
db.createObjectStore('test', { keyPath: 'email' });
db.createObjectStore('test2', { autoIncrement: true });
```

keyPath属性表示主键，默认null；autoIncrement属性表示是否使用自动递增整数为主键，默认false。一般keyPath和autoIncrement属性只要使用一个，如同时使用，表示主键为递增的整数，且对象不得缺少keyPath指定的属性。

🔺向数据库添加数据前，必须先创建数据库事务：

```js
var t = db.transaction(['items'], 'readwrite');
```

第一个参数是数组，表示涉及的对象仓库，通常只有一个。第二个参数是表示操作类型字符串。readonly为只读，readwrite为读写。添加数据用readwrite、读取数据用readonly，默认readonly。

- IDBObjectStore对象

对应一个对象仓库。IDBDatabase对象的transaction()返回一个事务对象，该对象objectStore()返回IDBObjectStore对象：

```js
db.transaction(['test'], 'readonly')
  .objectStore('test')
  .get(X)
  .onsuccess = function (e) {}
```

①属性

IDBObjectStore.indexNames：返回一个类似数组的对象（DOMStringList），包含当前对象仓库所有索引

IDBObjectStore.keyPath：返回当前对象仓库主键

IDBObjectStore.name：返回当前对象仓库名

IDBObjectStore.transaction：返回当前对象仓库所属事务对象

IDBObjectStore.autoIncrement：布尔值，主键是否自动递增

②方法

IDBObjectStore.add()

向对象仓库添加数据，返回IDBRequest对象。接受两个参数，第一个参数是键值，第二个参数是主键，该参数可选，默认null。

IDBObjectStore.put()

更新某个主键对应数据记录，如对应键值不存在，则插入一条新记录。返回IDBRequest对象。接受两个参数，第一个参数为新数据，第二个参数为主键，可选，且只在自动递增时才有必要提供，因为主键不包含在数据值里。

IDBObjectStore.clear()

删除当前对象仓库所有记录。返回IDBRequest对象。不需要参数

IDBObjectStore.delete()

删除指定主键的记录。返回IDBRequest对象，参数为主键的值

IDBObjectStore.count()

计算记录数量。返回IDBRequest对象。不带参数时返回当前对象仓库所有记录数量。如主键或IDBKeyRange对象为参数，返回对应记录数量

IDBObjectStore.getKey()

获取主键。返回IDBRequest对象。参数可是主键值或IDBKeyRange对象

IDBObjectStore.get()

获取主键对应的数据记录。返回IDBRequest对象。

IDBObjectStore.getAll()

获取对象仓库记录。返回IDBRequest对象

```js
// 获取所有记录
objectStore.getAll()

// 获取所有符合指定主键或 IDBKeyRange 的记录
objectStore.getAll(query)

// 指定获取记录的数量
objectStore.getAll(query, count)
```

IDBObjectStore.getAllKeys()

获取所有符合条件主键。返回IDBRequest对象

```js
// 获取所有记录的主键
objectStore.getAllKeys()

// 获取所有符合条件的主键
objectStore.getAllKeys(query)

// 指定获取主键的数量
objectStore.getAllKeys(query, count)
```

IDBObjectStore.index()

返回指定名称的索引对象IDBIndex

```js
var t = db.transaction(['people'], 'readonly');
var store = t.objectStore('people');
var index = store.index('name');

var request = index.get('foo');
```

IDBObjectStore.createIndex()

新建当前数据库一个索引。只能在versionChange监听函数里调用。

```js
objectStore.createIndex(indexName, keyPath, objectParameters)
```

三个参数分别为索引名、主键、配置对象（可选）。第三个参数可配置如下属性：

unique：如设为true，不允许重复的值

multiEntry：如设为true，对有多个值的主键数组，每个值将在索引里新建一个条目，否则主键数组对应一个条目。

IDBObjectStore.deleteIndex()

删除指定索引。只能在versionChange监听函数里调用。

IDBObjectStore.openCursor()

获取一个指针对象。

指针对象可用来遍历数据，该对象也是异步的。有success和error事件的监听函数，监听函数接受一个事件对象为参数，该对象的target.result属性指向当前数据记录。该记录的key和value分别返回主键和键值。continue()方法将光标移到下一个数据对象，如当前数据对象已经是最后一个数据，则光标指向null。

该方法第一个参数是主键值，或IDBKeyRange对象。如指定该参数，将只处理包含指定主键的记录；如省略，将处理所有记录。还可接受第二个参数，表示遍历方向，默认next，其他取值是prev、nextunique、prevunique。后两个值表示如果遇到重复值，会自动跳过。

IDBObjectStore.openKeyCursor()

获取一个主键指针对象。

- IDBTransaction对象

异步操作数据库事务。

事务执行顺序按创建顺序，而不是发出请求的顺序：

```js
var trans1 = db.transaction('foo', 'readwrite');
var trans2 = db.transaction('foo', 'readwrite');
var objectStore2 = trans2.objectStore('foo')
var objectStore1 = trans1.objectStore('foo')
objectStore2.put('2', 'key');
objectStore1.put('1', 'key');// key对应键值为1
```

事务可能失败，只有监听到事务的complete事件，才能保证事务操作成功。

①属性

IDBTransaction.db：当前事务所在数据库对象IDBDatabase

IDBTransaction.error：当前事务的错误

IDBTransaction.mode：事务模式，默认readonly，另一个值是readwrite

IDBTransaction.objectStoreNames：返回类似数组的对象DOMStringList，成员是当前事务涉及的对象仓库名

IDBTransaction.onabort：abort事件（事务中断）监听函数

IDBTransaction.oncomplete：complete事件（事务成功）监听函数

IDBTransaction.onerror：error事件（事务失败）监听函数

②方法

IDBTransaction.abort()：终止当前事务，回滚所有已经进行的变更

IDBTransaction.objectStore(name)：返回指定名称的对象仓库IDBObjectStore

- IDBIndex对象

数据库索引，通过这个对象可获取数据库里记录。数据记录的主键默认带索引，IDBIndex对象主要用于通过除主键以外其他键，建立索引获取对象

IDBIndex是持久性的键值对存储。只要插入、更新或删除数据记录，引用的对象库中的记录，索引就会自动更新。

IDBObjectStore.index()可获取该对象

①属性

IDBIndex.name：字符串，索引名

IDBIndex.objectStore：索引所在的对象仓库

IDBIndex.keyPath：索引的主键

IDBIndex.multiEntry：布尔值，针对keyPath为数组，如设为true，创建数组时，每个数组成员都会有一个条目，否则每个数组都只有一个条目。

IDBIndex.unique：布尔值，创建索引时是否允许相同主键。

②方法

异步，立即返回IDBRequest对象

IDBIndex.count()：获取记录数量。接受主键或IDBKeyRange对象为参数，只返回符合主键的记录数量，否则返回所有记录数量

IDBIndex.get(key)：获取符合指定主键数据记录

IDBIndex.getKey(key)：获取指定主键

IDBIndex.getAll()：获取所有数据记录。可接受两个可选参数，第一个用来指定主键，第二个指定返回记录的数量。如省略这两个参数，返回所有记录。由于获取成功时，浏览器必须生成所有对象，所以对性能有影响。如果数据集比较大，建议使用IDBCursor对象。

IDBIndex.getAllKeys()：与IDBIndex.getAll()相似，区别是获取所有主键

IDBIndex.openCursor()：获取IDBCursor对象，遍历索引里所有条目

IDBIndex.openKeyCursor()：与IDBIndex.openCursor()相似，区别是遍历所有条目主键

- IDBCursor对象

遍历数据仓库或索引的记录。

①属性

IDBCursor.source：返回正在遍历的对象仓库或索引

IDBCursor.direction：字符串，指针遍历方向，可能取值为next：从头向后、nextunique：从头向后，重复值只遍历一次、prev：从后向前、prevunique：从后向前，重复值只遍历一次。该参数通过IDBObjectStore.openCursor()的第二个参数指定，一旦指定不能改变。

IDBCursor.key：返回当前记录主键

IDBCursor.value：返回当前记录数据值

IDBCursor.primaryKey：返回当前记录主键。对于数据仓库，等同于IDBCursor.key，对于索引，IDBCursor.key返回索引的位置值，该属性返回数据记录的主键。

②方法

IDBCursor.advance(n)：指针向前移动n个位置

IDBCursor.continue()：指针向前移动一个位置，可接受一个主键为参数，跳转到这个主键

IDBCursor.continuePrimaryKey()：两个参数，第一个key，第二个primaryKey，将指针移到符合这两个参数的位置

IDBCursor.delete()：删除当前位置记录，返回IDBRequest对象。不会改变指针位置

IDBCursor.update()：更新当前位置记录，返回IDBRequest对象。参数是要写入数据库的新的值。

- IDBKeyRange对象

数据仓库里的一组主键。根据这组主键，可获取数据仓库或索引里一组记录。

IDBKeyRange可只包含一个值，也可指定上限和下限。

有四个静态方法，指定主键范围：

IDBKeyRange.lowerBound()：指定下限，默认包含端点值，可传入布尔值修改这个属性。

IDBKeyRange.upperBound()：指定上限，默认包含端点值，可传入布尔值修改这个属性。

IDBKeyRange.bound()：同时指定上下限，默认包含端点值，可传入布尔值修改这个属性。

IDBKeyRange.only()：指定只包含一个值

```js
// All keys > x &&< y
var r6 = IDBKeyRange.bound(x, y, true, true);

// All keys > x && ≤ y
var r7 = IDBKeyRange.bound(x, y, true, false);

// All keys ≥ x &&< y
var r8 = IDBKeyRange.bound(x, y, false, true);
```

与之对应，有四个只读属性：

IDBKeyRange.lower：返回下限

IDBKeyRange.lowerOpen：布尔值，表示下限是否为开区间

IDBKeyRange.upper：返回上限

IDBKeyRange.upperOpen：布尔值，表示上限是否为开区间

🔺IDBKeyRange实例对象生成后，将它作为参数输入IDBObjectStore或IDBIndex对象的openCursor()方法，就可在所设定范围内读取数据。

IDBKeyRange有一个实例方法includes(key)，返回布尔值，表示某个主键是否包含在当前这个主键组之内。

**18.15 Web Worker**

Web Worker作用：为JavaScript创造多线程环境。允许主线程创建Worker线程，将一些任务分配给后者运行。在主线程运行的同时，Worker线程在后台运行，两者互不干扰。等到Worker线程完成计算任务，再把结果返回给主线程。好处是一些计算密集型或高延迟任务可交由Worker线程执行，主线程（通常负责UI交互）能保持流畅，不会被阻塞或拖慢。

Worker线程一旦新建成功，就会始终运行，不会被主线程上活动（如用户点击按钮、提交表单）打断。有利于随时响应主线程通信。但也造成了Worker比较耗费资源，不应过度使用，而且一旦使用完毕，应该关闭。

注意点：

同源限制：分配给Worker线程运行的脚本文件，必须与主线程脚本文件同源

DOM限制：Worker线程所在的全局对象与主线程不同，无法读取主线程所在网页DOM对象，也无法使用document、window、parent等对象。但Worker线程可使用navigator对象和location对象。

全局对象限制：Worker全局对象WorkerGlobalScope，不同于网页全局对象Window，很多接口拿不到。理论上Worker线程不能使用console.log，但浏览器实际支持console.log。

通信联系：Worker线程和主线程不在同一个上下文环境，不能直接通信，必须通过消息完成。

脚本限制：Worker线程不能执行alert()和confirm()，但可使用XMLHttpRequest对象发出AJAX请求

文件限制：Worker线程无法读取本地文件，即不能打开本机文件系统，它所加载的脚本必须来自网络。

- 基本用法

①主线程

主线程采用new命令，调用Worker()构造函数，新建一个Worker线程：

```js
var worker = new Worker('work.js');
```

参数是一个来自网络的脚本文件。

然后，主线程调用worker.postMessage()，向Worker线程发消息。该方法参数是主线程传给Worker的数据，可是各种数据类型，包括二进制数据。

接着，主线程通过worker.onmessage指定监听函数，接收子线程发回的消息。事件对象的data属性可获取Worker发来的数据。

Worker完成任务后，主线程可通过worker.terminate()把它关掉。

②Worker线程

Worker线程内需要有一个监听函数，监听message事件。监听函数参数是事件对象，事件对象的data属性包含主线程发来的数据。self.postMessage()向主线程发送消息。self.close()用于在Worker内部关闭自身。

③Worker加载脚本

Worker内部如要加载其他脚本，可用importScripts()方法，该方法可同时加载多个脚本。

④错误处理

主线程可监听Worker是否发生错误。如发生错误，Worker会触发主线程error事件。Worker内部也可监听error事件。

⑤关闭Worker

使用完毕，为节省系统资源，必须关闭Worker。

主线程：worker.terminate()、Worker线程：self.close()

- 数据通信

这种通信是传值而不是传址，Worker对通信内容的修改，不会影响到主线程。

浏览器内部，先将通信内容串行化，然后把串行化后的字符串发给Worker，后者再将它还原。

🔺拷贝方式发送二进制数据，会造成性能问题。JavaScript允许主线程把二进制数据直接转移给子线程，但一旦转移，主线程无法再使用这些二进制数据，这是为了防止出现多个线程同时修改数据的麻烦局面。这种转移数据方法，叫做Transferable Objects。这使得主线程可快速把数据交给Worker，对于影像处理、声音处理、3D运算等很方便：

```js
// Transferable Objects 格式
worker.postMessage(arrayBuffer, [arrayBuffer]);

// 例子
var ab = new ArrayBuffer(1);
worker.postMessage(ab, [ab]);
```

- 同页面的Web Worker

```html
<!DOCTYPE html>
  <body>
    <script id="worker" type="app/worker">
      addEventListener('message', function () {
        postMessage('some message');
      }, false);
    </script>
  </body>
</html>
```

```js
var blob = new Blob([document.querySelector('#worker').textContent]);
var url = window.URL.createObjectURL(blob);
var worker = new Worker(url);

worker.onmessage = function (e) {
  // e.data === 'some message'
};
```

主线程和Worker的代码都在同一个网页上

- 实例：Worker线程完成轮询

Worker每秒轮询一次数据，然后跟缓存比较。如不一致，说明服务器端有了新变化，需要通知主线程。

- 实例：Worker新建Worker

Worker线程内新建多个Worker线程，依次向多个Worker发送消息。

- API

①主线程

```js
var myWorker = new Worker(jsUrl, options);
```

第一个参数是脚本网址，必须。第二个参数是配置对象，可选，一个作用是指定Worker名称，区分多个Worker线程。

```js
// 主线程
var myWorker = new Worker('worker.js', { name : 'myWorker' });

// Worker 线程
self.name // myWorker
```

Worker()构造函数返回一个Worker线程对象，供主线程操作Worker。

Worker线程对象属性和方法：

Worker.onerror：指定error事件监听函数

Worker.onmessage：指定message事件监听函数，发送过来的数据再Event.data属性

Worker.onmessageerror：指定messageerror事件监听函数。发送的数据无法序列化成字符串时，触发这个事件。

Worker.postMessage()：向Worker线程发送消息

Worker.terminate()：立即终止Worker线程

②Worker线程

self.name：Worker名字，只读，由构造函数指定

self.onmessage：指定message事件监听函数

self.onmessageerror：指定messageerror事件监听函数。

self.close()：关闭Worker线程

self.postMessage()：向产生这个Worker的线程发送消息

self.importScripts()：加载JS脚本

