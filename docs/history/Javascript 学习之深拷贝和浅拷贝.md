> 老话题，老是老，话也够多，题还是题。

## 1. 概念

深拷贝和浅拷贝是针对Object和Array这样的复杂对象提出的，浅拷贝之拷贝一层对象的属性，而深拷贝递归复制所有层级。



## 2. 浅拷贝

看以下代码：

```
var obj = { a:1, arr: [2,3] };

var shallowObj = shallowCopy(obj);

function shallowCopy(src) {
  var dst = {};
  for (var prop in src) {
    if (src.hasOwnProperty(prop)) {
      dst[prop] = src[prop];
    }
  }
  return dst;
}

//以上已完成浅拷贝
shallowObj.a = 100;
shallowObj.arr[0] = 200;

console.log(obj, shallowObj);

```

输出结果为：{a: 1, arr: [200, 3]} , {a: 100, arr: [200, 3]}

解析： JavaScript 存储 Object 和 Array 这种类复杂对象都是存地址的，所以浅拷贝会导致 obj.arr 和 shallowObj.arr 指向同一块内存地址。



## 3. 深拷贝

针对以上问题，看以下代码：

```
function deepCopy(src){
    if(typeof src !== 'object') //递归终止条件
    	return src;
    	
    var dst = src.constructor === Array ? [] : {};
    	
    if(window.JSON){ 
        dst = JSON.stringify(src); //系列化对象
        dst = JSON.parse(dst); //还原
    } else { 
        for(var i in obj){
            dst[i] = typeof src[i] === 'object' ? 
            deepCopy(src[i]) : src[i]; 
        }
    }
    return dst;
};

```



## 4. 关于Object.assign()

`Object.assign()` 只是一级属性复制，从某种意义上来讲，也算作浅拷贝。

查看`Object.assign()`的官方`Polyfill`，如下：

```
if (!Object.assign) {
  Object.defineProperty(Object, 'assign', {
    enumerable: false,
    configurable: true,
    writable: true,
    value: function(target) {
      'use strict';
      // assign方法的第一个参数为空，则抛错
      if (target === undefined || target === null) {
        throw new TypeError('Cannot convert first argument to object');
      }

      var to = Object(target);
      // 遍历剩余参数
      for (var i = 1; i < arguments.length; i++) {
        var nextSource = arguments[i];
        // 参数为空，则continue
        if (nextSource === undefined || nextSource === null) {
          continue;
        }
        nextSource = Object(nextSource);

        // 获取参数的所有key值，并遍历
        var keysArray = Object.keys(nextSource);
        for (var nextIndex = 0, len = keysArray.length; nextIndex < len; nextIndex++) {
          var nextKey = keysArray[nextIndex];
          var desc = Object.getOwnPropertyDescriptor(nextSource, nextKey);
          // 如果不为空且可枚举，则直接浅拷贝赋值
          if (desc !== undefined && desc.enumerable) {
            to[nextKey] = nextSource[nextKey];
          }
        }
      }
      return to;
    }
  });
}
```

由此可见，`Object.assign()` 只对顶层属性做了赋值，没有继续递归而把所有下一层的属性做深拷贝。