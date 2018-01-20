JavaScript的this和作用域

本文主要讨论一下JS的作用域和`this`关键字。作用域，就是你的方法或者变量可访问的区域，是他们执行的上下文。如果你见过这样的代码：
```js
function someFunc() {
  var _this = this;
  something.on("click", function() {
    console.log(_this);
  });
};
```

你就会很好奇为什么要用`this`赋值给一个变量`_this`呢？你看完这篇文章就会清楚这个问题的答案了。

第一种作用域叫做全局作用域（**Global Scope**）这很容易定义，如果一个方法、变量是全局作用域的，那么它就可以从任何的地方访问到。在浏览器里，全局作用域就是`window`对象。所以，如果你的代码里有：
```js
var x = 9;
```
你其实是在给`window.x`赋值为`9`（在浏览器里运行的时候）。如果你喜欢的话，也可以写成`window.x = 9`，当然这不是很必要。全局作用域对象的属性可以在代码的任何地方访问到。

另外的作用域就只有**局部作用域**了。在JavaScript里一般就是一个方法内部的作用域。比如：
```js
function someFunction() {
  var x = 5;
}
console.log(x); // undefined
```
变量`x`是在方法内部初始化的，那么就只能在方法内部访问。

## 一些需要注意的地方
如果你声明了一个变量，而且在声明的时候忘记使用`var`关键字。那么，这个变量自动被置为全局变量。比如：
```js
function someFunction(){
  x = 5;
}

// 执行一个这个方法，完成x的全局设置
someFunction();

console.log(x); // 5
```
这是一个非常差的实践。你应该尽量少的往全局作用域添加属性。这也是为什么你会经常看到一些库，比如jQuery会这么干：
```js
(function() {
  var jQuery = {/*全部的方法都在这里*/};
  window.jQuery = jQuery;
})();
```
把全有的属性、方法都放在一个方法里。然后立刻执行这个方法就会把全部的属性和方法都绑定在了局部作用域里。最后把`jQuery`对象绑定到全局作用域，间接的把之前定义的属性和方法都暴露出来方便调用。显然我大量的减少了jQuery的代码，但是这就是jQuery代码如何工作的。

由于局部作用域只有通过方法的定义来实现。任何在一个方法内部定义的方法都可以访问外部方法里定义的变量。比如：
```js
function outer() {
  var x = 5;
  function inner() {
    console.log(x); // 5
  }

  inner();
}
```
但是`outer()`方法不可以访问`inner()`方法定义的任何变量。
```js
function outer() {
  var x = 5;
  function inner() {
    console.log(x); // 5
    var y = 100;
  }

  inner();
  console.log(y); // undefined
}
```

这很容易理解。但是当我们试图要探究`this`关键字的时候就又变得复杂了。我相信很多人都遇到过这样的问题：
```js
${'myLink'}.on('click', function() {
  console.log(this);  // 指向myLink

  $.ajax({
    // 设置ajax相关
    success: function() {
      console.log(this);  // 指向的是全局对象。 ？？？
    }
  });
});
```
`this`是在方法执行的时候自动赋值的变量。它的值和方法的调用方式有很大的关系。比如：
```js
function foo() {
  console.log(this);  // global object
}

theApp = {};
theApp.foo = function() {
  console.log(this);  // 指向theApp对象
}

var link = doeument.getElementById('myLink');
link.addEventListener('click', function() {
  console.log(this); // 指向link
});
```
MDN对第[三个例子的解释](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)非常到位:
```
事件的this总是指向出发这个事件的元素对象。比如使用一个通用的事件监听器来处理一系列的有相似事件的元素。当使用addEventListener把一个方法添加到一个元素的事件处理器的时候，这个方法的this值就被改变了。注意，this的值是从调用者里传给方法的。
```

这么做：`$('myLink').on('click', function() {})`意味着link被点击的时候方法就会执行。由于这个方法是处理link的事件的，所以this的值就是这个link元素。

在Ajax请求的success回调方法只是一个普通的方法。所以，当它被调用的时候this指向的是全局对象。任何对象，不是对象方法或者事件的时候会遇到的情况。

上面的原因也就是你在很多地方看到`var _this = this;`或者`var that = this;`的原因。来看一个例子：
```js
$('myLink').on('click', function() {
  console.log(this); //指向myLink
  var _this = this;
  $.ajax({
    // ajax设置
    success: function() {
      console.log(this);  // 指向全局对象
      console.log(_this); // 指向myLink
    }
  });
});
```

原文地址：https://javascriptplayground.com/javascript-variable-scope-this/