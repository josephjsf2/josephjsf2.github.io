---
title: LeetCode - Longest Palindromic Substring
layout: post
categories: data structure and algorithm
unsplashTag: 'code,algorithm'
tags:
- Palindromic
- Palindromic Substring
- Longest Palindromic Substring
- Manacher Algorithm
- data structure
- algorithm


---

這一篇文章拖欠了半年之久，當時是因為解 Leetcode寫到 [Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/)，才接觸到 Manacher Algorithm，不過當時看網路文章看了許久，一直沒弄懂演算法的思路，大概也是不想面對所以拖欠許久，直到半年後才回來重新看這個演算法，花了幾天時間仔細研究，終於看懂原理與思路，這邊把心得記錄下來。

<!--more-->

## 介紹

Manacher Algorithm 可以使用接近線性時間 O(N) 來找出字串中最長的迴文，充分利用了迴文的特性，可以大幅降低時間複雜度。

## 迴文 Palindrome

意思是一個字串，如果符合由左至右或是由右至左排序，順序皆相同的即為迴文，左邊與右邊文字剛好成為鏡像。

如 abccba或是 abcba兩種皆是迴文。

這次遇到的 Leetcode題目不是判斷字串是不是迴文，而是要找出字串中最長的子迴文字串，所以在解題之前需要先知道迴文的概念。

## 最初解題思維

腦中只有暴力解，意思是把整個字串跑一輪，逐個字往兩側擴展，逐一找出最長的迴文，不過找的時候遇到一些麻煩的問題，像是babbad 與 babad 兩個都兩個字串中都有迴文，但是 abba 的中心在中間，寫程式時需要額外判斷，因此我最初作法是對字串中每一個字做下面兩種檢查，整體做下來，會有兩個迴圈，時間複雜度是 O(N^2)。

<img class="img-fluid" src="https://i.imgur.com/EiVU5yh.png"/>

<img class="img-fluid" src="https://i.imgur.com/rYgSM9t.png" />



## Manacher Algorithm

因為一開始的作法沒效率，最後找到最多人針對這提討論的演算法，其概念與 KMP 演算法有點像，會用一個陣列來儲存目前算出來的每位置的迴文半徑，並且搭配迴文『鏡像』的特性，在過程中避免重複的運算。

### 前處理

在開始使用 Manacher Algorithm 前，會先做一個前處理，前面有看到 babbad 與 babad 兩種字串會需要較麻煩的判斷，所以在使用 Manacher Algorithm 之前，會在字串前後與每個字之間插入一個完全不相關的字元(一定要是字串中不存在的字，否則可能會出錯)，這邊選擇用 # 符號。

如 babbad 會轉變為 #b#a#b#b#a#d#； babad 轉變為 #b#a#b#a#d#

這個情況下，字串的長度會變為 2 * N +1

透過這個方式，可以**解決迴文中心在兩個字元中間**的問題，但使用的符號一定要是字串中沒出現過的字元，如果一個字串為 ab#ba的情況，前處理時同樣使用 # 符號作前處理，會變為 #a#b###b#a#，雖然一樣可以找出最長的迴文長度，但是很難知道最長迴文字串是什麼，因為不知道要清除哪一個 # 符號。



### 記錄每個位置最長迴文長度

這邊使用陣列 **LPS** 來儲存字串中每個字的**最長迴文長度『半徑』**，上面的 babbad 與 babad 兩字串在算出各個字元位置的最長迴文半徑後，分別記錄下來 ( 有些文章會設定迴文半徑最少是 1，但這邊設定為 0，實際上不會影響運算。)

<img class="img-fluid" src="https://i.imgur.com/oKlzh7R.png" />

從這邊觀察結果，可以知道 babbad 字串最長迴文半徑位置在 index-6 的位置，可以得到最長迴文字串為#a#b#b#a#，只要把 # 移除，就可以得到長度為 4 的最長迴文 abba。

同樣的，字串 babad 最長迴文半徑分別在 index-3與 index-5 的位置，移除 # 後分別為 bab與 aba，長度分別為 3。

在找尋最長迴文字串時，時間最多會花在建立起 LPS 陣列，因為要逐個往外擴展比較，而 Manacher Algorithm 最主要是改良了過程的計算，避免不必要的運算。

### 迴文的鏡像特性

這個部分我自己認為是 Manacher Algorithm 最重要的部分，因為迴文擁有鏡像的特點，因此在算出某一個字元的迴文半徑後，在計算下一個字元時，可以利用之前算出來的結果來加快運算。這邊舉一個例子：

在開始前需要先提到，在計算完字串中一個字元後，可以找出一個字元的迴文區間，會有對應的**<u>迴文的左邊界與右邊界</u>**，而計算的的字元稱為**<u>迴文的中心</u>**。假如目前中心是 index-6， LPS[6]是4，那麼以 index-6 為迴文的字串範圍會是 6-4 ~ 6+4 = 2 ~ 10，而 **<u>2 就是目前迴文的左邊界， 10 是目前迴文的右邊界</u>**

**只要目前計算的迴文右邊界超過原本的迴文右邊界，就需要更新目前迴文的中心，與迴文的邊界資訊**。

使用babbad做為例子，目前已經找出 index-6迴文半徑是 4 ，所以我們可以知道 index-6 迴文字串位置介於 6 -4 ~ 6+4的位置，也就是陣列 2 - 10 的區間，下一個要計算 index-7的 LPS。

<img class="img-fluid" src="https://i.imgur.com/Nbf6zxN.png"/>

要計算 index-7 時，可以看 index-7 鏡像位置的 LPS數值，可以從鏡像位置的數值得到一些資訊

要知道 index-7的鏡像位置前，我們首先可以知道  index-7 - index-6 = index-6 - index-7-mirror， index-7-mirror是 index-7的鏡像位置，因使從這邊可以推斷出 index-7-mirror = 2 * index-6 - index-7，從基本的數學運算搬移得知的結論，這個就是網路上一些文章中寫的 mirror = 2 * C - i 

所以這邊，可以知道 index-7-mirror = 2 * 6 - 7 =5， index-7的鏡像位置在 index-5。

同理，如果現在在計算 index-8，那麼 index-8的鏡像位置為 2 * index-6 - index-8 = 4。

<img class="img-fluid" src="https://i.imgur.com/bXJxJyl.png"/>

### Case 1

接著就是 Manacher Algorithm的 case 1：如果 **mirror 所在位置為中心**的迴文字串範圍**沒有碰到**目前迴文的左邊界(不能包含左邊界)，**而且 i <右邊界**，那麼這個情況 index-i 的 LPS會等於 index-i' 的 LPS。

這段話我直接用圖片來展示，首先參考下面的圖片：

<img class="img-fluid" src="https://i.imgur.com/ZIwVUni.png"/>

圖片中紅色的區塊， index-i'在這邊是 index-5，其迴文的範圍是 5 -1 ~ 5 +1，也就是 4 - 6的區間，而目前中心點 index-6 的迴文區間為 2 - 10，index-5迴文的左邊邊界沒有交疊到index-6迴文的左邊界， 所以這個情況滿足 case 1。

這個情況下， index-7 的 LPS會等同於 index-5 的數值，也就是 1。

原因是因為鏡像的特性，以 index-6為中心的迴文，左右會成為鏡像，只要**迴文的範圍不碰到與不超過 index-6 的左右邊界**，可以知道左右的結果一定會相同。如果是碰到或是超過的情況，則屬於下一個 case 2的情境。

**而 index-7的迴文範圍沒有超過 index-6的右邊界，所以會繼續沿用以 index-6為中心迴文的左右邊界**。

所以再下一個，要找出 index-8，同樣符合 case 1，所以可以得到 LPS[8] 是 0。

**而 index-8的迴文範圍沒有超過 index-6的右邊界，所以會繼續沿用以 index-6為中心迴文的左右邊界**。

<img class="img-fluid" src="https://i.imgur.com/KAt7EpS.png"/>

###  Case 2

Manacher Algorithm的 case 2：如果 **mirror 所在位置為中心**的迴文字串範圍**碰到**目前迴文的左邊界或超過目前的左邊界，**而且 i <右邊界**，那麼這個情況 index-i 的 LPS 會是 min( LPS[mirror], 右邊界位置 - index-i )，**這邊取得的數值是 index-i 最少擁有的長度**，不是最終數值，接著還要嘗試去檢查目前迴文是否可以再更長。

要解釋這個情況，這邊分別計算 index-9 與 index-10來看

首先是 index-9，可以知道 index-9 的鏡像位置為 2 * 6 - 9 =3，而 index-3 的迴文範圍為 3 -3 ~ 3 + 3 = 0 ~ 6。

<img class="img-fluid" src="https://i.imgur.com/CjuHady.png"/>

這邊可以知道， index-3 的範圍超過了目前的左邊界範圍，在這個情況下， index-9 不能直接複製 index-3的數值，而是取用 min( LPS[3], index-6右邊界位置 - 9) = min(3, 10-9) = min(3, 1) = 1

這邊會得到 LPS[9] =1，這並不是最終的結果，得到的 1 只是從前面計算的結論得出 index-9 因為鏡像的特性，至少會有 1 的迴文長度，後續需要嘗試再繼續計算才能知道結果。

這邊使用最小值的原因，主要是因為我們只能保證左右邊界內的迴文內容可以從先前的結果推論出可能的迴文長度，如果超過邊界的部分就沒辦法保證。

<img class="img-fluid" src="https://i.imgur.com/lYCXciy.png"/>

<img class="img-fluid" src="https://i.imgur.com/nLmI7BN.png"/>

所以在計算 index-9時，會使用 min( LPS[3], 10 -9) = min(3, 1) = 1，原因就是這樣。

得到 LPS[9] = 1 還沒結束，接著要是著擴展到右邊界外，看超過右邊界的資料是否符合

所以接著要計算 index-9 +2 與 index3 - 2 是否相同( b != d)，這邊不同，所以 index-9 等於 1

**而 index-9的迴文範圍沒有超過 index-6的右邊界，所以會繼續沿用以 index-6為中心迴文的左右邊界**。

接著計算 index-10，按照相同的方式算出 index-10 鏡像位置位於 index-2，而 index-2的迴文區間剛好在左邊界上，所以 LPS[10] 是 min( LPS[2], 右邊界 - index-10) = min(0, 10-10) = 0，這邊得到 LPS[10]後還不一定是最終結果，接著要繼續計算 index-11 是否等於 index-1，結果是不相同，所以 LPS[10] = 0

<img class="img-fluid" src="https://i.imgur.com/PU2Hhi6.png"/>

**而 index-10的迴文範圍沒有超過 index-6的右邊界，所以會繼續沿用以 index-6為中心迴文的左右邊界**。

這邊再探討一個情況，如果將字串稍微做一個修改，將字串改為 aabbad，這個情況下取用min(LPS[i'], 右邊界 - index-i) ，跟前面一個不同，這次的 LPS[ i' ]不等於 0，但因為在邊界上，所以會推出 LPS[i]=0，接著再繼續嘗試擴充 LPS[i] 長度。

<img class="img-fluid" src="https://i.imgur.com/m6GLHZf.png"/>



### 更新迴文中心與邊界

在前面討論 case 1 與 case 2 時最後都會寫到，目前 index-i 為中心的迴文區間沒有超過右邊界，所以不改變迴文區間與迴文中心，

接下來計算 index-11時，因為 index-11 超過目前 index-6的右邊界，所以沒辦法使用前面推斷出的 LPS 來計算，只能從 0 開始 以 index-11為中心，逐漸往外擴展來找迴文範圍，這邊會得到 LPS[11] = 1。

<img class="img-fluid" src="https://i.imgur.com/CrUMbzg.png" />

這時候 index-11的迴文區間超過目前的右邊界，所以需要更新迴文中心與左右邊界，更新後如下圖：

<img class="img-fluid" src="https://i.imgur.com/vCkx0vG.png"/>

後續運算即與前面相同， case 1與 case 2，接著更新迴文中心與邊界，步驟大致如此。

### 最長迴文長度

因為在每個字元間都插入一個 #符號的緣故，字串會變為 2 倍，而 LPS雖然在前面是說迴文的半徑，但實際上 LPS 陣列中的數值就是迴文的長度，如 babbad的 LPS中最大數字為 4，即代表了字串中最長的迴文長度為 4。

<img class="img-fluid" src="https://i.imgur.com/aPAFqVp.png"/>

## 程式碼

```java
class Solution {
    public String longestPalindrome(String s) {
        // 前處理，在字元間加入 #
        char[] p = new char[ 2 * s.length() + 1 ];
        p[0] = '#';
        for(int i=0; i<s.length(); i++){
            p[ 2 * i + 1 ]=s.charAt(i);
            p[ 2 * i + 2 ] = '#';
        }
        // LPS陣列
        int[] lps = new int[2 * s.length() +1 ];
        
        // 目前使用的迴文中心
        int center =0;
        // 目前迴文中心的右邊界
        int centerRight = 0;
        
        int maxLPSIndex= 0;
        int maxLPS = Integer.MIN_VALUE;
        
        for(int i=0; i< p.length; i++){
            // mirror position
            int mirrorIdx = 2 * center - i;
            
            if(i <centerRight){
                lps[i] = Math.minㄋ(lps[mirrorIdx], centerRight -i);
            }
            
            // 嘗試擴充目前的迴文長度
            while( 
                i+lps[i] +1  < p.length 
                && i-lps[i] -1 >=0 
                && p[i + lps[i] +1] == p[i-lps[i] -1] ){
                
                lps[i]++;
            }
            
            // 如過i的迴文區間超過右邊界，則更新 center與 centerRight
            if(i + lps[i] > centerRight){
                center = i;
                centerRight = i+lps[i];
            }
            
            if(lps[i] > maxLPS){
                maxLPS = lps[i];
                maxLPSIndex = i;
            }
  
        }
        
        // 移除 # 並找到最長迴文字串
        String str = new String(p);
        str = str.substring(maxLPSIndex - maxLPS, maxLPSIndex + maxLPS +1);
        return str.replace("#", "");
    }
}
```



## 結果比較：

因為這提是在 Leetcode上遇到的，所以可以比較一開始的寫法與使用 Manacher Algorithm寫法的效率差多少：

後面反覆提交了幾次結果都一致，大概就是 6ms 與 38ms 的差異，不過在 38 ms結果統計是在效率較差的一群，在 6ms 則是在效率較好的一群，同樣也是參考就好。

<img class="img-fluid" src="https://i.imgur.com/206k80f.png"/>

## 結語

這個演算法對我來說很燒腦，大概過半年以上再回來就需要再想一陣子，希望作這些筆記對未來複習會有幫助。過程中參考不少的文章，也相當感謝願意在網路上分享學習心得的所有伙伴。



參考資料：

[Manacher Algorithm](https://en.wikipedia.org/wiki/Longest_palindromic_substring)

[https://www.geeksforgeeks.org/manachers-algorithm-linear-time-longest-palindromic-substring-part-1/](Manacher's Algorithm – Linear Time Longest Palindromic Substring – Part 1)

[Manacher’s Algorithm Explained— Longest Palindromic Substring](https://medium.com/hackernoon/manachers-algorithm-explained-longest-palindromic-substring-22cb27a5e96f)

[[演算法] Manacher’s Algorithm 筆記](https://medium.com/hoskiss-stand/manacher-299cf75db97e)

[Manacher's Algorithm 迴文演算法介紹](https://havincy.github.io/blog/post/ManacherAlgorithm/)

[Manacher 马拉车算法](https://blog.crimx.com/2017/07/06/manachers-algorithm/#%E6%9C%B4%E7%B4%A0%E8%AE%A1%E7%AE%97%E6%96%B9%E6%B3%95)

