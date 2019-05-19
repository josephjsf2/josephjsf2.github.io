---
title: 資料結構與演算法：Binary Search Tree 二元搜索樹
layout: post
categories: data structure and algorithm
tags:
- tree
- binary search tree
- BST
- data structure

---

記錄Binary Search Tree學習筆記。

<!--more-->

# Binary Tree 二元樹

<img src="https://imgur.com/E1XJ4jK.png" />

+ 每個Node最多只能有2個 child node
+ Level i 最大節點數： 2^(i-1), i>0，如level 3的Node數最多為 2^(3-1) = 4，如上圖之 DEFG
+ Depth k 最大Node數： 2^k -1, k>0，如上上面 depth 4的數最大可能的Node數量為 2^4 -1 ＝15

## 二元樹實做方式：

#### 1. Array
以陣列實做時，可以透過下面規則實做：

位於陣列第 i 個位置的node，則
1. root node 從 1開始，陣列 index 0 不放任何資訊
2. parent node 會位於 Floor(i/2) 的位置
3. 左邊子節點會位於 2i 位置
4. 右邊子節點會位於 2i +1 位置

如下圖， E的 index為5， left child為 10，right child為11。而H與I的parent node index 為 Floor(10/2) 與 Floor(11/2) 皆為5。

優點是程式運行速度快，可以快速定位到node
缺點是相當浪費空間，若非完美二元樹，則陣列中會存在許多空間未被使用。如果元素數量超過陣列大小時，重建陣列，會較為耗時。

<img src="https://imgur.com/XuP8OdY.png" />

#### 2. Linked List 

首先會定義 abstract data type：

```java
class Node<T>{
	Node leftNode;
	Node rightNode;
	T value;
}
```

透過leftNode與 rightNode關連 child node

程式需要記錄 root node，作為所有動作的起始位置。

# Binary Search Tree 二元搜索樹

## 1. 概念

1. left node會存放所有比當前node value小之 node；right node會存放所有比當前node value大之node。如果有重複的值，可自行決定要放左或右。
2. 建立起來之binary search tree(簡稱BST)，會是排序完成之狀態。
3. 搜尋時間會是O(LogN)，比起以往透過陣列與List之O(N) 會快上許多，如果二元樹愈平衡，效能愈佳。

如下列BST，35是 BST中的root node，可以注意到 35的右邊所有node 的value都比35大；左邊node value都比35小。後面所有subtree 也都有這個特性。

<img src="https://imgur.com/I1Ahpqe.png" />

  

時間複雜度：

|            |  Average    |   Worst   |
| ---------- | ---- | ---- |
| Insert | O(LogN) | O(N) |
|Delete|O(LogN)|O(N)|
|Search|(LogN)|O(N)|

## 2. 實做Binary Search Tree

首先先定義 TreeNode type：

這邊會要求傳入的Generic type必須實做 Comparable介面，目的是可以讓 binary search tree針對不同類別可以做到比較大小的目的。如20與 44可都是integer可以比較大小，但Person類別就需要實做Comparable介面才有辦法讓不同Person類別比較大小。

```java
public class TreeNode<T extends Comparable<T>> {

    private T data;
    private TreeNode leftNode;
    private TreeNode rightNode;

    public TreeNode(T data) {
        this.data = data;
    }
  // getter and setter
```

接著定義 Tree interface

```java
public interface Tree<T extends Comparable<T>> {

    public TreeNode<T> getRoot();

    public TreeNode<T> findElement(T value);

    public void insertElement(TreeNode<T> newNode);

    public void traverse();

    public void deleteElement(T value);

    public boolean compare(Tree<T> tree);

    public TreeNode<T> getNodeByRank(int rank);
}
```

實做tree的類別 BinarySearchTree

```java
public class BinarySearchTree<T extends Comparable<T>> implements Tree<T> {

    private TreeNode<T> root;

    @Override
    public TreeNode<T> getRoot() {
        return this.root;
    }

    @Override
    public TreeNode<T> findElement(T value) {
			return null;
    }
		//   ... etc

```



### 1. 查詢

透過遞迴可以容易的實現，概念如下：

1. 如果要找的value比當前node的value小，則往左邊node找

2. 如果要找的value比當前node的value大，則往右邊node找

```java
    @Override
    public TreeNode<T> findElement(T value) {
        if(this.getRoot() == null)
            return null;

        return this.findElement(this.getRoot(), value);
    }

    private TreeNode<T> findElement(TreeNode<T> currentNode, T value){
        if(currentNode.getData().compareTo(value) ==0)
            return currentNode;
        else if(currentNode.getData().compareTo(value) <0)
            return this.findElement(currentNode.getLeftNode(), value);
        else
            return this.findElement(currentNode.getRightNode(), value);
    }
```



### 2. 新增

在BST中新增element 概念如下：

1. 若要新增的node value比現在的node value小，則往left node繼續找可新增位置
2. 若要新增的node value比現在的node value大，則往right node繼續找可新增位置
3. 如果要找的left node或right node是空的，則新增node於該位置

如今要新增一個value為 41的Node，則步驟如下：

1. 41與35做比較，41大於35，故往35的右邊node繼續執行
2. 41比44做比較，41小於44，故往44的左邊node繼續執行
3. 41與40做比較，41大於40，故往40的右邊node繼續執行，但因為40右邊node是空的，所以會將41新增於 40的 right node

<img src="https://imgur.com/qUoXNbJ.png" />

```java
    @Override
    public void insertElement(TreeNode<T> newNode) {

        if(this.getRoot() == null)
            this.root = newNode;
        else
            this.insertElement(this.getRoot(), newNode);
    }

    private void insertElement(TreeNode<T> currentNode, TreeNode<T> newNode){

        if(newNode.getData().compareTo((currentNode.getData()))>=0){
            if(currentNode.getRightNode()!=null)
                this.insertElement(currentNode.getRightNode(), newNode);
            else
                currentNode.setRightNode(newNode);
        }else if(newNode.getData().compareTo(currentNode.getData())<0){

            if(currentNode.getLeftNode() !=null)
                this.insertElement(currentNode.getLeftNode(), newNode);
            else
                currentNode.setLeftNode(newNode);
        }
    }
```



### 3. 刪除

刪除共有三種情境：

1. 當要刪除的node處於最尾端時，直接刪除

<img src="https://imgur.com/8fkefhr.png" />

​	如果要刪除 40 ，只需要將 44 的 left node 設為 null 即可。

2. 當要刪除的node的 left node或right node一邊有資料時，要用child node取代目前node 位置

   <img src="https://imgur.com/dCnFOdw.png" />

   如果要刪除44，則只要將56取代掉44的位置即可。

   

3. 當要刪除的node left與right node皆有資料時，必須找 left node 下最大的value，或是 right node下最小的value。

   將找到的value與要刪除的value node交換，交換後，再重新執行一次刪除動作即可，直接搭配圖片來看：

<img src="https://imgur.com/HfsttRH.png" />

​	如果要刪除35，則可以找左側value最大的node，或是右側最小的node

<img src="https://imgur.com/Ha7brqN.png" />

​	左側最大為 32，右側做小為56，這邊選擇32 取代35

<img src="https://imgur.com/5qevLWX.png" />

實際上只是將原本35 node value與 32 node value 交換，交換完成後，則刪除 32 左側node下之 35即可。

### 4. Traversal

以下面這張二元樹來看，可以有幾種遍歷方式：

<img src="https://imgur.com/SGPS6QL.png" />

1. Pre-Order 前序遍歷
  遍歷順序為 root -> left -> right
  依序為+ * * / A B C D E

  ```java
      @Override
      public void traverse() {
          this.traverse(this.getRoot());
      }
  
      private void traverse(TreeNode currentNode){
          if(currentNode == null)
              return;
          // Traversing in pre-order
  	      System.out.println(currentNode);
          this.traverse(currentNode.getLeftNode());
          this.traverse(currentNode.getRightNode());
      }
  ```

  

2. In-Order  中序遍歷
  遍歷順序為 left -> root ->  right
  A / B * C * D + E 

  ```java
      @Override
      public void traverse() {
          this.traverse(this.getRoot());
      }
  
      private void traverse(TreeNode currentNode){
          if(currentNode == null)
              return;
          // Traversing in in-order
          this.traverse(currentNode.getLeftNode());
          System.out.println(currentNode);
          this.traverse(currentNode.getRightNode());
      }
  ```

  

3. Post-Order  後序遍歷
  遍歷順序為 left ->   right-> root
  A B / C * D * E +

  ```java
      @Override
      public void traverse() {
          this.traverse(this.getRoot());
      }
  
      private void traverse(TreeNode currentNode){
          if(currentNode == null)
              return;
          // Traversing in post-order
          this.traverse(currentNode.getLeftNode());
          this.traverse(currentNode.getRightNode());              				
        	System.out.println(currentNode);
      }
  ```

  

4. Level-Order
  一個level拜訪完後，再接著拜訪下一個level

  +* E *  D / C A B

  透過Queue實做BFS演算法即可，這邊就不實做了。

### 5. Compare two binary search tree

比較兩個二元搜索樹是否相同，需要比對node 所在位置與 node value，確定每個都相同後，才視為相同的 BST。

```java
@Override
    public boolean compare(Tree<T> tree) {
       if(this.getRoot() !=null && tree.getRoot() !=null) {
           return this.compare(this.getRoot(), tree.getRoot());
       }
       return false;
    }

    private boolean compare(TreeNode<T> node1, TreeNode<T> node2){
        // Node data相同且左右 children皆相同
        if(node1 !=null && node2 !=null){
                return node1.getData().compareTo(node2.getData()) ==0
                        && compare(node1.getLeftNode(), node2.getLeftNode())
                        && compare(node1.getRightNode(), node2.getRightNode());
        }
        return node1 == node2;
    }
```

### 6. K-th smallest element in binary search tree

目標是找出 BST中第 k 小的內容，如在一個BST中排行第6小的內容是多少。
可以透過 in-order traversal 後，找出第6小的內容，時間複雜度為 O(N)。

或是可以透過下列方式做到相同效果。
概念：BST中，所有在左側的subtree value皆小於current node；所有在右側的subtree value 皆大於current node。

所以當要找排行第 k 小的node時，我們可以計算左側subtree的node 數量，將數量加上1 (稱為 leftNodeSIze) 後與要查詢的 rank 做比較，

+ 如果 rank小於leftNodeSize，則表示第 k 小的 node位於 左邊subtree中，所以同樣的流程繼續往左subtree找。
+ 如果rank大於 leftNodeSize，則表示第k小的 node位於右邊的subtree中，則將要查的rank減掉leftNodeSize後，重複同樣流程往右側 subtree找。
+  leftNodeSize算法：left subtree nodes count +1
+ 如果BST夠平衡的情況下，時間複雜度平均為O(LogN)，最糟為O(N)

<img src="https://imgur.com/I1Ahpqe.png" />

假如要在上面的BST中查詢第 7 小的內容，則流程如下：

<img src="https://imgur.com/tlF2b7u.png" />

首先計算 root 35的 leftNodeSize為6， rank > leftNodeSize，可以知道rank =7 的node存在於右側subtree中，所以rank - leftNodeSize =1，繼續往右側subtree找第 1小的Node

<img src="https://imgur.com/Eno2nDg.png" />

到35的 right subtree後，計算 44的 leftNodeSize為 2，leftNodeSize > rank，可以知道 rank=1的node存在於 44的left subtree中，所以繼續往 44 left subtree中找第1小的 node。

<img src="https://imgur.com/Lo6e3N4.png" />

到44的left subtree後，計算leftNodeSize為 1，與要查詢的rank相同，即找到要查詢的node，40即為整個Binary Search Tree中第 7小的value。

```java
public TreeNode<T> getNodeByRank(int rank){
        if(this.getRoot() == null)
            return null;
        return this.getNodeByRank(this.getRoot(), rank);
    }

    private TreeNode<T> getNodeByRank(TreeNode<T> node, int rank){

        // left children數量 再加上 current node
        int leftNodeSize = this.getLeftNodeSize(node.getLeftNode()) +1;

        // Found
        if(leftNodeSize == rank)
            return node;

        // 目標大於當前left node size，則找right children
        if(rank > leftNodeSize)
            return getNodeByRank(node.getRightNode(), rank - leftNodeSize);
        else
            return getNodeByRank(node.getLeftNode(), rank);
    }

    private int getLeftNodeSize(TreeNode<T> currentNode){
        if(currentNode==null)
            return 0;
        return getLeftNodeSize(currentNode.getLeftNode()) + getLeftNodeSize(currentNode.getRightNode()) +1;

    }
```

