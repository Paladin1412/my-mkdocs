## 0. Question

编写一个函数repeat( str, n )， 将字符串str重复n次并输出。



## 1. Answer 1 

创建长度为n+1的空数组，并以str为间隔来`join`。

```
function repeat( str, n ){
  return Array(n+1).join(str);
}
```



## 2. Answer 2

基于 Answer 1 的改进，使用`call`方法。

```
function repeat( str, n ){
  return Array.prototype.join.call( { length: n+1 }, str)
}
```



## 3. Answer 3 

对 Answer 2 使用闭包。

```
var repeat = (function(){
	//利用闭包, 缓存数组原型join方法, 省得每次都重复创建
	let join = Array.prototype.join,
    	obj = {};
  	return function( str, n ){
  		obj.length = n+1;
    	return join.call( obj, str)
  	}
})()
```



## 4. Answer 4

使用二分法，降低时间复杂度到o(logn)。

```
function repeat( str, n ){
	let result = [];
  	for(; n > 0; n >>= 1){
    	if(n%2) 
    		result.push(str);
    	if(n == 1) 
    		break;
    	str += str;
  	}
  	return result.join('');
}
```



## 5. Answer 5

更加直观的二分法。

```
function repeat( str, n ){
  let c = n * str.length, s = str;
  for(; n > 0; n >>= 1){
    s += s;
  }
  return s.substring( 0, c );
}
```



## 6. Answer 6

递归，时间复杂度为o(logn)。

```
function repeat( str, n ){
	if(n == 1) return str;
    let s = repeat(str, n>>1);
    s += s;
    if(n%2) s += str;
    return s;
}
```

