---
title: js分号的坑
date: 2017-05-11 21:10:25
tags: 
- js
- 匿名函数
categories: 
- js
---
js默认可以不加分号，但是当遇到自执行函数的时候是有可能出现坑的。
<!-- more -->
有以下代码
```javascript
var a = function func1() {
    console.log('func1');
}
(function(){
    console.log('匿名函数');
})()
```
此时输出是
> func1  
> TypeError

为什么 `"func1"` 被输出了，并且执行结果有一个TypeError错误？先来看匿名函数的执行过程

## 匿名函数
匿名函数一般是 `()()` 形式。如果是 `function functionName(){}()` 或者 `function(){}()` 的形式是存在语法错误的。这是因为
1. `function(){}()` 的形式是函数声明， `(function(){})` 加了一对括号是函数表达式。
2. js预编译的时候会先解释函数声明，但是会跳过函数表达式。
3. 在执行的时候，js执行到 `function(){}()` 会直接跳过，后面紧跟着一对 `()` ，因为没有函数表达式或者函数名会报错；如果是执行到 `(function(){})` ，因为这是一个函数表达式，在执行的时候会求解得到返回值，并且返回值是一个函数，紧跟着 `()` 可以调用该函数。

所以一般用 `()` 得到函数表达式，然后紧跟着一对 `()` 调用该匿名函数。从原理上讲，自执行函数只要得到函数表达式就可以，不一定非要用 `()` 。有以下常见方法：
- 圆括号`()`
- 赋值操作`=`
- 逻辑运算`<`，`>`，`|`，`&`，`!`，`==`，`~`
- 数字运算`+`，`-`，`*`，`/`
- 分组`,`
- void

```javascript
(function() {
    console.log('圆括号1');
}());

(function() {
    console.log('圆括号2');
})();

var a = function(arg) {
    console.log(arg);
}('赋值运算');

!function() {
    console.log('逻辑运算');
}();

-function() {
    console.log('数字运算');
}();

void function() {
    console.log('void');
}();

1,function() {
    console.log('分组');
}()
```
控制台正常输出
> 圆括号1  
> 圆括号2  
> 赋值运算  
> 逻辑运算  
> 数字运算  
> void  
> 分组  

## 原因
现在再会过头看，报错的原因。我们知道匿名函数是可以传参的，而一个函数本质上是一个对象，也当然可以作为参数。如果把匿名函数当作参数arg，实际上开头的代码抽象为

```javascript
var a = function func1() {
    console.log('func1');
}(arg)()
```
由于缺少一个分号 `;` ，前面一部分部分被解释器当作是前面提到的赋值方式带参数执行匿名函数，并且后面的匿名函数被当作是参数，所以输出 `“func1”` 。最后一对 `()` 由于前面执行结果没有返回函数表达式，因此报错。所以解决办法是
**1.加上一个 `;`**

```javascript
var a = function func1() {
    console.log('func1');
};
(function(){
    console.log('匿名函数');
})()
```
输出结果
> 匿名函数

**2.改用其他的匿名函数自执行方法**

```javascript
var a = function func1() {
    console.log('func1');
}
!function(){
    console.log('匿名函数');
}()
```
输出结果
> 匿名函数

相比于圆括号方式，使用 `!` 方式减少一个字符，并且能避免 `;` 引起的错误。一般js代码压缩也使用这种方式。