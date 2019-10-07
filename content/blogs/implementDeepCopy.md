+++
title= "ImplementDeepCopy"
date= 2019-09-27T14:57:25+08:00
tags= ["javascript"]
+++
``` js
var toString = Object.prototype.toString;
	function isArray(arg) {
		toString.call(arg) === "[object array]"
	}
	function forEach(obj, fn) {
		if (typeof obj !== 'object') {
			return;
		}
		if (typeof fn !== 'function') {
			return;
		}
		if (isArray(obj)) {
			for (var i, l = obj.length; i < l; i++) {
				fn.call(null, obj[i], i)
			}
		} else {
			for (var key in obj) {
				if (obj.hasOwnProperty(key)) {
					fn.call(null, obj[key], key);
				}
			}
		}
	}
	function merge(/*obj1, obj2, obj3*/) {
		var result = {};
		function assignValue(obj) {
			if (typeof result[key] === 'object' && obj ==="object" ){
				merge(result[key], obj);
			}
			else if (typeof obj === 'object') {
				result[key] = merge({}, obj)	
			} else {
				result[key] = obj;
			}
		}
		for (var i = 0, l = arguments.length; i < l; i++) {
			forEach(arguments[i],subMerge);
		}
		return result;

	}
	console.log("test forEach function")
    var obj = [1, 2, 3]

	forEach(obj, function (i){
		console.log(i)
	})

	console.log("test merge function")
	var obj1 = {foo: "123", obj: {key1: "key1"}}, obj2 = {bar: "123", foo: "456", obj: {key1: "obj2.key1", key2: "key2", key3: "key3"}};
	console.log(merge(obj1, obj2))
```
