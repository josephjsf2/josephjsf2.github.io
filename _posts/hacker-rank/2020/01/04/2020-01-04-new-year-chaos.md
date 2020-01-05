---
title: HackerRank - New Year Chaos
layout: post
categories: HackerRank
unsplashTag: 'code'
tags:

- new year chaos
- hacker rank
- algorithm

---

久久沒解題，結果腦袋在解題過程中卡死。

<!--more-->

# 題目：New Year Chaos

## 題目說明

這是 HackerRank上的題目，題目相當長，可以直接到網站上參考題目資訊：[HackerRank](https://www.hackerrank.com/challenges/new-year-chaos/problem)

題目意思概略說明：

一開始陣列中會有多個人排隊，並且每個人都領有號碼牌，從第一個人1號到最後 N-1 號。每個人都可以賄賂前面的人，藉此來讓自己的位置往前，但是保有自己原本拿到的號碼牌(即便交換位置號碼牌也不會交換)，這邊規定每個人最多**只能往前賄賂2次**，不會有往後賄賂的情況發生。

題目大概如上描述，題目會給一個陣列，這個陣列是經過多次賄賂動作後的結果，要計算出從最一開始到最後結果**<u>最少</u>**需要經歷過幾次賄賂的動作。

## 最初解題思維

最初看完題目後，想到的就是氣泡排序法(Bubble sort)，透過氣泡排序法的概念來計算出過程中發生過幾次賄賂的動作。

但是最後解完後提交， 10個 測試中只通過 6 個，有4個測試因為數值太多，導致測試 timed out，表示有更有效率的寫法，苦戰一陣子，還是沒想出解法，參考了網路上大家的解題方式。



## 參考解法

這邊參考兩種解題思維，第一種解法是我最後找到覺得最簡單的解法，第二種解法一開始就找到了，但是因為看不是很懂解題的思維，所以一開始不採納這種方式解。

最後循著別人的方式解出來後，可以知道因為題目條件限制一個人最多只能往前賄賂兩次，所以可以知道一個人最多置多只能往前2個位置，一定要先有這個概念，這題解法特別需要用到這個概念。

### 解法一

以下列圖片序列來看， 5號的位置原本在 index-4，如果往前賄賂兩個人後，位置最多只能前進到 index-2，不可能再往前，如果有更往前的情況，則違反規則。

<img class="img-fluid" src="https://imgur.com/CjjRRB9.png"/>

上面情況來推論，可以得知原本在 index-i 上的內容，如果在有賄賂其他人情況下，最後位置可能會往前到 i-1 與 i-2的位置上。

這一個方法的概念是只要陣列上index-i 位置上內容不正確，則往前面元素找到正確的內容，並將原本屬於 index-i的內容換回來，這個方式必須從陣列最尾巴開始執行，概念就像是將所有過程的交換動作倒轉回去，故選擇從最尾巴開始執行。

當index-4的情況，因為 index-4 原本應該為 5號，目前為4號，所以會往 index-3與 index-2找尋 5號，找到後逐漸將 5號推回 index-4的位置，同時計算賄賂的次數。

<img class="img-fluid" src="https://imgur.com/HgwfPaU.png"/>

index-3 正確內容為 4號，現在陣列上 index-3 也是4號，處於正確位置，所以進行下一輪迴圈。

index-2 正確內容為 3號，現在陣列上 index-2 也是3號，處於正確位置，所以進行下一輪迴圈。

<img class="img-fluid" src="https://imgur.com/aDRU5bh.png"/>

index-1 正確內容為 2號，但陣列上目前為 1號，所以往 index-0找到 2號，因此與 index-1交換，讓2號回到 index-1，同時計算賄賂次數。

<img class="img-fluid" src="https://imgur.com/NC540s4.png"/>

index-0 正確內容為 1號，陣列上index-0為 1號，處於正確位置，所以不做處理。

做完全不後陣列內容也會回覆成最初使的狀態，變成排序過的結果。

<img class="img-fluid" src="https://imgur.com/gorW2Cp.png"/>

概念大致如上，因此寫成程式碼的大概是下面的樣子：

```javascript
function minimumBribes(q) {

    let count = 0;
    for(let i= q.length-1; i>=0; i--){
        const expectedVal = i+1;
        if(q[i] !== expectedVal){
            if(q[i-1] === expectedVal){
               	// 先檢查 i-1 可以避免 i數值為 -1 的問題，正確數值位於 i-1位置
                count++;
                swap(q, i, i-1);
            }else if (q[i-2] === expectedVal){
              	// 位於 i-2 位置
                count+=2;
                // Reverse order，將原本在第 i 個位置的元素依照順序交換回來
                swap(q, i-2, i-1);
                swap(q, i-1, i);
            }else {
                console.log('Too chaotic');
                return;
            }
        }
    }
    console.log(count)
}

function swap(arr, index1, index2){
    const temp = arr[index1];
    arr[index1] =arr[index2];
    arr[index2] = temp;
}
```

### 解法二

以這一題來看，解法二應該是最快的解法，且程式碼也相當簡短，第一個解法在過程中還需要做陣列元素交換，解法二可以省去這些步驟，讓程式運行更快速。這個解法我花了些時間驗證與思考才弄懂整個思路。

解法二的概念如下：

解法二的概念最主要來自於計算陣列中每一個位置 index-i 上的人共計接受過幾次賄賂，最後做加總即可求得總次數。

從下面圖片來看，這是最初的排隊列表，如果現在要計算 3號接受過的賄賂次數，則『賄賂過 3 號的人**最遠**的位置只能到 2 號的位置，意思是賄賂了 3 號與 2 號( 最多賄賂2次原則 )』，也就是『 **<u>賄賂過 index-i 的人最遠可以排到 index-(i-1) 的位置上</u>**』

<img class="img-fluid" src="https://imgur.com/6wDzfRW.png" />

先有了上面的概念後，接著來看下面的情況：

現在是一個最終的結果圖，如果要查 3 號 總共接受過幾次賄賂，可以這樣做：

1. 先找到 3 號原本的位置，我們可以直接推理得到 3 原本的位置在 index-2( 3-1=2)
2. 找到賄賂過3號的人最遠可以到達的位置，可以推理得到 index-1( 2-1 =1)
3. 如果 ( 3號最初位置 - 最後 3 號的位置) >  2 表示超過規定次數
4. 計算『賄賂過3號的人最遠的位置』到『現在 3 號前一個位置』間，比 3號數字還大的數字數量，即可算得 3 號總共被賄賂過幾次，可以直接從下圖來看會簡單些，用文字相當難理解這一段文字的意義。

在這張圖來看： 

1. 3 號現在位於 index-3 位置

2. 3 號最初在 index-2位置

3. 賄賂過 3 號的數字最遠可以到 index-1位置

4. 因為3 號最初位置 -  3 號最後位置 <=2，所以是符合規定的情況

5. 計算 index-1 到 index-2 間比 3號大的數字數量，即為 3 號接受賄賂的次數，在這邊可以算出是 1

<img class="img-fluid" src="https://imgur.com/dLMoJo4.png" />

再來一個情況：

1. 2號最初的位置為 index-1
2. 2號最後的位置為 index-4
3. 賄賂過2號的人最遠的位置為 index-0
4. 1 - 4  = -3  符合題目規定
5. 計算 index-0 到 index-3間比 2號大的數字數量，即為 2 號接受賄賂的次數，在這邊可以算出是 3

<img class="img-fluid" src="https://imgur.com/ziYnakO.png"/>

透過這個方式，就可以寫出最後的程式：

```javascript
function minimumBribes(q) {
    let count = 0;
    for(let i=0; i<q.length; i++){
        // 是否違反規則
        if( (q[i]-1) - i > 2){
            console.log('Too chaotic');
            return;
        }
        // 避免 index 為負值的情況
        for(let j = Math.max(0, (q[i]-1)-1); j<i; j++){
            if(q[j] > q[i])
                count++;
        }
    }             
    console.log(count)
}
```

## 結語

在第一種解法中，可以直接從程式碼中知道整個解題的思維，算是相當好理解的。第二個解法相當簡短，但通常這種愈短的程式反而愈難完全弄懂，光看程式碼沒有說明可能沒辦法徹底知道是用什麼概念來解決問題的，過程中參考了討論串中熱心的大神解說，才徹底弄懂第二種解法的原理，也是有相當大的收穫。

參考資料：

解法一： [[Medium] Minimum Swaps 2 (Hackerrank, javascript, arrays, sorting)](https://www.youtube.com/watch?v=Mk9Fre9_f64)

解法二：[Hacker Rank Discussion](https://www.hackerrank.com/challenges/new-year-chaos/forum)