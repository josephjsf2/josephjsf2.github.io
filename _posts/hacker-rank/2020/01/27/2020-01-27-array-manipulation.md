---
title: HackerRank - Array Manipulation
layout: post
categories: HackerRank
unsplashTag: 'code'
tags:

- Array Manipulation
- hacker rank
- algorithm

---



<!--more-->

# 題目說明

意思大概是依序對陣列一段連續區塊的數值作數值加總，在最後找出陣列中最大的數字。

如初使給定一個長度為  10 的陣列

第一組數值進來時給定 [1, 5, 3]，表示陣列中第一個到第五個數值都增加3

第二組數值進來時給定 [4, 8, 7]，表示陣列中第四個到第八個數值都增加7

第三組數值進來時給定 [6, 9, 1]，表示陣列中第六個到第九個數值都增加1

在最後求陣列中最大的數字，如同下圖呈現結果，最後的陣列中最大值為 10

<img class="img-fluid" src="https://imgur.com/g1yBXgS.png"/>

## 最初解題思維

這題很明顯的知道不能按照題目的意思去實做程式，但是腦中沒有更好的作法，只能乖乖的土法煉鋼把方法寫出來，結果雖然可行，但最後會因為運行時間過久而失敗。

程式的思路基本上與問題介紹中的思維是相同的，用一個陣列來儲存最後結果，過程中對給定的區間做數值增加，但是只要資料一多，程式效能問題就會浮現。這程式的時間複雜度大概是 O(N^2)

```javascript
function arrayManipulation(n, queries) {
    let result =[];
    for(let i=0; i<n; i++)
        result.push(0);

    for(let i=0; i<queries.length; i++){
        let start = queries[i][0]-1;
        let end=queries[i][1]-1;
        let qty = queries[i][2];
        for(let j=start; j<=end; j++)
            result[j]+=qty;
    }

    let max= 0;
    for(let i=0; i<result.length; i++){
        if(result[i] > max)
            max = result[i];
    }
    return max
}
```



## 程式調整

網路上找了一下後，彷彿看見新世界，這依題有一個方式可以讓時間複雜度降低到 O(N)，而且也不會太複雜。

思考方式是原本我在陣列中記錄所有數值結果，調整為陣列中 <u>第 i +1 筆相對於第 i 筆數值大多少 </u> 。

```javascript
// 初始值
[0,0,0,0,0,0,0,0,0,0]

// [1, 5, 3] => 實際結果變為[3,3,3,3,3,0,0,0,0,0]
[3,0,0,0,0,-3,0,0,0,0]
```

原本結果是 [3,3,3,3,3,0,0,0,0,0]，調整記錄方式為 [3,0,0,0,0,-3,0,0,0,0]

前面提到，陣列中每個值都會記錄 <u>當前資料相較於前一筆的差異值</u> ，所以從這個結果來看

第一筆是 3，表示初始數值

第二筆是0，表示第二筆與第一筆差異為 0，可以推得第二筆數值為 3 + 0 = 3

第三筆是0，表示第三筆與第二筆差異為 0，可以推得第三筆數值為 3 + 0 + 0 = 3

以此類推，

第六筆是 -3，表示第六筆比第五筆少 3 ，所以第六筆實際數值為 3+0+0+0+0-3 = 0

第七筆是 0，表示第七筆與第六筆差異為 0 ，可以推得第七筆實際數值為 0。

```javascript
// [1, 5, 3] => 實際結果變為[3,3,3,3,3,0,0,0,0,0]
[3,0,0,0,0,-3,0,0,0,0]

// [4,8,7] => [3,3,3,10,10,7,7,7,0,0]
[3,0,0,7,0,-3,0,0,-7,0]

// [6,9,1] => [3,3,3,10,10,8,8,8,1,0]
[3,0,0,7,0,-2,0,0,-7,-1]
```

從這個過程可以推出一些結論：

如果題目傳入 [a, b, q]數值

1. 因為 a - b 區間上要增量的值是相同的，所以 a- b 區間上的差異值不會變動，只有區間上第一個數值需要往上增量
2. 因為 a- b 區間數值增加，所以 b 後一個數值會相對於 b 來說會減少，所以 b 後一個元素必須相對的扣除 q 數值

因此當傳入  [1, 5, 3] 時現在要做的事情

1. 陣列第一個增加 3， 區間 1 - 5 之間差異值不會變動
2. 陣列第 (5 +1 ) 個 減少 3 (因為前面數值增加，第 6筆數值相對於第 5筆來說會再減少 3)

## 程式碼 

所以就可以上程式了：

```javascript
function arrayManipulation(n, queries) {
    let diff =[];
    for(let i=0; i<n; i++)
        diff.push(0);

    for(let i=0; i<queries.length; i++){
        let start = queries[i][0]-1;
        let end = queries[i][1]-1;
        let qty = queries[i][2];
        diff[start] +=qty;
        if(end <n)
            diff[end+1]-=qty;
    }

    let results =[0];
    let max = 0;
    for(let i=0; i<diff.length; i++){
        if(diff[i] + results[i] > max)
            max = diff[i] + results[i];
        results.push(diff[i] + results[i]);
    }
    return max
}
```

## 結語

印象中這個題型看到不少次，這一種方式來處理區間數值增量無疑的是一個很棒的方式，可以透過這個題目以新的方式來思考資料儲存方式也是相當值得。這提雖然是分類在困難題型，但是實際上最佳解的程式邏輯並不是太複雜，只是要想到這個方式來處理才是最困難的。

參考資料

[Logic used behind Array Manipulation of HackerRank](https://stackoverflow.com/questions/48162233/logic-used-behind-array-manipulation-of-hackerrank)

