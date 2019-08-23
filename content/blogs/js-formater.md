+++
title =  "javascript string formater"
tags = ["javascript"]
date = "2014-04-01"
+++
## 函数实现
```javascript
function format(string) {

    var args = arguments;

    var pattern = new RegExp("%([1-" + arguments.length + "])", "g");

    return String(string).replace(pattern, function (match, index) {

        return args[index];

    });

};
```
## 用法
```js
console.log(format("And the %1 want to know whose %2 you %3", "papers", "shirt", "wear"));
```
