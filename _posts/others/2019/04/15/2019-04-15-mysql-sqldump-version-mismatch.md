---
title: 遇到 mysqldump Version Mismatch問題解決方法
layout: post
categories: Mysql
unsplashTag: 'code,mysql,database'
tags:
- mysql

---

Mysql 執行 dump 時遇到的問題，直覺未來還是會遇到，所以在此做一個記錄

<!--more-->

因為自己用 MySql Workbench 8，Mysql 版本為5，不確定是不是因為這原因造成的，在export data時出現了這個訊息

<img  class="img-fluid" src="https://imgur.com/JM1drmV.png"/>

解決方法很簡單，在MySql workbench上找到 Edit -> Preference

<img  class="img-fluid" src="https://imgur.com/bBHUIkM.png"/>



這邊可以指定 sqldump tool的路徑

<img  class="img-fluid" src="https://imgur.com/ROC6bxr.png"/>



可以選擇要用哪一個版本的 tool來執行 sqldump，所以可以將路徑切換到 mysql 5的目錄下，可以找到sqldump.exe，選擇好後重新執行sqldump功能，即正常運作

<img  class="img-fluid" src="https://imgur.com/Lw7giWs.png"/>



