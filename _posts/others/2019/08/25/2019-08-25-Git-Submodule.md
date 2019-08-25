---
title: Git - submodule 使用方式
layout: post
categories: Git
tags:
- git
- submodule
---

關於 git submodule 基本使用方式筆記。

submodule 方便在不同 git專案中重複使用其他 git 專案之程式碼，可降低重複撰寫程式碼之問題，加快專案開發。

<!--more-->

## 新增 submodule

```git
git submodule add :repo
```

執行指令後會看到目錄下多了 submodule目錄

接著透過下列指令對 submodule初始化，會從 repo將 submodule程式clone回來

```git
git submodule init

# 如果 submodule中還有 submodule
git submodule update --init --recursive

# 單一 submodule update
get submodule update
```

submodule也同樣是一個 github專案，所以切換到 submodule目錄下，也可以透過 git status查看狀態，與一般的 git專案沒兩樣，在修改 submodule目錄程式之後，也可以 push回 remote。

在修改 submodule目錄內容後，需要回到主目錄更新專案 submodule參考資訊。



## 更新 submodule

在主目錄下，可以透過下列指令更新

```git
git submodule update --remote
```



## submodule HEAD detach 解決方法

在更新 submodule後，進入 submodule目錄下可能會看見 HEAD detach 訊息，這個情境下，**如果當下有調整過 submodule目錄下程式，一定要小心操作，否則可能會遺失程式碼**，情境有三種：

1. 未修改 submodule目錄出現 detach解決方式

```
git checkout master
```

直接切換回submodule master branch

2. 修改過 submodule目錄下程式後，出現 detach訊息解決方式

```git
# 建立一個臨時 branch
git checkout -b :branchName

# 切換回 master branch
git checkout master

# 合併 branch
git merge :branchName
```

3. 要從 origin 上同步 submodule回來時

```git
git submodule update --remote --merge
or 
git submodule update --remote --rebase
```



##  移除 submodule

```
# 清除指定 submodule目錄下內容
git submodule deinit :submoduleName

# 清除 submodule時本機留一份 cache
git rm --cached :submoduleName

# 清除 submodule時本機不留 cache
git rm :submoduleName

# 清除 .git 目錄下 submodule 目錄
rm -rf .git/modules/:submoduleName

# 加入移除 submodule之變更
git add .
git commit ...

# 最後移除空目錄即可(如果目錄仍存在)
```















