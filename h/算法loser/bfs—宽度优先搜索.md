# bfs—宽度优先搜索

#### 定义

​		宽度优先搜索算法（又称广度优先搜索）是最简便的图的搜索算法之一，这一算法也是很多重要的图的算法的原型。Dijkstra单源最短路径算法和Prim最小生成树算法都采用了和宽度优先搜索类似的思想。其别名又叫BFS，属于一种盲目搜寻法，目的是系统地展开并检查图中的所有节点，以找寻结果。换句话说，它并不考虑结果的可能位置，彻底地搜索整张图，直到找到结果为止。
​		**它是一种盲目搜寻法，目的是系统地展开并检查图中的所有节点，以找寻结果。并不考虑结果的可能位置，彻底地搜索整张图，直到找到结果为止。**

```
function bfs(graph, start) {
	let visited = new Set();
	let queue = [start];

  	while (queue.length > 0) {
    	let node = queue.shift();
    	if (!visited.has(node)) {
     		 visited.add(node);
     		 console.log(node);
      		queue.push(...graph[node]);
    	}
 	}
}

// 示例图结构
let graph = {
  	A: ['B', 'C'],
  	B: ['A', 'D', 'E'],
  	C: ['A', 'F'],
  	D: ['B'],
  	E: ['B', 'F'],
  	F: ['C', 'E'],
};

// 调用广度优先搜索函数
bfs(graph, 'A');
```

