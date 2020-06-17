> 想换工作了，再不换感觉就要废了；做什么都可以，只要能让技能提升，但求日后能有碗饭吃。胡乱刷刷题，说不定对面试有用。

<br />

## Q1. 二次封装函数 

已知函数 fn 执行需要 3 个参数。请实现函数 partial，调用之后满足如下条件：
1、返回一个函数 result，该函数接受一个参数
2、执行 result(str3) ，返回的结果与 fn(str1, str2, str3) 一致

```
function partial(fn, str1, str2) {
    return function(str3){
        return fn.call(null, str1, str2, str3)
    }
}
```

<br />

## Q2. 使用 apply 调用函数

实现函数 callIt，调用之后满足如下条件
1、返回的结果为调用 fn 之后的结果
2、fn 的调用参数为 callIt 的第一个参数之后的全部参数

```
function callIt(fn) {
    var args = Array.prototype.slice.call(arguments);
    return fn.apply(null, args.slice(1))
}
```

<br />

## Q3. 二次封装函数

实现函数 partialUsingArguments，调用之后满足如下条件：
1、返回一个函数 result
2、调用 result 之后，返回的结果与调用函数 fn 的结果一致
3、fn 的调用参数为 partialUsingArguments 的第一个参数之后的全部参数以及 result 的调用参数

```
function partialUsingArguments(fn) {
	var args = Array.prototype.slice.call(arguments);
    return function(){
        var args1 = Array.prototype.slice.call(arguments);
        return fn.apply(null, args.slice(1).concat(args1))
    }   
}
```

<br />

## Q4. 柯里化

已知 fn 为一个预定义函数，实现函数 curryIt，调用之后满足如下条件：
1、返回一个函数 a，a 的 length 属性值为 1（即显式声明 a 接收一个参数）
2、调用 a 之后，返回一个函数 b, b 的 length 属性值为 1
3、调用 b 之后，返回一个函数 c, c 的 length 属性值为 1
4、调用 c 之后，返回的结果与调用 fn 的返回值一致
5、fn 的参数依次为函数 a, b, c 的调用参数

```
function curryIt(fn) {
	var args = [], 
		l = fn.length;  //函数的 length 为形参个数
	
	var tmpFunc = function(arg){
      args.push(arg);
      
      //递归停止条件，当参fn参数收集完毕
      if(args.length >= fn.length) 
      	return fn.apply(null, args)
      	
      return tmpFunc
	}
	
	return tmpFunc;
}
```

```
var fn = function (a, b, c) {return a + b + c}; 
curryIt(fn)(1)(2)(3);  //output: 6
```

<br />

## Q5. 检查重复字符串

给定字符串 str，检查其是否包含连续重复的字母（a-zA-Z），包含返回 true，否则返回 false

```
function containsRepeatingLetter(str) {
	var reg = new RegExp(/([a-zA-Z])\1/);
    return reg.test(str)
}
```

**解析**：用到正则表表达式**分组和引用**的概念，利用()进行分组，使用斜杠加数字表示引用。

注意：引用的就是匹配成功后的文本内容，引用的是结果，而不是表达式。

<br />

## Q6. Array.reduce

用 ES6 的 Array.reduce() 方法计算一个字符串中每个字符出现的次数。

```
var str = 'aabbccabbccc';

Array.from(str).reduce((res, c) => {
  res[c] ? res[c]++ : res[c] = 1;
  return res;
},{});
```

<br />

## Q7. Thunk

thunk 函数定义：把一个函数的**执行参数**，**回调函数** 以及**函数本身**进行包装，变成单数的形式。

> fn( args, callback )        ==>      thunk( fn )( args )( callback )

```
function thunk(fn){
  return function(){
    var args = Array.from(arguments);
    return function(callback){
      args.push(callback);
      return fn.apply(null, args);
    }
  }
}

// 测试用例 add
function add(x,y,z,cb){
  cb();
  return x+y+z;
  //setTimeout(() => cb(x+y+z), 3000);
}

// 直接调用 add 函数
add(1,2,3, () => console.log('add completed!'))

// thunkify add 函数
thunk(add)(1,2,3)(() => console.log('thunk add completed!'));
```

<br />

##Q8. 单链表反转



## Tips

持续更新中。

