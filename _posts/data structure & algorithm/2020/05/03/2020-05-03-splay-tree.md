---
title: 資料結構與演算法：Splay Tree 伸展樹
layout: post
categories: data structure and algorithm
unsplashTag: 'code,algorithm'
tags:
- tree
- splay tree
- BST
- data structure
- algorithm


---

Splay Tree 學習筆記。

<!--more-->
## 伸展樹(Splay Tree) 介紹

是一種二元搜索樹，特性是『最近一次搜尋或新增的內容，會被移至樹的 root』，當下次搜尋同一個內容時，速度可以有所提升，因為 tree的搜尋都是從 root 開始，愈接近 root 愈快被找到。將最近搜尋到的項目移至 root 的行為稱為 splay，很重要一點是，在 splay 後樹不一定會是一棵左右平衡的樹，最糟可能會變成一個傾斜樹(Skewed Tree)。

Splay Tree大部分的概念與二元搜索樹都相同，大部分差異在於最後要多做一個 splay的動作。

## Splaying

首先說說Splay Tree的 Splay功能，Splay Tree 會在執行完搜尋與新增後，執行Splay的動作，目的是讓最近使用到的資料提升至 root。

Splay的操作實際上是透過 Left Rotation 與 Right Rotation來達成，透過對樹做旋轉的操作，讓節點提升至 root，同時讓樹保持相同的 inorder。

Splay 會依據節點位於樹的左側與右側有相反的操作，總共可分為三種，都是Left/ Right Rotation的搭配組合，來讓節點逐漸往上移至 root，這邊假設要移動的節點是 x：

* ### Zig：節點 x 的 parent 就是 root，所以只需要對 x.parent做 left 或 right rotation即可

<img class="img-fluid" src="https://imgur.com/D8IA2Oc.png" />

* ### Zig - Zig：表示 x.parent 不是 root，且 x與 x.parent 皆為左邊的節點或是皆為右邊的節點，因此會執行兩次 Left Rotation或 Right Rotation

<img class="img-fluid" src="https://imgur.com/EeKDlWH.png"/>

* ### Zig - Zag： x.parent不是 root，並且 x.parent是左節點， x是右節點；或是x.parent是右節點，x是左節點。

<img class="img-fluid" src="https://imgur.com/rkUoHyW.png"/>

上面的動作，會持續直到 x 變成 root才結束。

程式碼實做如下：

```java
    private void splay(TreeNode x){
        while(x.getParent()!=null) {
            // Zig
            if (x.getParent().getParent() == null) {
                if (x.getParent().getLeftChild() == x)
                    rightRotation(x.getParent());
                else
                    leftRotation(x.getParent());
            } else if (x.getParent().getParent().getLeftChild() == x.getParent() && x.getParent().getLeftChild() == x) {
                //  X 位於左子節點的左子節點 ZIG - ZIG
                rightRotation(x.getParent().getParent());
                rightRotation(x.getParent());
            } else if (x.getParent().getParent().getRightChild() == x.getParent() && x.getParent().getRightChild() == x) {
                //  X 位於右子節點的右子節點 ZIG- ZIG
                leftRotation(x.getParent());
                leftRotation(x.getParent());
            } else if (x.getParent().getParent().getLeftChild() == x.getParent() && x.getParent().getRightChild() == x) {
                //  X 位於左子節點的右子節點 ZIG - ZAG
                leftRotation(x.getParent());
                rightRotation(x.getParent());
            }else{
                //  X 位於右子節點的左子節點 ZIG - ZAG
                rightRotation(x.getParent());
                leftRotation(x.getParent());
            }
        }
    }
```

## Insert 新增節點

新增語法與二元搜索樹相同，但是在新增完成後，需要將新增的節點提升到 root，片段程式碼：

```java
    public void insertElement(Comparable data) {
	      // Some insertion code
        // 執行新增的程式碼，故省略，完成後執行 splay新增的節點
        splay(newNode);
    }
```

## Search 搜尋節點

查詢語法與二元搜索樹相同，但在搜尋後，需要將找到的節點提升到root，片段程式碼：

```java
    public TreeNode findElement(Comparable data) {
        // Some search code
        // 執行查詢的程式碼，故省略，完成後執行 splay查詢到的節點
        splay(target);
    }
```

## Delete 刪除節點

刪除語法概念與二元搜索樹相同，但在過程中看到另一個方式。概念是刪除時，會先將要刪除的節點提升到 root， root 就沒用了，因為被刪除，接著將 root 的左右節點拆為兩個 subtree，這時會有三種情況：

* ### 左邊 subtree沒東西，所以將右邊subtree的root設為整棵樹的 root

<img class="img-fluid" src="https://imgur.com/uTLSIqY.png"/>

* ### 右邊subtree沒東西，所以將左邊subtree的root設為整棵樹的 root

<img class="img-fluid" src="https://imgur.com/CGfBj2z.png" />

* ### 左右都有 subtree，則將右邊subtree的最小節點提升至subtree的 root，這時候右邊subtree  root節點左邊會沒有節點，接著將左邊 subtree接至右邊 subtree root的左邊節點即可

<img class="img-fluid" src="https://imgur.com/GuFq0qO.png"/>

## Splay Tree實際應用

1. 實做 Cache
2. Garbage Collection

## 時間複雜度

這邊的 amortized 是指在發生最糟情況時下，平均的耗費時間。可見 [wiki上定義](https://zh.wikipedia.org/wiki/%E5%B9%B3%E6%91%8A%E5%88%86%E6%9E%90)

```
| 動作名稱 | Average  | Worst case         |
| -------- | -------- | ------------------ |
| 查詢     | O(log N) | amortized O(log N) |
| 新增     | O(log N) | amortizedO(log N)  |
| 刪除     | O(log N) | amortizedO(log N)  |
```

## 總結

學習時覺得難度不高，忽略了一些細節，結果在寫程式碼時弄錯了 splay 順序，又花了些時間 debug。

不過過程中找到不錯的演算法網站，也是有些收穫，在學演算法過程中有視覺的東西來驗證與學習，真的可以大幅加學習與理解的速度。

## 參考資料

[Splay Tree Wiki](https://en.wikipedia.org/wiki/Splay_tree)

[Splay Tree Visualization](https://www.cs.usfca.edu/~galles/visualization/SplayTree.html)

