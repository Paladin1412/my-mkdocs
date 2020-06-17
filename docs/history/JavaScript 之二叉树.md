> 树是常用的数据结构，二叉树（Binary Trees）和二叉搜索树（Binary Search Trees）常优于其它数据结构，方便快速查找和增删数据等操作。

<br />

# 0. 概念

* **树**： “ A tree is made up of a set of nodes connected by edges.”
* **二叉树**： “ Binary trees restrict the number of child nodes to no more than two.”
* **二叉查找树（BST）**：“ A **binary search tree** is a binary tree in which data with lesser values are stored in left nodes and data with greater values are stored in right nodes.”

<br />



# 1. 创建 BST

```
// define Binary Tree Nodes
function Node(data, left, right) {
   this.data = data;
   this.left = left;
   this.right = right;
   this.show = show;
}

function show() {
   return this.data;
}
```

```
// build BST
function BST(){
  this.root = null;
  this.insert = insert;
  this.inOrder = inOrder;
}

function insert(data){
  var n = new Node(data, null, null);
  if(this.root == null){
    this.root = n;
  }else{
  	var current = this.root;
  	var parent;
    while(true){
      parent = current;	
      if(data < current.data){
        current = parent.left;
        if(current == null){
          parent.left = n;
          break;
        }
      }else{
        current = parent.right;
        if(current == null){
          parent.right = n;
          break;
        }
      }
    }
  }
}
```

<br />



# 2. 遍历 BST

遍历BST方法有三： 中序遍历，先序遍历和后序遍历。

+ **中序遍历（inorder）**： “An inorder traversal visits all of the nodes of a BST in ascending order of the node key values. ”

  >  即 left -> root -> right

+ **先序遍历（preorder）**:“ A preorder traversal visits the root node first, followed by the nodes in the subtrees under the left child of the root node, followed by the nodes in the subtrees under the right child of the root node.”

  > 即 root -> left -> right

+ **后序遍历（postorder）**:“A postorder traversal visits all of the child nodes of the left subtree up to the root node, and then visits all of the child nodes of the right subtree up to the root node.” 

  > 即 left -> right -> root

```
// 中序遍历
function inOder(node){
 if(node !== null){
   inOder(node.left);
   console.log(node.show());
   inOder(node.right);
 } 
}
```

```
// 先序遍历
function preOrder(){
  if(node !== null){
   	console.log(node.show());
   	preOrder(node.left);
    preOrder(node.right);
  } 
}
```

```
// 后序遍历
function postOrder(){
  if(node !== null){
   	postOrder(node.left);
    postOrder(node.right);
    console.log(node.show());
  } 
}
```

<br />



# 3. 从 BST 中删除节点

算法思想：

1. 若为叶节点，即无孩子，其父指向它的指针设为 null；
2. 若一个孩子，其父指向它的指针改指向其孩子；
3. 若俩孩子，首先，要么找到其左子树的最大值，要么找到其右子树的最小值。本文采用后者，后续步骤转到4；
4. 将该节点值替换为最小值，**递归**地删除右子树中最小值节点。

```
// 参数为被删节点的value
function remove(data) {
   root = removeNode(this.root, data);
}

function removeNode(node, data){
	if(node == null) 
		return null;
		
	if(data == node.data){
		// 无孩子
		if(node.left == null && node.right == null) 
			return null;
      	
      	// 无左孩子
     	if(node.left == null) 
     		return node.right;
     	
     	// 无右孩子	
     	if(node.right == null) 
     		return node.left;
     	
     	// 有俩孩子
     	var tmp = getSmallestValue(node.right); //自定义：右子树最小值
      	node.data = tmp;
      	node.right = removeNode(node.right, tmp);
      	return node;
      	
	}else if(data < node.data){
      node.left = removeNode(node.left, data);
      return node;
	}else{
      node.right = removeNode(node.right, data);
      return node;
	}
}
```













