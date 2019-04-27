title: 闭包、立即执行函数和模块化
author: Nico
tags:
  - JavaScript
categories:
  - JavaScript
date: 2019-04-27 18:17:00
---
#### 闭包

	一个拥有许多变量和绑定了这些变量的环境的表达式（通常是一个函数），因而这些变量也是该表达式的一部分。
    
###### 示例

1、函数作为返回值
```
//函数声明
function fn(){
	var max=10;
   return function bar(x){
   	if(x>max){
    	console.log(x);
    }
   };
}
//函数表达式
var f1=fu();
f1(15); //print out 15
//执行f(15)时，max变量的取值是10
```

2、函数作为参数被传递
```
var max=10,
	fn=function(x){
    	if(x>max){
        	console.log(x);
        }
    };
    
(function(f){
	var max=100;
   f(15);
})(fn);
/*
执行f(15)时，max变量的取值是10，而不是100
*/
```

#### 立即执行函数
	立即执行函数立即执行，函数体后要有小括号以及函数必须是函数表达式（()、+、-、!等）而不能是函数声明


###### 示例

1、匿名函数包裹在一个括号运算符中，后面跟一个小括号

```
(function(test){
	console.log(test); //print out 123
})(123);
```
2、匿名函数后面跟小括号，然后整个包裹在括号运算符中

```
(function(test){
	console.log(test);//print out 1234
}(1234));

```
#### 模块化

1、使用立即执行函数

	使用"立即执行函数"（Immediately-Invoked Function Expression，IIFE），可以达到不暴露私有成员的目的。
    
 ```
 var module1 = (function(){

　　　　var _count = 0;

　　　　var m1 = function(){
　　　　　　//...
　　　　};

　　　　var m2 = function(){
　　　　　　//...
　　　　};

　　　　return {
　　　　　　m1 : m1,
　　　　　　m2 : m2
　　　　};

　　})();
  //使用上面的写法，外部代码无法读取内部的_count变量
  console.info(module1._count); //undefined
 ```
2、放大模式

```

　var module1 = (function (mod){

　　　　mod.m3 = function () {
　　　　　　//...
　　　　};

　　　　return mod;

　　})(module1);
  //上面的代码为module1模块添加了一个新方法m3()，然后返回新的module1模块。
  //宽放大模式
  var module1 = ( function (mod){

　　　　//...

　　　　return mod;

　　})(window.module1 || {});
  //模块的独立性
  　var module1 = (function ($, YAHOO) {

　　　　//...

　　})(jQuery, YAHOO);
  
```


