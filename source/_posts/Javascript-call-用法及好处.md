title: ' Javascript call 用法及好处'
author: Nico
tags:
  - JavaScript
categories:
  - JavaScript
date: 2019-04-27 23:02:00
---
call 可以改变当前函数的作用域
    
   ###### 示例
   ```
   function Person(name){
   	this.name=name;
   }
   
   Person.prototype.say=function(){
   	console.log("My name is"+this.name);
   }
   
   var person =new Person("Mike");
   person.say();//My name is Mike
   
   var student={name:'Marry'};
   person.call(student);// My name is Marry
   
   ```
   ###### 好处
   
   	使用call方法防止调用对象的原型方法被改变
    
   ```
   var slice=[].slice;
   Array.prototype.slice=function(index){
   	console.log('我是改写的slice方法'+index);
   }
   
   var array=[1,2,3,4];
   slice.call(array,3);//print out 4
  	array.slice(3);// print out 我是改写的slice方法4 (原型方法被修改掉了)
   
   ```