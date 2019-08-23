+++
title = "javascript 中 调用new操作符时候发生了什么？"
tags = ["javascript"]
date = 2018-02-16
+++
+ 第一步，创建一个空对象
+ 第二步，空对象的__proto__指向constractor的prototype
+ 第三步，用该空对象为this调用constractor ES6中Reflect.constructor函数和test.js中createClass实现同样功能
```javascript
function createClass(constructor,args){
    var o = {};
    o.__proto__ = constructor.prototype;
    constructor.call(o,...args);
    return o;
}
function Animal(type,cando){
    this.type = type;
    this.cando = cando;
}
Animal.prototype.showType = function () {
    console.log('I am a ' + this.type + ",and I can " + this.cando);
}
var cat = createClass(Animal,["cat","捉老鼠"]);
cat.showType();
```
```