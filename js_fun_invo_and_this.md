# 理解Javascript方法的调用和“this”

过去的几年，我看过很多人对javascript方法的调用满是迷惑。尤其是对方法里的`this`的语义抱怨颇多。

在我（作者）看来，只要弄清楚了方法的本质，上面的以后自然清晰。事实上，这也是ECMAScript说明的作用。在某种意义上这篇文章是说明文档的简写版，但是核心内容是一致的。

## 核心本质
首先，我们方法调用的本质是一个方法的`call`方法[1]。它是如下运作的。
1. 创建一个参数列表（`argList`），包含了从头到尾的全部参数
2. 第一个参数是`thisValue`
3. 调动方法。把`this`作为`thisValue`，`argList`作为参数列表。

比如：
```javascript
function hello(thing) {
    console.log(this + " says hello " + thing);
}

hello.call("Yehuda", "world"); // => Yehuda says hello world
```
就如所看到的，我们调用了`hello`方法，`this`的值是`Yehuda`而一个单独的参数是`"world"`。这就是javascript方法调用的本质。你可以把所有其他的方法调用都看做是方便版。

## 简单的方法调用
显然用`call`调用方法非常的麻烦。Javascript允许我们这样调用方法：`hello("world")`。当我们这样调用的时候实际上是这样的：
```javascript
function hello(thing) {
    console.log("Hello " + thing);
}

// this;
hello("world");

// 等于
hello.call(window, "world");
```
这个行为会在使用**strick mode**[2]的时候发生改变：
```javascript
// this:
hello("world");

// 等于
hello.call(undefined, "world");
```
上面的例子简单来说就是：**一个方法的调用，比如：fn(...args)其实和fn.call(window[ES5-strict: undefined], ...args)**。

注意：上面的规则同样适用于inline的方法调用。`(function(){})()`和`(function() {}).call(window [ES5-strict:undefined])`。

## 成员方法
下一个常遇到的调用方法的情况是调用一个对象的方法（`person.hello()`）。在这个情况下，调用的顺序是：
```javascript
var person = {
    name: "Brendan Eich",
    hello: function(thing) {
        console.log(this + " says hello " + thing);
    }
};

// this
person.hello("world");

//等于
person.hello.call(person, "world");
```
注意：其实`hello`方法是如何绑定到对象上的。记住我们之前是如何把`hello`方法定义为一个单独的方法的。我们来看看如果方法是动态绑定到对象上的会发生什么：
```javascript
function hello(thing) {
    console.log(this + " says hello " + thing);
}

person = {name: "Brendan Eich};
person.hello = hello;

person.hello("world"); //等于person.hello.call(person, "hello");

hello("world"); // "[object DOMWindow]world"
```
注意方法里的`this`并不是一成不变的。它总是在被调用的时候被赋值。

## 使用Function.prototype.bind
如果`this`的值可以保持不变的话，那就方便多了。一个常用的方法就是使用闭包来让一个方法的this不再改变：
```javascript
var person = {
    name: "Brenda Eich",
    hello: function(thing) {
        console.log(this.name + " says hello " + thing);
    }
}

var boundHello = function(thing) {return person.hello.call(person, thing);}

boundHello("world");
```
虽然我们的`boundHello`的调用的本质还是`boundHello.call(window, "world")`，我们来看看`bind`是如何运作的：
```javascript
var bind = function(func, thisValue) {
    return function() {
        return func.apply(thisValue, aguments);
    }
}

var boundHello = bind(person.hello, person);

boundHello("world");    // "Brendan Eich says hello world"
```
为了理解上面的例子，你只需要知道两个事实。一、`arguments`是一个类似于数组的对象，它代表了全部传入方法的参数。二、`apply`方法和`call`方法本质上是一样的，只不过类数组的参数代替了逐个列出的参数。

我们的`bind`方法只是返回了一个新的方法。当它被调用的时候，我们的新方法调用了传入的方法，设置了this的值。同时其他的参数也传入进来。

因为这个基本上就是一个非常通用的模型。ES5给所有的`Function`都引入了一个新的`bind`。这个`bind`是这样工作的：
```javascript
var boundHello = person.hello.bind(person);
boundHello("world");
```
这个在你想把一个方法作为回调传入的时候非常有用：
```javascript
var person = {
    name: "Alex Russell",
    hello: function() {
        console.log(this.name + " says hello world");
    }
}

$("#some-div").click(person.hello.bind(person));

// 当div被点击的时候，"Alex Russell says hello world"就会出现
```
这确实显得很笨拙，TC39（指定新的ECMAScript标准的组织）还在制定更加优雅的，向后兼容的方法。

## 关于jQuery
因为jQuery用了非常的多的匿名方法，它内部调用了`call`方法来设置回调的`this`值。比如，所有回调都被用`call`方法设置了当前的元素为`this`值，而不是用`window`作为`this`的值。

这一点非常有用，因为匿名方法作为回调并不是特别有用，但是它会给刚接触Javascript的人一种`this`非常奇怪的印象：总之就是经常改变，难以推断。

如果你能把一个方法的调用顺其自然的理解为`func.call(thisValue, ...args)`的调用方式，那就可以轻而易举的理解Javascript里`this`的值了。