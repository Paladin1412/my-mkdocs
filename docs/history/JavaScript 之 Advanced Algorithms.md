> 本文主要讨论两个经典问题：**动态规划 (Dynamic Programming)** 和**贪心算法 (Greedy Algorithm)**。念书时也曾接触过这俩概念，如今又看到，重新整理总结一番。

<br />



# 1. 动态规划

>  “Dynamic programming is a technique that is sometimes considered the opposite of recursion.”

**递归**自顶向下，假设整个问题已解决，再解决小问题；**动态规划 (Dynamic Programming)** 反之，自底向上，解决小问题，然后把它们组合起来，成为整个问题的解。



## 1.1 Fibonacci数列

```
// 递归解法
function recurFib(n) {
	if (n < 2) 
   		return n;
   	else
      	return recurFib(n-1) + recurFib(n-2);
}
```

递归函数多次被调用，形成一个递归树，事实上在内存中运行效率极其低下。

动态规划使用`val`数组来存储中间结果：

```js
// 动态规划解法
function dynFib(){
	var val = [];
   	for (var i = 0; i <= n; ++i) {
    	val[i] = 0;
   	}
   	if (n == 1 || n == 2) {
      	return 1;
   	} else {
    	val[1] = 1;
      	val[2] = 2;
      	for (var i = 3; i <= n; ++i) {
        	val[i] = val[i-1] + val[i-2];
      	}
      	return val[n-1];
   }  
}
```



## 1.2 最长公共子串

寻找两个字符串中最长公共子串 ( Longest Common Substring ) 是动态规划的经典问题。

算法思想： 用二维数组记录两个字符串在相同位置的字母的比较结果。

1. 二维数组初始化为全0；
2. 每次两个字符串相同位置匹配时，二维数组相应位置加1，否则置0。

```
function lcs(word1, word2){
	var max = 0;     // 公共子串最大长度
	var index = 0;   // 标记公共子串位置
	
	var lcsarr = new Array(word1.length);  // 声明 lcsarr 矩阵，并初始化   
	for(var i = 0; i < word1.length; i++){
		lcsarr[i] = new Array(word2.length);
		for(var j = 0; j < word2.length; j++){
			lcsarr[i][j] = 0; 
		}
	}

	for(var i = 0; i < word1.length; i++){
		for(var j = 0; j < word2.length; j++){
			if(word1[i] == word2[j]){
				if(i == 0 || j == 0){
					lcsarr[i][j] = 1;
				}else{
					lcsarr[i][j] = lcsarr[i-1][j-1] + 1;
				}
				
				if(max < lcsarr[i][j]){
					max = lcsarr[i][j]; 
                	index = i;
				}
			}
		}
	}

	//console.log(lcsarr)
	return word1.substr(index-max+1, max);
}
```



## 1.3 背包问题

背包问题 (Knapsack Problem)：一堆物品，各有 value 和 size 两个属性，背包限定抓取物品 size 的总大小，如何让所选之物总值 value 最大。

假设有如下物品：

```
var capacity = 16;           // 背包大小
var n = 5;                   // 物品数量
var value = [4,5,10,11,13];  // 物品价值
var size = [3,4,7,8,9];      // 物品大小
```



### 1.3.1 递归解决背包问题

```
// 返回值为背包内物品价值总和
function knapsack(capacity, n, size, value){
	if(capacity == 0 || n == 0)  // 递归终止条件
  		return 0;  
  	
  	if(size[n-1] > capacity)  // 若装不下，则弃之
  		return knapsack(capacity, n-1, size, value)
  	
  	var a = knapsack(capacity, n-1, size, value),
  		b = knapsack(capacity - size[n-1], n-1, size, value) + value[n-1];
  	
  	return a > b ? a : b;
}
```



### 1.3.2 动态规划解决背包问题

算法思想：用二维矩阵记录背包和物品在扩张过程中的最优解。

矩阵用二维数组 K\[i]\[j] 表示，i ∈ [0, …, n]，j ∈  [0, …, capacity]。

```
function dKnapsack(capacity, n, size, value){
	var K = []; // 声明矩阵并初始化
	for(var i = 0; i <= n; i++) 
		K[i] = [];
    
    for(var i = 0; i <= n; i++){
    	for(var j = 0; j <= capacity; j++){
        	if(i == 0 || j == 0){
            	K[i][j] = 0; 
        	}else if(j < size[i-1]){
            	K[i][j] = K[i-1][j]	
        	}else{
            	var a = K[i-1][j],
            		b = K[i-1][j-size[i-1]]+value[i-1];
            	K[i][j] = a > b ? a : b;
        	}
    	}
    }
    
    //console.log(K)
    return K[n][capacity]
}
```

<br />



# 2. 贪心算法

> “A greedy algorithm  looks for “good solutions”,  called **local optima**, which will hopefully lead to the correct final solution, called the **global optimum**. ”

* **思想**：贪心算法不从整体最优考虑，它只作出**局部最优**选择。即使贪心算法不能得到整体最优解，其最终结果却是最优解的很好近似。
* **贪心算法**与**动态规划算法**的区别：**动态规划算法**通常自底向上的求解各子问题；而**贪心算法**则通常自顶向下进行，迭代的作出贪心选择，每次贪心选择就将所求问题简化为规模更小的子问题。



## 2.1 硬币问题

硬币问题 (Coin-changing Problem)： 现有面值25美分、10美分和1美分三种面值硬币，要将63美分换成零钱，按照贪心算法该怎么换？

```
function makeChange(origAmt){
	var remainAmt = 0;
	if (origAmt % 25 < origAmt) {
		console.log("25-cent coins: ", parseInt(origAmt / 25));
    	origAmt = origAmt % 25
	}
	if (origAmt % 10 < origAmt) {
		console.log("10-cent coins: ", parseInt(origAmt / 10));
    	origAmt = origAmt % 10
	}
	console.log("1-cent coins: ", parseInt(origAmt / 1));
}
```



## 2.2 部分背包问题

部分背包问题 (Fractional Knapsack Problem)：按照贪心算法，每次选**性价比最高**的物品放入背包。

算法步骤：

1. 背包拥有容量 `capacity` ， 物品拥有价值 `value` 和大小`size`。
2. Rank items by `value/size` ratio.
3. Consider items in terms of decreasing ratio.
4. Take as much of each item as possible.

```
function ksack(capacity, n, size, value){
	// 冒泡排序, value/size递减
	for(var i = 0; i < n; i++){
    	for(var j = 0; j < n-i-1; j++){
        	if(value[j]/size[j] < value[j+1]/size[j+1]){
            	var tmp = value[j];
            	value[j] =  value[j+1];
            	value[j+1] = tmp;
            	tmp = size[j];
            	size[j] =  size[j+1];
            	size[j+1] = tmp;
        	} 
		} 
	} 
	//console.log({value, size})
	
	var load = 0, result = 0;
	i = 0;
	while(load < capacity && i < n){
    	if(size[i] <= capacity - load){
        	load += size[i];
        	result += value[i];
      	}
      	i++;
	}
	return result;
}
```

