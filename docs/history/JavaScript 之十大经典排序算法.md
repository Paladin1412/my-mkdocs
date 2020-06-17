## 0.概念

十大排序算法通常是指**内部排序**，即所有数据记录都是在内存中进行排序。常见的内部排序算法有：插入排序、希尔排序、选择排序、冒泡排序、归并排序、快速排序、堆排序、基数排序等。

其中，

In-place sort（不占用额外内存或占用常数的内存）：插入排序、选择排序、冒泡排序、堆排序、快速排序。

Out-place sort：归并排序、计数排序、基数排序、桶排序。

![1](http://47.94.165.69:3031/upload/aaa/50b075ad3d154.png)



## 1.冒泡排序

思想：比较相邻两个元素，如果它们的顺序错误就把它们交换过来。

```
function bubbleSort( arr ){
	let len = arr.length;
	for( let i = 0; i < len-1; i++ ){
		for( let j = 0; j < len-1-i; j++){
          if(arr[j] > arr[j+1]){
            let tmp = arr[j];
            arr[j] = arr[j+1];
            arr[j+1] = tmp;
          }
		}
	}
    return arr
}
```



## 2.选择排序

思想：每次找一个最小值。

```
function selectionSort( arr ){
  let len = arr.length;
  for( let i = 0; i < len; i++ ){
    let minIdx = i;
    for( let j = i+1; j < len; j++ ){
      if(arr[j] < arr[minIdx]){
        minIdx = j;
      }
    }
    //swap
    let tmp = arr[i];
    arr[i] = arr[minIdx];
    arr[minIdx] = tmp;
  }
  return arr;
}
```



## 3.插入排序

思想：对未排序的数据，在已排序的序列中从后向前扫描，找到相应的位置并插入。

```
function insertionSort( arr ){
  let len = arr.length;
  for(let i = 1; i < len; i++ ){
    let key = arr[i];
    for( var j = i-1; j >= 0 && key < arr[j]; j--){
      arr[j+1] = arr[j];
    }
    arr[j+1] = key;
    //console.log(i, key, j, arr)
  }
  return arr;
}

//var x = [7,6,5,4,3,2,1]
//console.log(insertionSort(x))
```



## 4. 希尔排序（Shell Sort）

概念：希尔排序是**插入排序**的一种更高效率的实现，核心在于**增量**的设置。

思想： 希尔排序是把记录按下标的一定增量分组，对每组使用直接插入排序算法排序；随着增量逐渐减少，当增量减至1时，所有记录被分成一组，算法终止。

实现：示例采用**希尔增量**，选择增量gap=length/2，继续缩小增量gap = gap/2，这种增量选择可以用一个序列来表示，{n/2, (n/2)/2…1}，称为**增量序列**。

```
function shellSort( arr ){
  let len = arr.length,
  	  gap = Math.floor(len/2);
  
  for(; gap > 0; gap = Math.floor(gap/2)){
  	//循环内是一个插入排序
    for(let i = gap; i < len; i++){
      let key = arr[i];
      for( var j=i-gap; j>=0 && key<arr[j]; j-=gap){
      	arr[j+gap] = arr[j];
      }
      arr[j+gap] = key;
      //console.log(gap, i, key, j, arr)
    }
  }
  
  return arr;
}

//var x = [7,6,5,4,3,2,1]
//console.log(shellSort(x))
```



## 5. 归并排序（Merge Sort）

思想：将已有序的子序列合并，得到一个完全有序的序列。

实现：示例为**二路归并排序**。

```
function mergeSort( arr ){
  //console.log('1...',arr);
  let len = arr.length;
  if(len < 2)
  	  return arr;
  
  let middle = Math.floor(len/2),
  	  left = arr.slice(0, middle),
  	  right = arr.slice(middle);
  	  
  return  merge(mergeSort(left), mergeSort(right));;
}

//合并两个有序数组
function merge( left, right ){
	let result = [];
	let i=0, j=0;
	while( i<left.length && j<right.length){
		if(left[i] < right[j])
			result.push(left[i++]);
		else
			result.push(right[j++]);
	}
	
	while(i<left.length)
		result.push(left[i++]);
		
	while(j<right.length)
		result.push(right[j++]);
	
	//console.log('2...',result);
	return result
}


var x = [7,6,5,4,3,2,1]
console.log(mergeSort(x))
```



## 6.快速排序

思想：挑出一个元素作为 “**基准**”（pivot），一次排序进行**分区**，所有小于基准的元素居左，所有大于基准的元素居右，对分区进行**递归**。

```
/* arguments 依次是 arr, left, right */
function quickSort( arr ){
	
	let left = arguments[1] || 0,
		right = isNaN(arguments[2])? arr.length - 1 : arguments[2];
		
	if(right-left < 1) return arr;
	
	let pivotIdx = left, pivot = arr[left];
	
	for(let i = left+1; i <= right; i++){
      if(arr[i] < pivot){
        let tmp = arr[++pivotIdx];
        arr[pivotIdx] = arr[i];
        arr[i] = tmp;
      }
	}
	arr[left] = arr[pivotIdx];
	arr[pivotIdx] = pivot;

	quickSort(arr, left, pivotIdx-1);
	quickSort(arr, pivotIdx+1, right)
	
	return arr;
}
```



## 7.堆排序

思想： 创建**大根堆**，即每个节点的值都大于其子节点，用于升序排列。排序时取出堆顶(最大值)，放在队尾，剩下的元素调整为大根堆。

```
function heapSort( arr ){
	for(let len = arr.length; len > 0; len--){
      heap(arr, 0, len)
      let tmp = arr[0];
      arr[0] = arr[len-1];
      arr[len-1] = tmp;
      //console.warn(arr);
	}
	return arr;
}

//建立大根堆，i为堆顶位置， length为元素个数
function heap( arr, i, length){
	var left = i*2+1, right = i*2+2, max = i;
	
	if(left < length){
      heap( arr, left, length);
      if(arr[left] > arr[max]) max = left;
	}
	
	if(right < length){
      heap( arr, right, length);
      if(arr[right] > arr[max] ) max = right;
	}
    
	if(max !== i){
      let tmp = arr[i];
      arr[i] = arr[max];
      arr[max] = tmp;
      heap( arr, max, length)
      //console.log( arr, i, length )
	}		
}

```



## 8. 计数排序

思想：将元素value转化为key，存在新数组key对应位置。因此前提，数值有范围，且为整数。

```
function countingSort( arr ){
	let len = arr.length, max = 0, tmpArr=[];
	
	for(let i = 0; i < len; i++ ){
      if(arr[i] > max) max = arr[i];
      if(!tmpArr[arr[i]]) 
      	tmpArr[arr[i]] = 1;
      else
      	tmpArr[arr[i]]++;
	}
	console.log(tmpArr);
	
	let k = 0;
	for(let i = 0; i <= max; i++){
      while(tmpArr[i] > 0){
        arr[k++] = i;
        tmpArr[i]--;
      }
      console.log(arr);
	}
	
	return arr;
}


var x = [7,6,5,4,3,2,1]
console.log(countingSort(x))
```

## 9. 桶排序

思想：计数排序的升级版，将输入的N个数据均匀的分配到K个桶中。

 ```
function bucketSort( arr ){
  const BUCKET_SIZE = 5; //default set to 5
  let min, max;
  for(let i = 0; i<arr.length; i++){
    min = arr[i] > min ? min : arr[i];
    max = arr[i] < max ? max : arr[i];
  }
  
  let bucketCount = Math.floor((max - min) / BUCKET_SIZE) + 1;
  let buckets = []; //二维数组桶的初始化
  for (let i = 0; i < bucketCount; i++) buckets[i] = [];
  
  //console.log({min, max, buckets, bucketCount});
  
  //利用映射函数将数据分配到各个桶中
  for(i = 0; i < arr.length; i++){
  	var bucketIdx = Math.floor((arr[i] - min) / BUCKET_SIZE); //映射函数
    buckets[bucketIdx].push(arr[i]);
    for(let j = buckets[bucketIdx].length-1; j > 0;j--){
   	  if(buckets[bucketIdx][j]<buckets[bucketIdx][j-1]){
         let tmp = buckets[bucketIdx][j];
         buckets[bucketIdx][j] = buckets[bucketIdx][j-1];
         buckets[bucketIdx][j-1] = tmp;
      }
    }
  }
   
   arr = [];
   for( let i = 0; i<bucketCount; i++){
     arr = arr.concat(buckets[i]);
   }
   
   return arr;
}
 ```



## 10. 基数排序

思想： 利用了桶的概念，根据键值的每位数字来分配桶。

实现：基数排序LSD是按照低位先排序，然后收集；再按照高位排序，然后再收集；依次类推，直到最高位。

```
function radixSort(arr) {
    let max, maxDigit, mod = 1;
    for(let i = 0; i<arr.length; i++){
    	max = arr[i] < max ? max : arr[i];
  	}
  	maxDigit = max.toString().length;
    
    for (let i = 0; i < maxDigit; i++, mod *= 10) {
    	let buckets = [];
        for(let j = 0; j < arr.length; j++) {
            let bucketIdx = parseInt(arr[j]/mod)%10;
            if(!buckets[bucketIdx]) buckets[bucketIdx] = [];
            buckets[bucketIdx].push(arr[j]);
        }
       
        arr = [];
        for( let j = 0; j<buckets.length; j++){
        	arr = arr.concat(buckets[j]);
        	arr = arr.filter(item => item!==undefined)
        }
        //console.log({ maxDigit, buckets, arr })
    }
    
    return arr;
}

//var x = [7,6,5,3,2,1]
//console.log(radixSort(x))
```

