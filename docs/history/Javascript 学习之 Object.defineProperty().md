## 1. 概念

The `Object.defineProperty()` method defines a new property directly on an object, or modifies an existing property on an object, and returns the object.



## 2. 参数

```
Object.defineProperty(obj, prop, descriptor)
```

`obj`：目标对象

`prop`：被定义或改变的属性名

`descriptor`：目标属性的特性



## 3. `descriptor`参数

**value**：*Defaults to undefined.* 属性的值。

**writable**：*Defaults to false.* 若为false，属性的值只读。

**configurable**：*Defaults to false.* 若为false，就不能再设置或删除其value，writable，enumerable，以及configurable。

**enumerable**：*Defaults to false.* 是否能在for...in循环中遍历出来或在Object.keys中列举出来。

**get**：*Defaults to undefined.*

**set**：*Defaults to undefined.*

```
var o={}, bValue = 38;
Object.defineProperty(o, 'b', {
  get: function() { 
  	console.log('get读取b的value时触发')
  	return bValue; 
  },
  set: function(newValue) { 
  	console.log('set设置b的value时触发')
  	bValue = newValue; 
  },
  enumerable: true,
  configurable: true
});
o.b; // get读取b的value时触发 38
```

