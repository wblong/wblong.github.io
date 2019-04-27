title: Javascript 模块加载
author: Nico
tags:
  - JavaScript
categories:
  - JavaScript
date: 2019-04-27 16:50:00
---
`Javascript` 模块加载分为浏览器端加载和服务器端加载。

#### CommonJS

	服务器端模块化的规范，Nodejs实现了这种规范。
    
###### 规范
- 一个单独的JS文件就是一个module，拥有单独的作用域;
- 每个单独的module是一个单独的作用域。也就是说在一个文件里定义的变量和函数都是私有，对其他文件不可见，除非用exports导出了;
- 通过require来加载模块;
- 通过exports和module.exports来暴露模块中的内容。


###### CommonJS分为三部分：
- require 模块加载
- exports 模块导出
- module 模块本身

###### 示例
hello.js
```
exports.world=function(){
	console.log('Hello World');
}
```
main.js
```
var hello=require('./hello');
hello.world();
```
或hello.js
```
function Hello() { 
    var name; 
    this.setName = function(thyName) { 
        name = thyName; 
    }; 
    this.sayHello = function() { 
        console.log('Hello ' + name); 
    }; 
}; 
module.exports = Hello;
```
main.js
```
var Hello = require('./hello'); 
hello = new Hello(); 
hello.setName('BYVoid'); 
hello.sayHello(); 
```

#### AMD

	Asynchronous Module Definition 的缩写，意思是异步模块定义，是一种异步模块加载的规范。
    
    主要用于浏览器端的JS加载，为了不造成网络阻塞。只有当依赖的模块加载完毕，才会执行回调。
    
###### 规范
	AMD使用define来定义模块，require来加载模块;
    AMD允许输出的模块兼容CommonJS;
    RequireJS是AMD的一种实现。

*define(id?, dependencies?, factory);*
- id：指定义中模块的名字
- dependencies：是一个当前模块依赖的
- factory，模块初始化要执行的函数或对象。如果为函数，它应该只被执行一次。如果是对象，此对象应该为模块的输出值。

###### 示例

1、定义模块 math.js
```
define(['jquery'], function ($) {//引入jQuery模块
    return {
        add: function(x,y){
            return x + y;
        }
    };
});
```
2、调用模块 main.js
```
require(['jquery','math'], function ($,math) {
    console.log(math.add(10,100));//110
});
```
#### UMD
	Universal Module Definition。
    把前后端加载糅合在了一起，提供了一个前后端统一的解决方案。
    支持AMD和CommonJS模式。
    
###### 规范
1. 先判断是否支持Node.js模块格式（exports是否存在），存在则使用Node.js模块格式。
2. 再判断是否支持AMD（define是否存在），存在则使用AMD方式加载模块。
3. 前两个都不存在，则将模块公开到全局（window或global）

###### 示例

- root <-- this
- factory <-- function (){}

```
// if the module has no dependencies, the above pattern can be simplified to
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD. Register as an anonymous module.
        define([], factory);
    } else if (typeof exports === 'object') {
        // Node. Does not work with strict CommonJS, but
        // only CommonJS-like environments that support module.exports,
        // like Node.
        module.exports = factory();
    } else {
        // Browser globals (root is window)
        root.returnExports = factory();
  }
}(this, function () {

    // Just return a value to define the module export.
    // This example returns an object, but the module
    // can return a function as the exported value.
    return {};
}));
```
![注解](images/js_module_load.png)

