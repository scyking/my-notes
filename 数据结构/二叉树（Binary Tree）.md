# 二叉树（Binary Tree）

标签（空格分隔）： 数据结构

---

[toc]

## 数据结构

	public class BinaryTreeNode<E> ｛		
		E val;
		BinaryTreeNode<E> left = null;
		BinaryTreeNode<E> right = null;
		
		BinaryTreeNode<E>(E val){
			this.val = val;
		}
	｝

## 遍历算法

递归方式：

- 前序遍历

		public void preBinaryTreeOrder(TreeNode root) {
			if (null != root) {
				System.out.println(root.val);
				preBinaryTreeOrder(root.left);
				preBinaryTreeOrder(root.right);
			}
		}

- 中序遍历
		
		public void inBinaryTreeOrder(TreeNode root) {
			if (null != root) {
				inBinaryTreeOrder(root.left);
				System.out.println(root.val);
				inBinaryTreeOrder(root.right);
			}
		}

- 后序遍历  

		public void postBinaryTreeOrder(TreeNode root) {
			if (null != root) {
				postBinaryTreeOrder(root.left);
				postBinaryTreeOrder(root.right);
				System.out.println(root.val);
			}
		}

## 常见特殊二叉树

- 满二叉树

	除最后一层无任何子节点外，每一层上的所有结点都有两个子结点二叉树。 

		/*
		*  			A
	 	* 		   / \
	 	*         /   \
	 	*        B     C
	 	*       / \   / \
	 	*      /   \ /   \
	 	*     D    E F    G 
	 	*    
	 	*/

- 完全二叉树

	一棵二叉树至多只有最下面的一层上的结点的度数可以小于2，并且最下层上的结点都集中在该层最左边的若干位置上，则此二叉树成为完全二叉树。 

		/*  
	 	*          A            
	 	*         / \           
	 	*        /   \       
	 	*       B     C     
	 	*      / \            
	 	*     /   \          
	 	*    D     E      
	 	*
	 	*/

- 平衡二叉树（AVL）

	是一棵空树或左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一棵平衡二叉树。

		/*  
	 	*          A            
		*         / \           
		*        /   \       
	 	*       B     C     
	 	*      / \     \      
	 	*     /   \     \      
	 	*    D     E     F 
	 	*         / 
	 	*        /
	 	*       H
	 	*
	 	*/ 

- 二叉搜索/排序树

	是一棵空树或者是具有下列性质的二叉树： 
		
	- 若它的左子树不空，则左子树上所有结点的值均小于它的根结点的值； 
	- 若它的右子树不空，则右子树上所有结点的值均大于它的根结点的值； 
	- 它的左、右子树也分别为二叉排序树 

			/*  
	 		*           6                
	 		*          / \             
	 		*         /   \            
	 		*        3     8          
	 		*       /     / \         
	 		*      /     /   \       
	 		*     1     7     9     
	 		*      
	 		*/

	PS：中序遍历，结果是有序数组。

- 红黑树

	平衡二叉搜索树

		/*  
	 	*           6
	 	*          / \                
	 	*         /   \             
	 	*        /     \            
	 	*       2       8          
	 	*      / \     / \         
	 	*     /   \   /   \       
	 	*    1     4 7     9     
	 	*         / \
	 	*        /   \    
	 	*       3     5
	 	*/

	特性：

	- 每个节点或者是黑色，或者是红色。
	- 根节点是黑色。
	- 每个叶子节点（NIL）是黑色。 [注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点！]
	- 如果一个节点是红色的，则它的子节点必须是黑色的。
	- 从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。

- 哈夫曼树（Huffman树，最优二叉树）

	给定n个权值作为n个叶子结点，构造成带权路径长度达到最小的二叉树。

	例，[1,3,5,7,9]构造的哈夫曼树可能如下（不唯一）：


		/*  
	 	*           O
	 	*          / \                
	 	*         /   \             
	 	*        /     \            
	 	*       O       O          
	 	*      / \     / \         
	 	*     /   \   /   \       
	 	*    5     O 7     9     
	 	*         / \
	 	*        /   \    
	 	*       1     3
	 	*/

	**哈夫曼编码**：假设结点到左子结点的路径标记为0，到右子结点的路径标记为1，则根节点到叶子结点的路径标记顺序为该叶子结点的哈夫曼编码。



