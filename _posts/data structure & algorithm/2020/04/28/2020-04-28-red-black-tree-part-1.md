---
title: 資料結構與演算法：Red Black Tree 紅黑樹 part 1
layout: post
categories: data structure and algorithm
unsplashTag: 'code,algorithm'
tags:
- tree
- red black tree
- BST
- data structure
- algorithm

---

紅黑樹學習筆記。

<!--more-->


因為一些因素，自己休息了一段時間，回復後繼續保持學習熱忱！
這次的主題紅黑樹大約一個月前讀完，只是到現在才打下筆記，跟預料的一樣，幾乎全忘了，所以學習時間就變成了兩倍。

## 紅黑樹(Red-Black Tree)介紹

之前提過AVL Tree，AVL Tree是讓二元樹隨時保持在balance狀態的二元樹演算法，在新增刪除中會自動修正至平衡，而紅黑樹也有這個特性，但相較於AVL Tree，紅黑樹只有在違反某些條件下，才會執行平衡修正，其餘是不會做修正的。

紅黑樹節點會有以下欄位來儲存資訊：

1. value：要儲存之資料
2. left：左節點
3. right：右節點
4. color：節點顏色
5. parent：記錄parent node

```java
@Gettter
@Setter
public class Node {
    private int data;
    private NodeColor color;
    private Node leftChild;
    private Node rightChild;
    private Node parent;

    public Node(int data){
        this.data = data;
        this.color = NodeColor.RED;

    }
}

public enum NodeColor {
    RED, BLACK;
}

```

同時紅黑樹會滿足下列幾個條件(<u>紅黑樹最重要的特性與條件</u>)：

1. 每個節點不是黑色就是紅色
2. 整個 tree的 root是一定黑色
3. <u>每個 leaf(null 值) 視為黑色</u>
4. 紅色節點其左右兩個 children一定是黑色(意即不會有連續兩個紅色節點做為父子關係)，黑色節點無此限制
5. 樹上的任何一個節點，其任何到達 tree的 leaf 的 simple path上(意即只能繼續往下方路徑，不可往上迂迴)，會擁有相同數量的黑色節點

如果有違反的情況，則必須進行修正，就是類似AVL Tree做的 rotation，修正到tree符合 Red-Black tree條件為止

紅黑樹圖形如下：

<img class="img-fluid" src="https://imgur.com/dnyl5Ex.jpg" />

從圖片可以看出：

1. 每個節點接標有顏色，不是紅就是黑

2. root 是黑色

3. 沒有兩個紅色節點擁有父子關係

4. 節點 7從左右兩側到達任意一個 leaf路徑，都擁有2個黑色節點

   1. 7 -> 2 -> 1
   2. 7 -> 2 -> 5 -> 4
   3. 7 -> 11 -> 8
   4. 7 -> 11 -> 14 -> 15

   這四個路徑上，都只存在兩個黑色節點，節點數都是相同的

如果在樹異動的過程，如新增，違反了上列的條件，則需要做調整，來讓樹繼續維持相同的特性，通常會透過 left rotation/ right rotation 與改變節點顏色的方式來做到。



## Left Rotation/ Right Rotation

左右旋轉的原理與AVL Tree 的方式幾乎是一樣的，這邊不再討論 Left Rotation與 Right Rotation，因為與AVL Tree相同。

需要留意的是AVL Tree實做實是用單向的方式，由 parent指向 children，而Red-Black Tree 是使用雙向的，除了 parent指向 children外， children同時也會指向 parent。

Left/Right Rotation程式碼：

```java
    private void rightRotation(TreeNode node){
        System.out.println("Rotating to right on the Node: " + node);
        TreeNode tempRight  = node.getLeftChild().getRightChild();
        TreeNode tempLeft = node.getLeftChild();

        node.setLeftChild(tempRight);
        if(tempRight !=null)
            tempRight.setParent(node);
    
        // 設定 node 的 parent 指向之child node
        if(node.getParent() == null)
            root = tempLeft;
        else if(node.getParent().getLeftChild() == node)
            node.getParent().setLeftChild(tempLeft);
        else
            node.getParent().setRightChild(tempLeft);

        tempLeft.setParent(node.getParent());
        tempLeft.setRightChild(node);
        node.setParent(tempLeft);
    }

    private void leftRotation(TreeNode node){
        System.out.println("Rotating to left on the Node: " + node);
        TreeNode tempLeft = node.getRightChild().getLeftChild();
        TreeNode tempRight = node.getRightChild();
        node.setRightChild(tempLeft);
        if(tempLeft !=null)
            tempLeft.setParent(node);
        
        // 設定 node 的 parent 指向之child node
        if(node.getParent() == null)
            root = tempRight;
        else if(node.getParent().getRightChild() == node)
            node.getParent().setRightChild(tempRight);
        else
            node.getParent().setLeftChild(tempRight);

        tempRight.setLeftChild(node);
        tempRight.setParent(node.getParent());
        node.setParent(tempRight);
    }
```

## 新增節點

新增的部分，與AVL Tree大同小異，差異在於 AVL Tree 需要隨時計算了兩側節點高度，並做適當的修正讓樹變回 balance狀態，而紅黑樹則是需要檢查是否符合條件，如果不符合條件，則需要修正至符合條件

不管修正到怎麼樣，<u>最後一定要將 root 節點設為黑色，才不會違反紅黑樹的限制。</u>

在新增節點時，<u>預設新增的節點顏色是紅色</u>，這邊會用 X 節點表示新增的節點，實際上會有六種case，左側3種與右側3種，但兩個的情況剛好是鏡像，<u>右側的操作等同左側的反向操作</u>，故這邊主要會用左側的情境來說明：

### case 1：uncle 節點是紅色

從這邊來看，左右兩種情況是鏡像結果，所以這邊僅會用左側的圖片情境來說明，右側的操作與左側的相反

X的uncle節點(C節點)是紅色，這個情況最單純，只需要對節點重新標顏色即可

```
X's parent B -> 黑

X's uncle C-> 黑

X's grandparent A-> 紅
```

轉換前：

<img class="img-fluid" src="https://imgur.com/W6PQlfN.png" />

轉換後：

轉換就是將節點重新標色，做完後， 下面區塊即完成檢查，但因為grandparent節點變成紅色，<u>所以需要再往上層再做一次重複的檢查</u>，確保重新表色後整個樹都滿足紅黑樹的限制。

<img class="img-fluid" src="https://imgur.com/4slSZ4X.png" />

### case 2：uncle 節點是黑色， X 是右節點 (以左側為說明情境)

遇到這個情況，則需要做轉換，讓 case 2轉變為 case 3的情況，因此要對 X 的parent做一次left rotation(如果是右側則是right rotation)

```
left-rotation(X's parent B)
```

轉換前：

<img class="img-fluid" src="https://imgur.com/g9G9NEN.png" />

轉換後：

<u>要特別注意是轉換前焦點是放在 X節點，轉換後會變成 B節點，也就是X節點原本的 parent</u>

<img class="img-fluid" src="https://imgur.com/H9qkU7c.png" />

做完轉換後，接續做 case 3的處理

### case 3：uncle 是黑色，且X是左節點(以左側為說明情境)

case 2 最後即轉變為 case 3的樣子接續處理，如果是 case 2轉過來的情況，這邊一樣會將 X 視為新增的節點

```
X's grandparent A -> 紅
X's parent B -> 黑
right-rotation(X's grandparent A)
```

轉換前：

<img class="img-fluid" src="https://imgur.com/6p7nMG0.png" />

轉換後：

<img class="img-fluid" src="https://imgur.com/TuiNilt.png" />

AVL Tree 在查詢的速度上是大於等於  Red-Black Tree的(因為一直保持著 balance狀態)，但新增的速度則是 Red-Black Tree大於等於 AVL Tree。因此，如果情境是需要大量的異動節點，會建議使用 Red-Black Tree，效能會比較好；如果異動的情況較少，會建議使用AVL Tree。

再提醒一次，<u>修正到最後 root 可能會變成紅色，所以在最後完成時，一定要將 root 設定為黑色</u>

這邊實際演練一次，讓程式依序新增10, 20, 30, 40, 50, 60，觀看整體的變化：

新增10，因為是root的關係，所以顏色會修正為黑色：新增20，沒有違反紅黑樹規則，所以不做任何動作

<img class="img-fluid" src="https://imgur.com/VJ9V6Yp.png" />

新增30，因為20與30皆為紅色，紅黑樹中不允許連續兩個紅色節點做為父子關係，所以需要修正，這個情況下，滿足case 3條件( uncle 在 null情況下視為黑色)，因此會將 10標為紅色，20標為黑色，並對10 做left rotation

<img class="img-fluid" src="https://imgur.com/qlkyeAY.png"/>

做完修正後如下，到這邊沒有違反規則，所以修正完成：

<img class="img-fluid" src="https://imgur.com/VuJetsw.png" />

新增40，違反紅黑樹規則，所以需要再做修正，這邊滿足 cas I情況， uncle是紅色，所以只需要做節點的顏色調整即可

<img class="img-fluid" src="https://imgur.com/QjalKKk.png"/>

調整完如下，但是因為 root被改為紅色，所以需要再將root修正為黑色

<img class="img-fluid" src="https://imgur.com/FnwJlQE.png" />

因此最後結果如下：

<img class="img-fluid" src="https://imgur.com/z4SHfAL.png" />

新增 50，違反紅黑樹規則，滿足 case3，需要調整30與40的顏色，並對30做 left rotation

<img class="img-fluid" src="https://imgur.com/cT03roh.png"/>

調整後即無須再調整：

<img class="img-fluid" src="https://imgur.com/tRWt75O.png" />

最後新增60，違反規則，滿足 case I，所以調整顏色即可，但是需要從 40 再往上檢查是否違反規則

<img class="img-fluid" src="https://imgur.com/78VbGCg.png" />

調整後：

<img class="img-fluid" src="https://imgur.com/BGJlRBP.png"/>

接著新增70與80，新增70的規則與前面相同，故不再描述，要說的是新增80的時候，新增後如下：

可以觀察到整棵樹已經有點往右偏離，這時因違反紅黑樹規則，滿足case I，所以對節點顏色作調整

<img class="img-fluid" src="https://imgur.com/qUsFx1M.png"/>

調整後：

在調整後，60的節點需要再做一次檢查，可以觀察到在調整顏色後，雖然下層滿足了紅黑樹規則，但是60的地方違反了紅黑樹規則，滿則case III，所以需要對20與40做重新標色，並對20做 left rotation

<img class="img-fluid" src="https://imgur.com/ym7Hbp9.png"/>

調整後，可以發現樹回復平衡，且所有節點都滿足紅黑樹的規則

<img class="img-fluid" src="https://imgur.com/L5pEZ6B.png"/>

新增節點的程式如下：

```java
    @Override
    public void insertElement(T data) {
        TreeNode node = new TreeNode(data);
        this.root = this.insert(root, node);
        fixInsert(node);
    }

    private TreeNode insert(TreeNode root, TreeNode newNode){

        if(root == null)
            return newNode;

        if(root.getData().compareTo(newNode.getData()) >=0){
            root.setLeftChild(this.insert(root.getLeftChild(), newNode));
            root.getLeftChild().setParent(root);
        }else{
            root.setRightChild(insert(root.getRightChild(), newNode));
            root.getRightChild().setParent(root);
        }
        return root;
    }
```

整體新增完後，接著從新增的節點開始檢查否違反規則，並做修正，這邊貼程式碼，實際建議直接參考演算法代碼會比較簡單

```java

    private void fixInsert(TreeNode node){
        while(node!= this.root && node.getParent().getColor() == NodeColor.RED){
            TreeNode parent = node.getParent();
            TreeNode grandParent = parent.getParent();
            TreeNode uncle = null;

            // left side
            if(parent == grandParent.getLeftChild()){
                uncle = grandParent.getRightChild();
                // case I
                if(uncle!=null && uncle.getColor() == NodeColor.RED){
                    uncle.setColor(NodeColor.BLACK);
                    parent.setColor(NodeColor.BLACK);
                    grandParent.setColor(NodeColor.RED);
                    // now focus on grandparent node
                    node = grandParent;
                }else {
                    // case II
                    if(node == parent.getRightChild()){
                        leftRotation(parent);
                        parent = node;
                        node = parent.getParent();
                    }
                    // case III
                    grandParent.setColor(NodeColor.RED);
                    parent.setColor(NodeColor.BLACK);
                    rightRotation(grandParent);
                }
            }else {
                uncle = grandParent.getLeftChild();
                // case I
                if(uncle !=null && uncle.getColor() == NodeColor.RED){
                    parent.setColor(NodeColor.BLACK);
                    uncle.setColor(NodeColor.BLACK);
                    grandParent.setColor(NodeColor.RED);
                    // now focus on grandparent node
                    node = grandParent;
                }else {
                    if(node == parent.getLeftChild()){
                        // case II
                        rightRotation(parent);
                        parent = node;
                        node = parent.getParent();
                    }
                    // case III
                    grandParent.setColor(NodeColor.RED);
                    parent.setColor(NodeColor.BLACK);
                    leftRotation(grandParent);
                }
            }
        }
        this.root.setColor(NodeColor.BLACK);
    }
```

## 紅黑樹與AVL Tree

AVL Tree 會在過程中不斷做 rotate的動作來修復整棵樹，平衡性最好，透過計算BF讓樹兩側高度差不大於1。

相較之下，紅黑樹兩側高度差可能會有大於1的情況，但在後續會逐漸修復回來。

也因為AVL Tree在新增時做修正的頻率比較高，所以新增時速度會比較慢，但是查詢速度最快，因為樹的平衡最好。

紅黑樹在新增時速度比較快，因為不會太頻繁的修正整個數的平衡，但也因為這樣，所以搜尋的速度會比AVL Tree慢一些。

總體來說，如果經常有需要新增節點的情況，建議使用紅黑樹，效率會比較好。如果新增的情況相對少，則可以使用AVL Tree。

## 紅黑樹實際應用

1. Completely Fair Scheduler(CFS) in Linux Kernel
2. Computational Geometry
3. HashMap implementation in Java 8(取代舊的 LinkedList)

## 時間複雜度

```
| 名稱                          | 查詢      | 新增     | 刪除      |
| ---------------------------- | -------- | -------- | -------- |
| AVL Tree(Average case)       | O(log N) | O(log N) | O(log N) |
| AVL Tree(Worst case)         | O(log N) | O(log N) | O(log N) |
| Red Black Tree(Average case) | O(log N) | O(log N) | O(log N) |
| Red Black Tree(Worst case)   | O(log N) | O(log N) | O(log N) |
```

## 總結

這邊只介紹紅黑樹的新增與概念，刪除的部分因為比較複雜，只好留到下一篇[紅黑樹刪除功能](./red-black-tree-part-2.html)說明，因為自己在讀文章時花了很多時間在弄懂整體思路。學了紅黑樹後，大概最大的收穫是領悟到在學習資料結構或是演算法時，搭配影片或是大量圖片可以大大大幅的提高學習效果，也可以幫助理解演算法的內容，演算法內容懂了，自己實做程式才不會卡住。


這邊的程式碼可以參考連結：[Red Black Tree implementation](https://github.com/josephjsf2/DS-Alg/tree/master/RedBlackTree)
