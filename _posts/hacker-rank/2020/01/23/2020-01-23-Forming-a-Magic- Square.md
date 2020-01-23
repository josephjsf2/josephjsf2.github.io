---
title: HackerRank - Forming a Magic Square
layout: post
categories: HackerRank
unsplashTag: 'code'
tags:

- Forming a Magic Square
- hacker rank
- algorithm
- magic square

---
HackerRank上 Forming a Magic Square 問題解決方式。
<!--more-->

# 題目：Forming a Magic Square

## 題目說明

給定一個矩陣，要求找出置換矩陣數字後得到magic square的最小成本。

<img class="img-fluid" src="https://imgur.com/cxZOW3S.png"/>

## Magic square定義

簡單的定義：給定一個 n x n矩陣，矩陣中每個數字都是唯一的(不會出現重複的數字)，數值位於 1 ~ N^2，若填滿矩陣後可以滿足橫列、縱列與斜線數字加總結果相同，則稱為 magic square。

如果是 3 x 3 矩陣，則矩陣中只會出現 1 ~ 9數字

如果是 4 x 4矩陣，則矩陣中指會出現 1 ~ 16數字

## 解題思維

一開始看到題目，考慮過需要寫一個程式來把所有 magic square 結果找出來後再進行比對。後來參考到討論區的解法後，因為題目指明了  3x3 的矩陣，而 3x3 的所有解數量有限，所以可以先找出所有解，接著再逐一計算出最小 cost 值。使用這個方式解題程式複雜度會大幅下降，由原本需要找出所有解的解法，變成單純比對矩陣差異值。

要找出所有 3X3 magic square值，可以透過一些方式快速找出來，首先觀察矩陣：

第一種方式是翻轉矩陣，分別對矩陣旋轉 90 度，可以得到另外3組解，如下圖：

<img class="img-fluid" src="https://imgur.com/NbmKCMO.png" />

第二種方式是鏡像，對第一種解的矩陣以鏡像方式反射，可以得到4種不同解

<img class="img-fluid" src="https://imgur.com/YKoV6Wo.png"/>

到這邊總各得到了 8組解，將這找出來的8組解以陣列形式儲存，即可以拿來計算 min cost。

## 程式碼

過程中會將傳入的 s陣列轉為一維陣列，其餘沒什麼特別困難的地方。

撇除找到所有解的方式，這提難度應該不會太高。

```javascript
function formingMagicSquare(s) {
    // 所有解
    const sol = [
        [4,9,2,3,5,7,8,1,6],
        [4,3,8,9,5,1,2,7,6],
        [6,7,2,1,5,9,8,3,4],
        [8,1,6,3,5,7,4,9,2],
        [6,1,8,7,5,3,2,9,4],
        [2,9,4,7,5,3,6,1,8],
        [8,3,4,1,5,9,6,7,2],
        [2,7,6,9,5,1,4,3,8],
    ]
    // to 1-dem
    let dim1Source = [];
    for(let i=0; i<s.length; i++)
        dim1Source = [...dim1Source, ...s[i]]
    let minCost = Number.MAX_SAFE_INTEGER;
    for(let i=0; i< sol.length; i++){
        let cost = 0;
        let pSol = sol[i]
        for(let j=0; j<pSol.length; j++)
            if(pSol[j] !== dim1Source[j])
                cost += Math.abs(pSol[j] - dim1Source[j])
        if(minCost > cost )        
            minCost = cost;
    }
    return minCost;
}
```

## 結語

對於這樣的題目，為了求效率，這樣的方式我想也是可以被接受的，畢竟是用了最快的方式解決了問題。



參考資料：

[Hackerrank: Forming a magic Square](https://www.mathblog.dk/hackerrank-forming-a-magic-square/)

