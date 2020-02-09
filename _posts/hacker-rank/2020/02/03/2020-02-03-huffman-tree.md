---
title: HackerRank - Huffman Decoding
layout: post
categories: HackerRank, algorithm
unsplashTag: 'code'
tags:

- huffman decoding
- hacker rank
- algorithm
- huffman tree

---

解個題順便複習 Huffman Tree。

<!--more-->
## 題目說明：

題目描述較長，可以直接到[Hacker Rank](https://www.hackerrank.com/challenges/tree-huffman-decoding/problem)上看題目說明。

簡單來說就是題目會給定一個字串 s，與一個 Huffman Tree，我們需要依照給予的 Huffman tree內容將字串 s 還原為原本內容。

題目不困難，只是如果在解題前可以充分瞭解Huffman code與 Huffman tree的概念，解起來會輕鬆許多。

## 什麼是 Huffman Code?

這邊做一個簡單的介紹，Huffman code是無失真的資料壓縮演算法，透過『字元出現的頻率』來決定每個字元轉為二元字串時表達方式，意思即依據每個字元出現的頻率，出現次數愈多的字編碼後長度較短，出現頻率愈低的字編碼後長度愈長。

這邊先解釋，codeword 這邊是指編碼過後呈現的字元。

舉例來說，以下列的表來看：

<img class="img-fluid" src="https://imgur.com/DJS8yEy.png" />

可以看見每個字元出現的頻率，a 出現 45k 次；b出現 13k次，依此類推。

如果現在不以壓縮方式傳輸，每個字都固定以 3 bits (Fixed-length codeword) 來表示，則總共需要

```mathematica
45k * 3 + 13k *3 + 12k *3 + 16k *3 + 9K *3 + 5k *3 = 300k bits
```

如果改以前面提到的方式壓縮來處理(Variable-length codeword)，則總共需要

```mathematica
45k *1 + 13k * 3 + 12k *3 +16k *3 +9k*4 + 5k *4 = 224k bits
```

比較兩者，同樣的內容，使用 Variable-length codeword  會比 Fixed-length codeword 節省幾乎 25 %的空間，而 Huffman code就是這樣的概念，出現次數愈多，則編碼後二進制字元欲短，反之欲長，

### Prefix codes

原文定義是： 

> No codeword is also a prefix of some other codeword, such code are called orefix codes.

這一個特點相當重要，**每一個 codeword都不會是其他 codeword的前綴**，如前面的表格， a 的 codeword如果是 0， 那**絕對不會有任何一個字元的 codeword 是 0開頭**，意即不會有 codeword為 00或01的情況出現。如 c 的 codeword是 100，就不會出1001或是 10000001 這類的 codeword。這樣的情況稱為 prefix codes。

在滿足這個條件下，才能建立起 Huffman codes。



## 建立 Huffman Tree

這邊只演示作法與說明概念，不會實做建立Huffman Tree的程式

如果今天有一組資料，資料如下：

<img class="img-fluid" src="https://imgur.com/e0bYfdO.png" />

這邊數字已經依照出現的頻率排序，而Huffman Tree建立的作法概略如下，因為建立的是二元樹，所以過程中我會以 node 稱呼每一筆資料：

1. 取出所有項目中當前出現次數最低的兩個 node x與 y
2. 建立一個新的 node z，這個 node的 left 與 right  child 分別為剛剛取出的 x與 y node
3. 將 x 與 y 的次數相加之後，賦予到 z 的次數  ( z.freq = x.freq + y.freq)
4. 原本的陣列中移除 x與 y node，並將新建立的 z 加入陣列中
5. 重複步驟1，直到所有資料皆完成

接著用展示一次建立流程：

取出陣列中次數最低的兩個 node x & y，並建立一個新 node z， z的 left 與 right child 指派為 x與 y，並將 x 與 y的次數加總後賦予到 z.freq

<img class="img-fluid" src="https://imgur.com/ZltssTo.png" />

接著移除陣列中剛取出的 x 與 y node，並將 z 新增至陣列中

<img class="img-fluid" src="https://imgur.com/qKEVVKt.png" />

接著重複相同的動作，取出陣列中次數最低兩個node，執行相同的動作，得到一個新的 z 頻率為 25

<img class="img-fluid" src="https://imgur.com/RnMd1LT.png" />

將原本陣列中b 與 c 移除，新增剛剛建立的 z至陣列中

<img class="img-fluid" src="https://imgur.com/s3jYDuE.png" />

再取出陣列中次數最少的兩個 node，建立 z 得到次數為 30

<img class="img-fluid" src="https://imgur.com/6CjqzRP.png" />

從陣列中移除 14 與 d ，將 z 新增至陣列中

<img class="img-fluid" src="https://imgur.com/Tav62co.png" />

接著取出 25 與 30 ，建立新的 z，次數相加後得到 55

<img class="img-fluid" src="https://imgur.com/6s7I6PL.png" />

將 25 與 30 自陣列中移除，將 55 新增至陣列中

<img class="img-fluid" src="https://imgur.com/E1aTm7B.png"/>

接著取出 a:45與 55，建立新的 z，得到 100

<img class="img-fluid" src="https://imgur.com/JxJ6S65.png"/>

接著將 a:45與 55自陣列移除，將 z 新增至陣列中，這時候因為陣列只剩下一個，**最後一個 node就是 Huffman tree的 root node**

<img class="img-fluid" src="https://imgur.com/uqLgVBT.png" />

最後可以建立起 variable-length codeword表

<img class="img-fluid" src="https://imgur.com/gJFUlVA.png" />

到這邊就建立起這組資料的 Huffman Tree了。

## 解題思維

瞭解Huffman Tree後，這邊想到的解題方式是先將 codeword表建立起來，透過 BST 的 traversal概念，遍歷整棵樹將codeword 表建立起來。

接著從傳入的 s 中逐一取出字元，根據prefix code提到的內容，所以不會找到模糊或是錯誤的 codeword，可以透過這方式找到所有編碼內容對應的 codeword，並還原原本內容。

這提 HackerRank上沒有Javascript選項，所以選擇 Java程式來實做

```java
  Map<String, String> codeMap = new HashMap<>();
	void decode(String s, Node root) {
        // Generate code to char mapping
        generateCodeMap(root.left,"0");
        generateCodeMap(root.right,"1");
        // Strin to Queue
        List<String> list = new LinkedList<>();
        for(int i=0; i<s.length(); i++)
            list.add(s.charAt(i) + "");

        String code = "";
        while(list.size()>0){
            code+=list.remove(0);
            if(codeMap.get(code) !=null){
                System.out.print(codeMap.get(code));
                code = "";
            }
        }
    }

    /**
    * Traverse all leaves and generate code map
    */
    private void generateCodeMap(Node node, String code){
        if(node ==null)
            return;
        if(node.left ==null && node.right == null){
            this.codeMap.put(code, node.data + "");
            return;
        }
        
        generateCodeMap(node.left, code + "0");
        generateCodeMap(node.right, code + "1");
        
    }
```

## 結語

大概大學畢業後就沒看到 Huffman Tree的東西了，直到做到這提題目後才跑回去找演算法中的 Huffman Tree內容。

了解 Huffman Tree後，這個題目的難度就相對降低許多，也當作是複習演算法吧(雖然演算法書中一堆證明定理內容我都跳過不看就是了XD)