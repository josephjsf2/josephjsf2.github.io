---
title: 資料結構與演算法：Red Black Tree 紅黑樹 part 2
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

這篇主要在說明紅黑樹的刪除怎麼處理，[第一部份的連結](./red-black-tree-part-1.html)。

<!--more-->


刪除的步驟與 BST 相同，透過與要刪除節點的 predecessor或 successor 交換，再進行刪除，透過這個方式，降低刪除節點時造成整個樹巨大變更。

這邊複習 successor與 predecessor在BST中的含意是什麼：

>successor：在 inorder traversal 中下一個比自己數值大的節點
>
>predecessor：在 inorder traversal 中前一個比自己數值小的節點

在刪除節點的過程中，會用到x、y 與 z三個位置，不同情況下有不同的意義。z 表示最初要刪除的節點位置， y 表示最後實際上會從樹中被移除的節點位置， x 表示 y 節點被刪除後，要遞補至原本 y 位置的節點。

## 樹的節點轉換(Transplant)

Transplant 用途在於傳入 u與 v兩個節點， v 將會取代 u 的位置，過程會在 transplant 中調整 u.p child 指向與 v.p 指向節點，讓原本 u.p 指向 v，同時 v.p指向 u.p

```java
TRANSPLANT(T, u, v)
	if u.p == T.nil
		T.root = v
	elseif u== u.p.left
		u.p.left=v
	else 
		u.p.right =v
	v.p=u.p
```

## 刪除功能

方式與二元樹不會差太多，但是需要調整 parent 與 child 指向，同時需要記錄實際被刪除顏色，過程中，這邊會特別先記錄下『 y 節點最初的顏色』，目的是後續修正紅黑樹時使用：

```java
    @Override
    public void deleteElement(T data) {

        TreeNode z = this.findElement(data); //要刪除的資料
        if(z ==null)
            return;
        TreeNode y = z;  // 刪除node後，要補上的節點
        TreeNode x=null;    //要取代原本 Y 位置的節點
        
        //  最後 x 的parent，目的在於 x 可能是 null，但fixup需要 x的 parent 資訊
        TreeNode parentOfX = null;  

        NodeColor successorOriginColor = y.getColor();  // 儲存 Y 原始顏色
        
        if(z.getLeftChild() == null){
            x = z.getRightChild();
            parentOfX = z.getParent();
            transplant(z, z.getRightChild());

            if(parentOfX==null)
                this.root = x;
        }else if(z.getRightChild() ==null){
            x = z.getLeftChild();
            parentOfX = z.getParent();

            if(parentOfX==null)
                this.root = x;
            transplant(z, z.getLeftChild());
        }else{
          	// z 左右節點都不是 null
            y = this.getMin(z.getRightChild()); // 取得 Z 的 successor
            successorOriginColor = y.getColor();
            x = y.getRightChild();

            if(y.getParent() == z) {
                // Y是Z的直屬child
                parentOfX = y;
                if(x!=null)
                    x.setParent(y);
            }else {
                parentOfX = y.getParent();
                // 將 X 取代 Y
                transplant(y, y.getRightChild());
               
                y.setRightChild(z.getRightChild());
                y.getRightChild().setParent(y);
            }
            transplant(z, y); // 將 Y 取代 Z
            y.setLeftChild(z.getLeftChild());
            y.getLeftChild().setParent(y);
            y.setColor(z.getColor());

            if(y.getParent() == null)
                this.root = y;
        }

        // 如果 Y 原始顏色是黑色，才進行 FIXUP
        if(successorOriginColor == NodeColor.BLACK){
            deleteFixup(x, parentOfX);
        }
    }
```

從刪除的程式來看，如果z位置的子節點少於2個，透過二元樹方式執行刪除，會直接將節點刪除，不會做節點的交換，所以 y 節點位置會等同 z 節點位置，x位置表示刪除 y 後，要補上y位置的節點。(圖片第二個樹的 x 位置畫錯了，應該在 17 的位置才對)

<img class="img-fluid" src="https://imgur.com/pMhzDge.png"/>

如果z節點位置的子節點有兩個，透過二元樹執行刪除時，會找到 z 節點的 successor 節點位置 y，表示 y 位置節點會取代掉 z位置節點 ( 將 y 節點位置移至 z 節點位置，表示 y 位置的節點實際上被刪除了)，且 y 的顏色也會調整為 z 的顏色，而 x 位置節點表示當 y 取代 z的位置後，要遞補至原本 y 位置的節點。這個過程中，<u>需要記錄最初找到 y 時 y 節點的顏色。</u>

<img class="img-fluid" src="https://imgur.com/6rp8YkQ.png"/>

## 刪除後調整

刪除的第一步驟大致如上，在執行刪除後，可能會違反紅黑樹的規則，所以在刪除後可能需要再做一次修正，這邊會透過 y 最初的顏色來判斷是否需要修正，並透過 x 來判斷如何修正，修正的情境如下：

1. 如果原本 y 位置的節點顏色是紅色，則不需要進行修正，因為刪除紅色節點不會違反紅黑樹規則
2. 如果原本 y 位置的節點顏色是黑色，則需要進行修正，因為會違反紅黑樹規則：從任何節點至葉節點(leaf)的任意簡單路徑上，黑色節點的數量會相同，所以如果過程中， y原本顏色是黑色，被刪除後，會有一個路徑上少一個黑色節點，就會違反這個規則，所以需要進行修正。

違反紅黑樹規則的情境可以分成幾類，這邊先看的程式：

```java
// 因為 x 可能有 null 情況，故這邊連同 parent一同傳入   
private void deleteFixup(TreeNode x, TreeNode parent){
        while(x!= this.root && (x== null || x.getColor() == NodeColor.BLACK )){

            TreeNode w;	// sibling
          	// 如果 x 是在 parent的左邊
            if(parent.getLeftChild() == x){
                w = parent.getRightChild();

                // case I： sibling 是紅色節點，會將 tree調整為 case II III IV 其中一種
                if(w !=null && w.getColor() == NodeColor.RED){
                    w.setColor(NodeColor.BLACK);
                    parent.setColor(NodeColor.RED);
                    leftRotation(parent);
                    w = parent.getRightChild();
                }

              	// case II： sibling 是黑色且 sibling 兩個 child都是黑色
                if( (w.getRightChild()== null || w.getRightChild().getColor()== NodeColor.BLACK) ||
                        (w.getLeftChild()== null || w.getLeftChild().getColor() == NodeColor.BLACK )){
                    w.setColor(NodeColor.RED);
                    x = x.getParent();	// 重新指定 x，在下一個迴圈再依據新的 case 調整
                    parent = x.getParent();
                }else{
                    // case III：sibling是黑色，且sibling 左邊child是紅色，右邊 child是黑色
                    if(w.getRightChild() == null ||w.getRightChild().getColor() == NodeColor.BLACK){
                        w.getLeftChild().setColor(NodeColor.BLACK);
                        w.setColor(NodeColor.RED);
                        rightRotation(w);
                        w = parent.getRightChild();
                    }

                    // case IV：sibling是黑色，且sibling 右邊child是紅色
                    w.setColor(parent.getColor());
                    parent.setColor(NodeColor.BLACK);
                    w.getRightChild().setColor(NodeColor.BLACK);
                    leftRotation(parent);
                    x = this.root; //調整完成，直接指向 root離開迴圈
                }
            }else{
							// 與左邊操作相反，左右對調即可，故省略
            }
        }
				// x 最後都會被調整為黑色，包含 x=root情況
        x.setColor(NodeColor.BLACK);

    }  	
```

首先，會分為兩個部分來修正，<u>會依據 x 是位於左邊節點還是右邊節點來做處理</u>，接著透過 x 的兄弟節點(sibling) w 來判斷處理方式，這邊一一列出，總共有 4種 case，並且<u>分為左右兩種，左右兩種操作方式相同，只是左右的操作對調</u>：

* <u>這邊的 case 是將 y取代z位置，並且 x 補上至 y原本位置後，視當下條件整理出來的情境與調整方式</u>
* 下列的各個case都是整個樹中的片段，不代表整個樹的樣貌
* 下列case 規則都是建立在紅黑樹的規則中，<u>如果要驗證，一定要符合條件的紅黑樹來驗證</u>，而不是隨意使用一個不合規則的紅黑樹驗證(我就是做了這件事情所以卡了很久)，所有的case要在合法的紅黑樹才會有效

### case 1： x的 sibling w是紅色，這一步目的是將 w 調整為黑色，再透過 case II III IV 處理

1. w 設為黑色
2. x.parent 設為紅色
3. leftRotation(x.parent)
4. 重新指定 w = x.p

<img class="img-fluid" src="https://imgur.com/UPCkYan.png" />

### case 2：x的 sibling w 是黑色，且 w 的兩個 children 都是黑色

1. 將 w 設為紅色
2. x = x.parent，進入下一個迴圈重新判斷 case

sibling w一側黑色節點數量明顯比較多，只需要將 w 設為紅色，下方就會恢復平衡，但仍需要判斷上方情況，所以<u>將 x 往上移動一層</u>。

<img class="img-fluid" src="https://imgur.com/5WvV8oY.png" />

進入下一輪迴圈後，如果 x 是紅色，則直接修正為黑色即修正結束，如果 x是黑色，則重新判斷符合哪一個 case 再繼續調整

### case 3： x的 sibling w是黑色，且 w的左邊 child是紅色，w的右邊 child是黑色

1. w的左邊節點設為黑色
2. w顏色設為紅色
3. RightRotation(w)
4. w = x.parent.right

<img class="img-fluid" src="https://imgur.com/19h8C6F.png"/>

case 3 這樣調整後，會符合 case 4 的條件，所以 case 3的目的是將條件轉為 case 4。

### case 4：x的 sibling w 是黑色，且 w的右邊 child是紅色

1. w設為 x.parent顏色
2. x.parent 設為黑色
3. w.right 設為黑色
4. LeftRotation(x.parent)
5. x = root 

將x設為root，目的是離開迴圈，表示調整完成

<img class="img-fluid" src="https://imgur.com/Dn7A9aB.png">

下列直接用幾個範例來示範刪除的功能，有了上面的概念，接著直接看範例會比較簡單。下列範例是擷取自[Red-black tree deletion: steps + 10 examples](https://www.youtube.com/watch?v=eO3GzpCCUSg) 中的部分範例。

## 範例一

<img class="img-fluid" src="https://imgur.com/gdN4Djb.png" />

## 範例二

<img class="img-fluid" src="https://imgur.com/Hfzas5T.png"/>

## 範例三

<img class="img-fluid" src="https://imgur.com/GdwlOlS.png"/>

## 範例四

<img class="img-fluid" src="https://imgur.com/drP94FA.png"/>

## 範例五

<img class="img-fluid" src="https://imgur.com/dNg7EFN.png"/>

## 範例六

<img class="img-fluid" src="https://imgur.com/nQFZjyo.png"/>

## 範例七

<img class="img-fluid" src="https://imgur.com/hmR9uz4.png"/>

## 參考文章

紅黑樹刪除功能在一開始自學還滿痛苦的，過程卡在很多地方，還好友許多熱心的人分享了學習文章與影片，相當感謝網路上所有熱心的人。

教科書：Introduce to Algorithms 3rd edition

這篇文章寫得相當詳細，幫助我很多：[Red Black Tree: Delete(刪除資料)與Fixup(修正)](http://alrightchiu.github.io/SecondRound/red-black-tree-deleteshan-chu-zi-liao-yu-fixupxiu-zheng.html)

學到最後可以用這個影片來驗證，這影片說明的非常好：[Red-black tree deletion: steps + 10 examples](https://www.youtube.com/watch?v=eO3GzpCCUSg)

