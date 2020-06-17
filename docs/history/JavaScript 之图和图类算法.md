> 复习一下图的概念和基础的算法思想，以及如何用 JavaScript 来展示图和实现图的简单算法。

<br />



# 0. 概念

* **图 (Graph)** ：G = ( V, E), 其中V代表顶点Vertex，E代表边Edge，一条边就是一个顶点对 (u, v)。图分为***无向图 (Unordered Graph)*** 和***有向图 (Directed Graph)***。 

```
// 定义顶点
function Vertex(label) {
   this.label = label;
   this.wasVisited = false;
}
// 本文只介绍图的算法思想，用简单数组表示顶点集合，所以此处顶点定义用不着。
```

* **权值**： 一个顶点到另一个顶点的“代价”。若不联通，则认为无限大。
* 图的数据结构可用**邻接表 (Adjacency List) **或**邻接矩阵 (Adjacency Matrix) **表示。
* **邻接表**，可表示为以顶点为key值的二维数组

```
 // adjacency list
 0 -> [1, 2]
 1 -> [0, 3]
 2 -> [0, 4]
 3 -> [1]
 4 -> [2]
```

* **邻接矩阵**，可表示为|V|*|V|的矩阵 Matrix。若(u, v)联通，则 Matrix\[u][v] = 1；若边有加权，则 Matrix\[u][v] = 权值。

<br />



# 1. 创建图

```
function Graph(v){
		this.vertices = v;
    this.edges = 0;
    this.adj = [];
  	for(var i = 0; i < this.vertices; i++){
    	this.adj[i] = [];
  	}
  	this.addEdge = addEdge;
  	this.showGraph = showGraph;
}

function addEdge(u, v){
  	this.adj[u].push(v);
  	this.adj[v].push(u);
  	this.edges++;
}

function showGraph(){
	for(var i = 0; i < this.vertices; i++){
    	var str = i + " -> ";
      	for (var j = 0; j < this.vertices; j++) {
        	if(this.adj[i][j] != undefined)
        		str += this.adj[i][j] + ' ';
      	}
      	console.log(str);
	}
}
```

<br />



# 2. 图遍历

图的遍历算法有两种：**深度优先遍历 (Depth-First Search)** 和**广度优先遍历 (Breadth-First Search)** 。

图遍历中有一个**已访问顶点**的概念，因此重新定义一下图的数据结构：

```
function Graph(v) {
   this.vertices = v;
   this.edges = 0;
   this.adj = [];
   for (var i = 0; i < this.vertices; ++i) {
      this.adj[i] = [];
   }
   this.addEdge = addEdge;  // 不赘述
   this.showGraph = showGraph;  // 不赘述
   this.dfs = dfs;
   this.marked = []; // 记录已访问过的顶点
   for (var i = 0; i < this.vertices; ++i) {
      this.marked[i] = false;
   }
}
```

* **深度优先遍历算法**：
  1. 从开始顶点搜索一条路径，直到无路可走；
  2. 回退到上一顶点，往下继续遍历未访问过的顶点；
  3. 依次类推，直到所有的顶点都已经访问过。

```
// 深度优先遍历, 参数v为开始顶点
function dfs(v){
	this.marked[v] = true;
	
	if(this.adj[v]){
    	console.log("Visited vertext: "+v) 
	}

	this.adj[v].forEach(w => {
		if(!this.marked[w]) this.dfs(w)
	})
}
```

* **广度优先遍历算法**：
  1. 取当前节点的一个未被访问的邻节点，标记为已访问，加入队列；
  2. 取下一个节点 v，标记为已访问；
  3. 将 v 邻节点中所有未访问的节点加入队列。

```
// 广度优先遍历, 参数v为开始顶点
function bfs(s){
	var queue = [];
	this.marked[s] = true;
	queue.push(s);
	while(queue.length > 0){
		var v = queue.shift();
		console.log("Visited vertext: "+v) 
		this.adj[v].forEach(w => {
			if(!this.marked[w]){
				this.marked[w] = true;
				queue.push(w)
			}
		})
	}
}
```

<br />



# 3. 图的最短路径



### 3.1 广度优先遍历，获取无权最短路径

>  为什么说**广度优先搜索**可以用来求**无权最短路径**呢？

因为，广度优先搜索每次都会先发现距离s为k的所有顶点，然后才会发现距离s为k+1的所有顶点。 s为起始点。

```
// 修改 Graph 类 
this.edgeTo = []; //记录从一个到另一个顶点的边

// 修改 bfs 函数
function bfs(s){
	var queue = [];
	this.marked[s] = true;
	queue.push(s);
	while(queue.length > 0){
		var v = queue.shift();
		console.log("Visited vertext: "+v) 
		this.adj[v].forEach(w => {
			if(!this.marked[w]){
				this.edgeTo[w] = v;  // 增加这个步骤
				this.marked[w] = true;
				queue.push(w)
			}
		})
	}
}
```

执行`this.dfs(0)`后，顶点0到其它各顶点的路径已经记录在`this.edgeTo`内。

```
// 假设起点为0，v为目标顶点
function pathTo(v) {
	var source = 0; 
  
  	if(!this.marked[v]) // 若v未被访问过，则说明从0到v无路可走
  		return undefined;
  	
  	var path = [];
  	for(var i = v; i != source; i = this.edgeTo[i])
  		path.unshift(i);
  	path.unshift(source);
  	
  	return path;
}
```



### 3.2 Dijkstra算法

Dijkstra算法是典型的单源最短路径算法，用于计算**带权有向图**一个节点到其他所有节点的最短路径。

算法步骤：

1. G = (V,E) 是一个带权有向图，图中顶点两组，S 和 U。
2. 初始，S 只包含源点，即S＝{ v }，v 的距离为0；U 包含除v外其他顶点。若v与U中顶点u有边，则 <u,v> 正常有权值，若u不是v的出边邻接点，则 <u,v> 权值为∞。
3. 从 U 中选取一个距离 v 最小的顶点 k，把 k 加入 S （该距离就是 v 到 k 的最短路径长度）。
4. 以 k 为中间点，修改 U 中各顶点的距离。若从源点v到顶点u的距离（经过k）比原来距离（不经过k）短，则修改顶点 u 的距离值。
5. 重复步骤 3 和 4 ，直到所有顶点都包含在S中。

<br />



# 4. 拓扑排序

定义：将**有向无环图**的顶点以线性方式排序。即对于任何连接自顶点u到顶点v的有向边uv，在最后的排序结果中，顶点u总是在顶点v的前面。

> “The algorithm for *topological sorting* is similar to the algorithm for *depth-first search*.”

```
function topSort() {
	var stack = [];
   	var visited = [];
   	for (var i = 0; i < this.vertices; i++)
   		visited[i] = false;
   
   	for (var i = 0; i < this.vertices; i++) {
   		if (visited[i] == false) {
        	this.topSortHelper(i, visited, stack);
      	}
   	}

   console.log({ stack })  // 记录了top排序之后的结果
}

function topSortHelper(v, visited, stack) {
   visited[v] = true;
   this.adj[v].forEach(w => {
   		if (!visited[w]) {
         	this.topSortHelper(w, visited, stack);
      	}
   	})
   stack.unshift(v);
} 
```

<br />



# 5. 附录

完整代码如下：

```
function Graph(v) {
   this.vertices = v;
   this.edges = 0;
   this.adj = [];
   this.edgeTo = [];
   for (var i = 0; i < this.vertices; ++i) {
      this.adj[i] = [];
   }
   this.addEdge = addEdge;  // 不赘述
   this.showGraph = showGraph;  // 不赘述
   this.dfs = dfs;
   this.bfs = bfs;
   this.pathTo = pathTo;
   this.marked = []; // 记录已访问过的顶点
   for (var i = 0; i < this.vertices; ++i) {
      this.marked[i] = false;
   }
   this.topSort = topSort;
   this.topSortHelper = topSortHelper;
}


function addEdge(u, v){
  	this.adj[u].push(v);
  	this.adj[v].push(u);
  	this.edges++;
}


function showGraph(){
	for(var i = 0; i < this.vertices; i++){
    	var str = i + " -> ";
      	for (var j = 0; j < this.vertices; j++) {
        	if(this.adj[i][j] != undefined)
        		str += this.adj[i][j] + ' ';
      	}
      	console.log(str);
	}
}


function dfs(v){
	this.marked[v] = true;
	
	if(this.adj[v]){
    	console.log("Visited vertext: "+v) 
	}

	this.adj[v].forEach(w => {
		if(!this.marked[w]) this.dfs(w)
	})
}


function bfs(s){
	var queue = [];
	this.marked[s] = true;
	queue.push(s);
	while(queue.length > 0){
		var v = queue.shift();
		console.log("Visited vertext: "+v) 
		this.adj[v].forEach(w => {
			if(!this.marked[w]){
				this.edgeTo[w] = v;
				this.marked[w] = true;
				queue.push(w)
			}
		})
	}
}


function pathTo(v) {
	var source = 0; 
  
  	if(!this.marked[v]) // 若v未被访问过，则说明从0到v无路可走
  		return undefined;
  	
  	var path = [];
  	for(var i = v; i != source; i = this.edgeTo[i])
  		path.unshift(i);
  	path.unshift(source);
  	
  	console.log({ path })
  	return path;
}


function topSort() {
   var stack = [];
   var visited = [];
   for (var i = 0; i < this.vertices; i++) {
      visited[i] = false;
   }
   for (var i = 0; i < this.vertices; i++) {
      if (visited[i] == false) {
         this.topSortHelper(i, visited, stack);
      }
   }

   console.log({ stack })
}


function topSortHelper(v, visited, stack) {
   visited[v] = true;
   this.adj[v].forEach(w => {
   		if (!visited[w]) {
         	this.topSortHelper(w, visited, stack);
      	}
   	})
   stack.unshift(v);
} 


g = new Graph(6);
g.addEdge(1,2);
g.addEdge(2,5);
g.addEdge(1,3);
g.addEdge(1,4);
g.addEdge(0,1);
g.showGraph();
// g.dfs(0);
g.bfs(0);
g.pathTo(5)
g.topSort();
```























