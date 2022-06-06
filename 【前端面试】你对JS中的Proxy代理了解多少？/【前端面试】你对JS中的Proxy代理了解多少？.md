# 前言

就目前而言，前端框架基本上被 Vue 和 React 瓜分得差不多了。如果你去面试一个前端岗位，那么或多或少都会问你一些关于 Vue 和 React 框架得知识，无论是原理还是使用，我们都有必要去了解一番。

数据响应式可以说是这些框架的一大特色与核心，这里我们就拿 Vue 来说。在 Vue2.x 时代，实现数据响应式主要是使用 Object.defineProperty()这个 API 来实现的，而到了 Vue3.x 时代，数据响应式主要是使用 Proxy()来实现的。

如果你还不了解 Proxy，那么你很有必要跟着本篇文章学习一下！

# 1.基本概念

想要学习一个新的 API 或者知识，不能一上来就看它怎么使用。我们要学习从基本概念入手，这样才能做到有始有终。

Proxy 是在 ES6 中才被标准化的，而 Vue2.x 版本是基于 ES6 版本之前的 Object.defineProperty()设计的，我们先来看下官方是怎么解释 Proxy 的。

**官网解释：**

>Proxy 对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）。
为了方便大家好理解，这里先抓几个关键词出来：

* 对象
* 创建对象代理
* 拦截
* 自定义
从上面的关键词大家应该能够揣摩个一二了，首先 Proxy 是一个对象，它可以给另外一个对象创建一个代理，代理可以简单理解为代理某一个对象，然后通过这个代理对象，可以针对于该对象提前做一些操作，比如拦截等。

**通俗的解释：**

>假如我们有一个对象 obj，使用 Proxy 对象可以给我们创建一个代理，就好比我们打官司之前，可以先去律师事务所找一个律师，律师全权代理我们。有了这个代理之后，就可以对这个 obj 做一些拦截或自定义，比如对方想要直接找我们谈话时，我们的律师可以先进行拦截，他来判断是否允许和我谈话，然后再做决定。这个律师就是我们对象的代理，有人想要修改 obj 对象，必须先经过律师那一关。
基本概念其实不复杂，有些小伙伴不太理解的原因大多是平时写代码的时候，以为对象就是一个独立的变量，比如声明了一个对象 obj={name:"小猪课堂"}，我们通常也不会去做什么拦截，想改就改。

这就是一个惯性思维！

# 2.如何使用

既然我们知道了 Proxy 的作用，那么我们如何使用它呢？或者说如何给一个对象创建代理。

Proxy 的使用非常简单，我们可以使用 new 关键字实例化它。

**代码如下：**

```javascript
const p = new Proxy(target, handler)
```
代码非常的简单，重点是我们需要掌握 Proxy 接收的参数。
**参数说明**

**target:**

需要被代理的对象，它可以是任何类型的对象，比如数组、函数等等，注意不能是基础数据类型。

**示例代码：**

```javascript
<script>
  let obj = {
    name: '小猪课堂',
    age: 23
  }
  let p = new Proxy(obj, handler);
</script>
```
**handler:**
它是一个对象，该对象的属性通常都是一些函数，handler 对象中的这些函数也就是我们的处理器函数，主要定义我们在代理对象后的拦截或者自定义的行为。handler 对象的的属性大概有下面这些，具体使用方法我们在后面章节详解：

* handler.apply()
* handler.construct()
* handler.defineProperty()
* handler.deleteProperty()
* handler.get()
* handler.getOwnPropertyDescriptor()
* handler.getPrototypeOf()
* handler.has()
* handler.isExtensible()
* handler.ownKeys()
* handler.preventExtensions()
* handler.set()
* handler.setPrototypeOf()
我们使用 new 关键词后生成了一个代理对象 p，它就和我们原来的对象 obj 一样，只不过它是一个 Proxy 对象，我们打印出来看看就能更好理解了。

**示例代码：**

```javascript
<script>
  let obj = {
    name: '小猪课堂',
    age: 23
  }
  let p = new Proxy(obj, {});
  console.log(obj);
  console.log(p);
</script>
```
**输出结果：**
![图片](data:image/png;base64,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)


# 3.Handler 对象详解

上一节的使用只是简单的初始化了一个代理对象，而我们需要重点掌握的是 Proxy 对象中的 handler 参数。因为我们所有的拦截操作都是通过这个对象里面的函数而完成的。

就好比律师全权代理了我们，那他拦截之后能做什么呢？或者说律师拦截之后他有哪些能力呢？这就是我们 handler 参数对象的作用了，接下来我们就一一来讲解下。

## 3.1 handler.apply

该方法主要用于函数调用的拦截，比如我们代理的对象是一个函数，那么我们代理这个函数之后，可以在它调用之前做一些我们想做的事。

**语法：**

```javascript
// 函数拦截
let p1 = new Proxy(target, {
  apply: function (target, thisArg, argumentsList) {
  }
});
```
**参数解释：**
* target：被代理对象，也就是目标函数
* thisArg：调用时的上下文对象，也就是 this 指向，它绑定在 handler 对象上面
* argumentsList：函数调用的参数数组
**使用案例：**

```javascript
function sum(a, b) {
  return a + b;
}
let p1 = new Proxy(sum, {
  apply: function (target, thisArg, argumentsList) {
    return argumentsList[0] + argumentsList[1] * 100;
  }
});
// 正常调用
console.log(sum(1, 2)); // 3
// 代理之后调用
console.log(p1(1, 2)); // 201
```
上段代码中我们代理了 sum 函数对象，并产生了新的 p1 代理对象，在 p1 代理对象里面，我们对函数的调用做了拦截，让它返回了新的值。
**注意：**

我们这里代理的函数对象必须是可调用的，也就是 target 可调用，否则会报错。

## 3.2 handler.construct

该方法主要是用于拦截 new 操作符的，我们通常使用 new 操作符都是在函数的情况下，但是我们不能说 new 操作符只能作用与函数，确切的说 new 操作符必须作用于自身带有[[Construct]]内部方法的对象上，而这种对象通常就是函数，总之一句话，使用 new targe 是必须有效的。

**语法：**

```javascript
// 构造函数拦截
let p2 = new Proxy(target, {
  construct: function (target, argumentsList, newTarget) {
  }
});
```
**参数解释：**
* target：被代理对象，需要能够使用 new 操作符初始化它的实例，通常就是一个函数
* argumentsList：使用 new 操作符是传入的参数列表
* newTarget：被调用的构造函数，也就是 p2
**使用案例：**

```javascript
let p2 = new Proxy(function () { }, {
  construct: function (target, argumentsList, newTarget) {
    return { value: '我是' + argumentsList[0] };
  }
});
console.log(new p2("小猪课堂")); // {value: '我是小猪课堂'}
```
上段代码中 p2 就是一个构造函数，只不过是代理之后的新函数，我们使用 new 操作符实例化它的，首先就会去执行 handler 里面的 construct 方法。
**注意：**

这里有两个点需要大家注意

* target 必须能够使用 new 操作符初始化
* construct 必须返回一个对象
## 3.3 handler.defineProperty

这个方法其实比较有意思，Object.defineProperty 方法本身就有拦截对象的意思在里面，但是我们的 Proxy 对象可以正针对 Object.defineProperty 操作进行拦截，对于 Object.defineProperty 方法不熟悉的同学可以先去学学。

**语法：**

```javascript
// 拦截 Object.defineProperty
let p3 = new Proxy(target, {
  defineProperty: function (target, property, descriptor) {
  }
});
```
**参数解释：**
* target：被代理对象
* property：属性名，也就是当我们使用 Object.defineProperty 操作的对象的某个属性
* descriptor：待定义或修改的属性的描述符
**使用案例：**

```javascript
let p3 = new Proxy({}, {
  defineProperty: function (target, property, descriptor) {
    descriptor.enumerable = false; // 修改属性描述符
    console.log(property, descriptor);
    return true;
  }
});
let desc = { configurable: true, enumerable: true, value: 10 };
Object.defineProperty(p3, 'a', desc); // a {value: 10, enumerable: false, configurable: true}
```
上段代码中我们使用 Proxy 代理了一个空对象，并产生了新的代理对象 p3，当使用 Object.defineProperty 操作 p3 对象时，就会触发 handler 中的 defineProperty 方法。
**注意：**

* 被代理的对象必须要能被扩展
* hanlder 中的 defineProperty 方法必须返回一个 Boolean 值
* 不能添加或者修改一个属性为不可配置的，如果它不作为一个目标对象的不可配置的属性存在的话
## 3.4 handler.deleteProperty

该方法用于拦截对对象属性的 delete 操作，我们经常使用 delete 删除对象中的某个属性，我们可以使用 deleteProperty 方法对该做进行拦截。

**语法：**

```javascript
let p4 = new Proxy(target, {
  deleteProperty: function (target, property) {
  }
});
```
**参数解释：**
* target：被代理的目标对象
* property：将要被删除的属性
**使用案例：**

```javascript
let p4 = new Proxy({}, {
  deleteProperty: function (target, property) {
    console.log("将要删除属性：", property)
  }
});
delete p4.a; // 将要删除属性：a
```
当我们删除 p4 对象的属性时，便会执行 handler 中的 deleteProperty 方法。
**注意：**

代理的目标对象的属性必须是可配置的，即可以删除，否则会报错。

## 3.5 handler.get

该方法用于拦截对象的读取属性操作，比如我们要读取某个对象的属性，就可以使用该方法进行拦截。

**语法：**

```javascript
// 拦截读取属性操作
let p5 = new Proxy(target, {
  get: function (target, property, receiver) {
  }
});
```
**参数解释：**
* target：被代理的目标对象
* property：想要获取的属性名
* receiver：Proxy 或者继承 Proxy 的对象
**使用案例：**

```javascript
// 拦截读取属性操作
let p5 = new Proxy({}, {
  get: function (target, property, receiver) {
    console.log("属性名：", property); // 属性名：name
    console.log(receiver); // Proxy {}
    return '小猪课堂'
  }
});
console.log(p5.name); // 小猪课堂
```
可以看到我们代理的对象其实是一个空对象，但是我们获取 name 属性是是返回了值的，其实是在 handler 对象中的 get 函数返回的。
**注意：**

代理的对象属性必须是可配置的，get 函数可以返回任意值。

## 3.6 handler.getOwnPropertyDescriptor

该方法用于拦截 Object.getOwnPropertyDescriptor 操作，也可以说它是该方法的钩子，如果对 getOwnPropertyDescriptor 还不熟悉的小伙伴可以先去了解一下。

**语法：**

```javascript
let p6 = new Proxy(target, {
  getOwnPropertyDescriptor: function (target, prop) {
  }
});
```
**参数解释：**
* target：被代理的目标对象
* prop：返回属性名称的描述
**使用案例：**

```javascript
let p6 = new Proxy({ name: '小猪课堂' }, {
  getOwnPropertyDescriptor: function (target, prop) {
    console.log('属性名称：' + prop); // 属性名称：name
    return { configurable: true, enumerable: true, value: '张三' };
  }
});
console.log(Object.getOwnPropertyDescriptor(p6, 'name').value); // 张三
```
上段代码中我们在拦截其中重新设置了属性描述，所以最后打印的 value 是”张三“。
**注意：**

* getOwnPropertyDescriptor 必须返回一个 object 或 undefined。
* 使用 getOwnPropertyDescriptor 时，目标对象的该属性必须存在 
## 3.7 handler.getPrototypeOf

当我们读取代理对象的原型时，会触发 handler 中的 etPrototypeOf 方法。

**语法：**

```javascript
let p7 = new Proxy(obj, {
  getPrototypeOf(target) {
  }
});
```
**参数解释：**
* target：被代理的目标对象
**使用案例：**

```javascript
let p7 = new Proxy({}, {
  getPrototypeOf(target) {
    return { msg: "拦截获取对象原型操作" }
  }
});
console.log(p7.__proto__); // {msg: '拦截获取对象原型操作'}
```
**以下操作会触发代理对象的该拦截方法：**
* Object.getPrototypeOf()
* Reflect.getPrototypeOf()
* __proto__
* Object.prototype.isPrototypeOf()
* instanceof
**注意：**

getPrototypeOf 方法必须返回一个对象或者 null。

## 3.8 handler.has

该拦截方法主要是针对 in 操作符的，in 操作符通常用来检测某个属性是否存在某个对象内。

**语法：**

```javascript
let p8 = new Proxy(target, {
  has: function (target, prop) {
  }
});
```
**参数解释：**
* target：被代理的目标对象
* prop：需要检查是否存在的属性
**以下操作可以触发该拦截函数：**

* 属性查询：foo in proxy
* 继承属性查询：foo in Object.create(proxy)
* with 检查: with(proxy) { (foo); }
* Reflect.has()
**使用案例：**

```javascript
let p8 = new Proxy({}, {
  has: function (target, prop) {
    console.log('检测的属性: ' + prop); // 检测的属性: a
    return true;
  }
});
console.log('a' in p8); // true
```
上段代码中我们代理对象是其实没有 a 属性，但是我们拦截之后直接返回的一个 true。
**注意：**

has 函数返回的必须是一个 Boolean 值。

## 3.9 handler.isExtensible

Object.isExtensible()方法主要是用来判断一个对象是否可以扩展，handler 中的 isExtensible 方法可以拦截该操作。

**语法：**

```javascript
// 拦截 Object.isExtensible()
let p9 = new Proxy(target, {
  isExtensible: function (target) {
  }
});
```
**参数解释：**
* target：被代理的目标对象
**使用案例：**

```javascript
let p9 = new Proxy({}, {
  isExtensible: function (target) {
    console.log('操作被拦截了');
    return true;
  }
});
console.log(Object.isExtensible(p9));
```
**注意：**
isExtensible 方法必须返回一个 Boolean 值或可转换成 Boolean 的值。

## 3.10 handler.ownKeys

静态方法 Reflect.ownKeys() 返回一个由目标对象自身的属性键组成的数组。handler 对象的 ownKeys 方法可以拦截该操作，除此之外，还有一些其它操作也会触发 ownKeys 操作。

**语法：**

```javascript
let p10 = new Proxy(target, {
  ownKeys: function (target) {
  }
});
```
**参数解释：**
* target：被代理的目标对象
以下操作会触发拦截：

* Object.getOwnPropertyNames()
* Object.getOwnPropertySymbols()
* Object.keys()
* Reflect.ownKeys()
**使用案例：**

```javascript
let p10 = new Proxy({}, {
  ownKeys: function (target) {
    console.log('被拦截了');
    return ['a', 'b', 'c'];
  }
});
console.log(Object.getOwnPropertyNames(p10)); // ['a', 'b', 'c']
```
**注意：**
ownKeys 的结果必须是一个数组，数组的元素类型要么是一个 String，要么是一个 Symbol。

## 3.11 handler.preventExtensions

Object.preventExtensions()方法让一个对象变的不可扩展，也就是永远不能再添加新的属性。handler.preventExtensions 可以拦截该项操作。

**语法：**

```javascript
let p11 = new Proxy(target, {
  preventExtensions: function (target) {
  }
});
```
**参数解释：**
* target：被代理的目标对象
**使用案例：**

```javascript
let p11 = new Proxy({}, {
  preventExtensions: function (target) {
    console.log('被拦截了');
    Object.preventExtensions(target);
    return true
  }
});

Object.preventExtensions(p11);
```
**以下操作会触发拦截：**
* Object.preventExtensions()
* Reflect.preventExtensions()
**注意：**

如果目标对象是可扩展的，那么只能返回 false

## 3.12 handler.set

当我们给对象设置属性值时，将会触发该拦截。

**语法：**

```javascript
let p12 = new Proxy(target, {
  set: function (target, property, value, receiver) {
  }
});
```
**参数解释：**
* target：被代理的目标对象
* property：将要被设置的属性名
* value：新的属性值
* receiver：最初被调用的对象，通常就是 proxy 对象本身
**以下操作会触发拦截：**

* 指定属性值：proxy[foo] = bar 和 proxy.foo = bar
* 指定继承者的属性值：Object.create(proxy)[foo] = bar
* Reflect.set()
**使用案例：**

```javascript
let p12 = new Proxy({}, {
  set: function (target, property, value, receiver) {
    target[property] = value;
    console.log('property set: ' + property + ' = ' + value); // property set: a = 10
    return true;
  }
});
p12.a = 10; 
```
**注意：**
set() 方法应当返回一个布尔值

## 3.13 handler.setPrototypeOf

Object.setPrototypeOf() 方法设置一个指定的对象的原型，当调用该方法修改对象的原型时就会触发该拦截。

**语法：**

```javascript
let p13 = new Proxy(target, {
  setPrototypeOf: function (target, prototype) {
  }
});
```
**参数解释：**
* target：被代理的目标对象
* prototype：对象新原型或者为 null
**使用案例：**

```javascript
let p13 = new Proxy({}, {
  setPrototypeOf: function (target, prototype) {
    console.log("触发拦截"); // 触发拦截
    return true;
  }
});
Object.setPrototypeOf(p13, {name: '小猪课堂'})
```
**注意：**
如果成功修改了[[Prototype]], setPrototypeOf 方法返回 true,否则返回 false。

# 总结

很多小伙伴可能因为 Vue3 的原因知道了 Proxy 代理的存在，但是很多都只了解 set、get 等方法，其实 Proxy 提供了很多拦截供我们使用，具体在什么场景下使用什么拦截函数，还需要自己独立思考。

