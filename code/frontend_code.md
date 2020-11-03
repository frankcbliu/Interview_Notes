# 前端手写代码合集

## 一、用ES5实现数组的map方法

**核心要点**

1.回调函数的参数有哪些，返回值如何处理。

2.不修改原来的数组。

```js
/*
var new_array = arr.map(function callback(currentValue[, index[, array]]) {
 // Return element for new_array 
}[, thisArg])
*/

Array.prototype.myMap = function (fn, context) {
	let arr = Array.prototype.slice.call(this);
  let res = [];
  for (let i = 0; i < arr.length; i++) {
    res.push(fn.call(context, arr[i], i, arr));
  }
  return res;
}
```

## 二、用ES5实现数组的reduce方法(累加器)

**核心要点**

1、初始值不传怎么处理

2、回调函数的参数有哪些，返回值如何处理。

```js
// array.reduce(function(total, currentValue, currentIndex, arr), initialValue)
Array.prototype.myReduce = function(fn, initialValue) {
  let arr = Array.prototype.slice.call(this);
  let total = initialValue ? initialValue : arr[0];
  let startIndex = initialValue ? 0 : 1;
  for (let i = startIndex; i < arr.length; i++) {
    total = fn.call(null, total, arr[i], i, this);
  }
  return total;
}
```



## 【引申】用ES5实现数组的reduceRight方法

```js
Array.prototype.myReduceRight = function(fn, initialValue) {
	let arr = Array.prototype.slice.call(this);
  let len = arr.length;
  let total = initialValue ? initialValue : arr[len-1];
  let startIndex = initialValue ? len-1 : len-2;
  for (let i = startIndex; i >= 0; i--) {
    total = fn.call(null, total, arr[i], i, this);
  }
  return total;
}
```





## 三、实现call/apply

思路: 利用this的上下文特性。

```js
// function.call(thisArg, arg1, arg2, ...)
Function.prototype.myCall = function(context, ...args) {
  let func = this;
  let fn = Symbol('fn');
  context[fn] = func;
  let res = context[fn](...args);
  delete context[fn];
  return res;
}

Function.prototype.myApply = function(context, args) {
  let func = this;
  let fn = Symbol('fn');
  context[fn] = func;
  let res = context[fn](...args);
  delete context[fn];
  return res;
}
```

## 四、实现Object.create方法(常用)

**`Object.create()`**方法创建一个新对象，使用现有的对象来提供新创建的对象的`__proto__`。

```js
// Object.create(proto[, propertiesObject])
Object.myCreate = function(proto) {
  function F(){}
  F.prototype = proto;
  // F.prototype.constructor = F;
  return new F();
}
```

## 五、实现bind方法

**核心要点**

1.对于普通函数，绑定this指向

2.对于构造函数，要保证原函数的原型对象上的属性不能丢失

当 bind 返回的函数作为构造函数的时候，bind 时指定的 this 值会失效，但传入的参数依然生效。

```js
// function.bind(thisArg[, arg1[, arg2[, ...]]])
Function.prototype.myBind = function (context, ...args) {
  let self = this;
  let fBound = function () {
    // this instanceof fBound：当作为构造函数时，this为实例，为true，指向this；当为普通函数，this为window，指向context
    return self.apply(this instanceof fBound ? this : context, args.concat(Array.prototype.slice.call(arguments)));
    // apply 如果第一个参数context是null或undifined，默认为window（严格模式下默认 context 是 undefined） 所以这里无需加判断
    // return self.apply(this instanceof fBound ? this : context || window, args.concat(Array.prototype.slice.call(arguments)));
  }
  fBound.prototype = Object.create(this.prototype);
  return fBound;
}
```

## 六、实现new关键字

**核心要点**

1. 创建一个全新的对象，这个对象的__proto__要指向构造函数的原型对象
2. 执行构造函数
3. 返回值为object类型则作为new方法的返回值返回，否则返回上述全新对象

```js
function myNew (fn, ...args) { // fn 构造函数
  let instance = Object.create(fn.prototype);
  let res = fn.apply(instance, args);
  return (res && (typeof res == "object" || typeof res == "function")) ? res : instance;
}
```

## 七、实现instanceof的作用

**`instanceof`** **运算符**用于检测构造函数的 `prototype` 属性是否出现在某个实例对象的原型链上。

核心要点：原型链的向上查找。

```js
function myInstanceof (left, right) { // left:实例对象 right：构造函数
  let proto = Object.getPrototypeOf(left)
  while (proto) {
    if (proto === right.prototype) return true;
    proto = Object.getPrototypeOf(proto);
  }
  return false;
}
```

## 八、实现单例模式

核心要点: 用闭包和Proxy属性拦截

```js
function getSingle (func) {
  let instance = null;
  let handler = {
    construct: function (target, args) {
     	if (!instance) {
        instance = new func(...args);
        // instance = Reflect.construct(func, args);
      }
      return instance;
    }
  }
  return new Proxy(func, handler);
}
```



> `new Proxy(target, handler)`
>
> `target`：要使用 `Proxy` 包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）。
> `handler`：一个容纳一批特定属性的占位符对象。它包含有 Proxy 的各个捕获器（trap）。
>
> `handler` 对象的方法14种（所有的陷阱是可选的。如果没有定义某个陷阱，那么就会保留源对象的默认行为)
>
> - `handler.getPrototypeOf()`: `Object.getPrototypeOf` 方法的陷阱。
> - `handler.setPrototypeOf()`: `Object.setPrototypeOf` 方法的陷阱。
> - `handler.isExtensible()`: `Object.isExtensible` 方法的陷阱。
> - `handler.preventExtensions()`: `Object.preventExtensions` 方法的陷阱。
> - `handler.getOwnPropertyDescriptor()`: `Object.getOwnPropertyDescriptor` 方法的陷阱。
> - `handler.defineProperty()`: `Object.defineProperty` 方法的陷阱。
> - `handler.has()`: `in` 操作符的陷阱。
> - **`handler.get()`: 属性读取操作的陷阱。**
> - **`handler.set()`: 属性设置操作的陷阱。**
> - `handler.deleteProperty()`: `delete` 操作符的陷阱。
> - `handler.ownKeys()`: `Object.getOwnPropertyNames` 方法
> - `Object.getOwnPropertySymbols` 方法的陷阱。
> - `handler.apply()`: 函数调用操作的陷阱。
> - **`handler.construct()`: `new` 操作符的陷阱。**
>
> `Object.defineProperty(obj, prop, descriptor)`
>
> `obj`: 要定义属性的对象。
> `prop`: 要定义或修改的属性的名称或 `Symbol` 。
> `descriptor`: 要定义或修改的属性描述符。
>
> **属性描述符**（一个描述符只能是这两者其中之一；不能同时是两者。）
>
> - **数据描述符**：具有值的属性，该值可以是可写的，也可以是不可写的。
> - **存取描述符**：由 `getter` 函数和 `setter` 函数所描述的属性。
>
> **描述符键值**
>
> - 共享
>   - `configurable`: 是否可修改（包括删除）。当且仅当该属性的 `configurable` 键值为 `true` 时，该属性的描述符才能够被改变，同时该属性也能从对应的对象上被删除。默认为 `false`。
>   - `enumerable`：是否可枚举。当且仅当该属性的 `enumerable` 键值为 `true` 时，该属性才会出现在对象的枚举属性中。默认为 `false`。
>
> - **数据描述符**
>   - `value`: 该属性对应的值。可以是任何有效的 `JavaScript` 值（数值，对象，函数等）。默认为 `undefined`。
>   - `writable`: 当且仅当该属性的 `writable` 键值为 `true` 时，属性的值，也就是上面的 `value`，才能被赋值运算符改变。默认为 `false`。
>
> - **存取描述符**
>   - `get`: 属性的 `getter` 函数，如果没有 `getter`，则为 `undefined`。当访问该属性时，会调用此函数。执行时不传入任何参数，但是会传入 `this` 对象（由于继承关系，这里的`this`并不一定是定义该属性的对象）。该函数的返回值会被用作属性的值。默认为 `undefined`。
>   - `set`: 属性的 `setter` 函数，如果没有 `setter`，则为 `undefined`。当属性值被修改时，会调用此函数。该方法接受一个参数（也就是被赋予的新值），会传入赋值时的 `this` 对象。默认为 `undefined`。



## 九、实现数组的flat

```js
var arr = [1,[2,[3,{a:4}],5,6]]
// 实现：flatten(arr) // [1, 2, 3, {a:4}, 5, 6]

// 方法一：直接调用flat
function flatten(arr) {
  return arr.flat(Infinity);
}

// 方法二：利用JSON 和 正则
function flatten(arr) {
  var str = JSON.stringify(arr);
  str = str.replace(/(\[|\])/g, ''); // 替换掉[]
  str = '[' + str + ']';
  return JSON.parse(str);
}

// 方法三：递归处理
function flatten(arr) {
  let res = [];
  for (let item of arr) {
    if (Array.isArray(item)) {
      res = res.concat(flatten(item));
    }
    else res.push(item);
  }
  return res;
}

// 方法四：利用 reduce
function flatten(arr) {
  return arr.reduce((total, value) => {
    return total.concat(Array.isArray(value) ? flatten(value) : value);
  }, [])
}

// 方法五：扩展运算符
function flatten(arr) {
  while (arr.some(Array.isArray)) { 
    arr = [].concat(...arr);
  }
  return arr;
}
```

> `arr.some(callback(element[, index[, array]])[, thisArg])`
>
> 判断测试数组中是不是至少有1个元素通过了被提供的函数测试。它返回的是一个Boolean类型的值。
>
> `callback`：用来测试每个元素的函数，接受三个参数：
>
> - `element`：数组中正在处理的元素。
> - `index`：可选，数组中正在处理的元素的索引值。
> - `array`：可选，`some()`被调用的数组。
> - `thisArg`：可选，执行 `callback` 时使用的 `this` 值。
>
> ```js
> Array.prototype.mySome(fn, context) {
>   let arr = Array.prototype.slice.call(this);
>   for (let i of arr) {
>     if (fn.call(context, i)) return true;
>   }
>   return false;
> }
> ```

## 【拓展】实现数组去重

```js
var arr = [1,2,3,1,1,4]
function fun(arr) {
  return Array.from(new Set(arr)) // Array.from 把set转为数组
}
```



> 已知如下数组：var arr = [ [1, 2, 2], [3, 4, 5, 5], [6, 7, 8, 9, [11, 12, [12, 13, [14] ] ] ], 10];
>
> 编写一个程序将数组扁平化去并除其中重复部分数据，最终得到一个升序且不重复的数组
>
> ```js
> function fun(arr) {
> let flatArr = arr.flat(Infinity); // 扁平化
> let disArr = Array.from(new Set(flatArr)); // 去重
> return disArr.sort((a,b)=>{a-b});
> }
> ```





## 十、实现防抖功能

**核心要点**

如果在定时器的时间范围内再次触发，则重新计时。

```js
function debounce(fn, delay) {
  let timer = null;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args)
    }, delay);
  }
}
```



## 十一、实现节流功能

**核心要点**

如果在定时器的时间范围内再次触发，则不予rf理睬，等当前定时器完成，才能启动下一个定时器。

```js
function throttle(fn, delay) {
  let flag = true;
  return (...args) => {
    if (!flag) return;
    flag = false;
    setTimeout(() => {
      fn.apply(this, args);
      flag = true;
    }, delay);
  }
}
```



## 十二、实现 generator 异步自动执行器

```js
function takeLongTime(n) {
    console.log(n)
    return new Promise(resolve => {
        setTimeout(() => resolve(n + 200), n);
    });
}

var doIt = function *() {
    const time1 = 300;
    const time2 = yield takeLongTime(time1); // 返回一个promise
    const result = yield takeLongTime(time2);
    console.log(`result is ${result}`);
}

// 自动执行器
function run(gen) {
  let g = gen();
  function _next(data) {
    let temp = g.next(data); // data赋值给上一个执行完的yield语句左边的变量
    if (temp.done) { // 已经执行完
      console.log('done')
    } else { // 未执行完
      temp.value.then((data) => {
        _next(data)
      })
    }
  }
}
```



## 十三、实现async函数

```js
/*
async function fn(args){
  // ...
}
*/

function fn(args) {
  function _run(gen) {
    return new Promise((resolve, reject) => {
      let g = gen();
      function _next(data) {
        let temp = g.next(data);
        if (temp.done) {
          return resolve(temp.value);
        }
        temp.value.then((data) => {
          _next(data);
        })
      }
      _next();
    })
  }
  return _run(function *() {
    // ...
  })
}


function fn(args){ 
  function _spawn(genF) {
    return new Promise(function(resolve, reject) { // 返回一个promise
      var gen = genF();
      function step(nextF) {
        try {
          var next = nextF();
        } catch(e) {
          return reject(e); 
        }
        if(next.done) {
          return resolve(next.value);
        } 
        Promise.resolve(next.value).then(function(v) {
          step(function() { return gen.next(v); });      
        }, function(e) {
          step(function() { return gen.throw(e); });
        });
      }
      step(function() { return gen.next(undefined); });
    });
  }
  return _spawn(function*() {
    // ...
  }); 
}
```



## 十四、实现trim函数

trim() 方法用于删除字符串的**头尾空格**。

```js
// str.trim()
String.prototype.myTrim = function () {
  let str = this;
  str.replace(/^\s+|\s+$/g, '')
}
```

> `\s`： `space`， 空格
> `+`： 一个或多个
> `^`： 开始，`^\s`，以空格开始
> `$`： 结束，`\s$`，以空格结束
> `|`：或者
> `/g`：`global`， 全局



## 十五、格式化数字（每三位加逗号）

```js
var num = 132435234.123
// 方法一
function formatNum(num) {
  return num.toLocaleString()
}
// 方法二
function formatNum(num) {
  var res = [], counter = 0;
  let numArr = (num || 0).toString().split('.');
  // 格式化小数点左边
  res[0] = '';
  for (let i = numArr[0].length-1; i >= 0; i--) {
    counter++;
    res[0] = numArr[0].charAt(i) + res[0];
    if (counter % 3 === 0 && i !== 0) res[0] = ',' + res[0]; // 排除刚好为3位的情况
  }
  
  // 格式化小数点右边
  if (numArr[1]) {
    res[1] = '', counter = 0;
    for (let i = 0; i < numArr[1].length; i++) {
      counter++;
      res[1] = res[1] + numArr[1].charAt(i);
      if (counter % 3 === 0 && i !== numArr[1].length-1) res[1] = res[1] + ','; // 排除刚好为3位的情况
    }
  }
  return res.join('.');
}

// 方法三
function formatNum(num) {
  var res = [];
  let numArr = (num || 0).toString().split('.');
  res.push(numArr[0].replace(/(\d)(?=(?:\d{3})+$)/g, '$1,')); // 格式化小数点左边
  if (numArr[1]) {
    res.push(numArr[1].replace(/(\d)(?=(?:\d{3})+)/g, '$1,')); // 格式化小数点右边
  }
  return ;
}
```

>\b：匹配单词边界 如1234551277.8945511，匹配并替换结果为 ,1234551277,.,894551,
>\B：匹配出\b之外的 如1234551277.8945511，匹配并替换结果为 1,2,3,4,5,5,1,2,7,7.8,9,4,5,5,1
>\B(?=)：匹配\B，同时符合后面条件的
>\d{3}：匹配三个数字
>\d{3}+：多次匹配
>\d{3})+\.：匹配到的位置后面存在多个\d{3}且后面刚好接一个.

```javascript
function formatNumbers(num) {
    let numStr = ''+num;
    let regExp = /\B(?=(\d{3})+\.)/g;
    return (numStr.replace(regExp, ','));
}
```



## 十六、冻结对象

```js
var constantize = (obj) => {
	Object.freeze(obj);
  Object.keys(obj).forEach((key)=>{
    if (typeof obj[key] === 'object') constantize(obj[key])
  })
}
```



## 十七、手写原生ajax

```js
var xhr;
if(window.XMLHttpRequest){
    xhr = new XMLHttpRequest();
}else{
    xhr = new ActiveXObject('Microsoft.XMLHTTP');
}
// const xhr = new XMLHttpRequest();
xhr.open('GET', url);
xhr.onreadystatechange = () => {
  if (xhr.readyState === 4) {
    console.log('请求响应完毕');
    if (xhr.status >= 200 && xhr.status < 300) {
      console.log('响应成功');
      let res = xhr.responseTest;
      console.log(res);
    } else if (xhr.status > 400) {
      console.log('响应失败')
    }
  }
}
xhr.ontimeout = (e) => {
  console.log('请求超时')
}
xhr.timeout = 3000;
xhr.send();
```

## 十八、函数柯里化

https://www.jianshu.com/p/2975c25e4d71

```js
// 实现一个add方法，使计算结果能够满足如下预期：
add(1)(2)(3) = 6;
add(1, 2, 3)(4) = 10;
add(1)(2)(3)(4)(5) = 15;


function add() {
  let _args = Array.prototype.slice.call(arguments);
  let _adder = function () { // 收集所有arguments
    _args.push(...arguments);
    return _adder;
  }
  _adder.toString = function () { // 利用toString隐式转换的特性，当最后执行时隐式转换，并计算最终的值返回
    return _args.reduce((total, num) => {
      return total + num;
    })
  }
  return _adder;
}

add(1)(2)(3)                // 6
add(1, 2, 3)(4)             // 10
add(1)(2)(3)(4)(5)          // 15
add(2, 6)(1)                // 9
```



## 十九、用 es5 实现 const

```js
// 数据描述符实现（无法抛出错误）
function myConst(key, value) {
    Object.defineProperty(window, key, {
        value: value,
        writable: false
    })
}
// 存取描述符
function myConst(key, value) {
    window[key] = value;
    Object.defineProperty(window, key, {
        get: function () {
            return value;
        },
        set: function (newValue) {
          throw new TypeError('Assignment to constant variable.');
        }
    })
}
```



## 二十、大数相加

```js
function bigNumAdd(num1, num2) { // 传入字符串
    let arr1 = num1.split('').reverse();
    let arr2 = num2.split('').reverse();
    let maxLen = Math.max(arr1.length, arr2.length);
    let temp1, temp2, sum; //当前位值、当前和
    let temp = 0; // 进位
    let res = []; // 结果
    for (let i = 0; i < maxLen; i++) {
        temp1 = arr1[i] || 0;
        temp2 = arr2[i] || 0;
        sum = Number(temp1) + Number(temp2) + temp;
        if (sum > 9) {
            temp = 1;
            res.push(sum%10);
        } else {
            temp = 0;
            res.push(sum);
        }
    }
    return res.reverse().join('');
}
```