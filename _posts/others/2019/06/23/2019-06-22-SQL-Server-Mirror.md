---
title: SQL Server 新增 Mirror DB筆記
layout: post
categories: SQL Server
unsplashTag: 'code,database'
tags:
- SQL Server
- Mirror


---

總覺得這不會是最後一次設定SQL Server之鏡像功能，這次過程中花費不少時間在處理設定面的問題，故記錄下來以便未來回顧。

<!--more-->

## 前言

SQL Server現在提供了數種資料庫同步機制，鏡像屬於其中一種，主要是讓資料庫寫入與更新的資料可以同時更新到不同的SQL Server主機上，讓資料同時存在多台SQL Server上，降低SQL Server出問題或主機出問題時，資料損失的量。

SQL Server目前應該有更好的機制，這次我使用到的是鏡像的功能，故針對鏡像的設定與操作做一個記錄。

## 步驟


### 1. 前置作業

1. 兩台相同版本之SQL Server
2. 兩台主機需要有**相同Windows 登入帳號與密碼**
3. 確定SQL Server 與 SQL Server agent 以同樣登入帳號執行
4. 確保主機上防火牆允許 5022 port 通過


### 2. 權限檢查

檢查服務設定，確定以相同帳號執行SQL Server與SQL Server Agent

檢查兩台主機以之SQL SERVER與SQL SERVER AGENT以相同帳號密碼執行，帳號需要透過旁邊『瀏覽(B)』按鈕來找到對應帳號，選擇對應帳號後會自動帶入。

SQL Server：  

<img  class="img-fluid" src="https://imgur.com/l5md9ii.png"/>

SQL Server Agent：  

<img  class="img-fluid" src="https://imgur.com/pVBVJ4E.png"/>



### 3. 資料還原

1. 從主體資料庫(Principal) **<u>完整備份資料庫</u>**與**<u>交易紀錄</u>**，並還原至鏡像資料庫(Partner)，使用SQL 查詢，並執行下列語法，會輸出backup.bak與backup.trn 檔案：

```sql
-- Full Backup
BACKUP DATABASE YOUR_DB_NAME to DISK=N'PATH_TO_FILE\backup.bak'
-- Transaction Log Backup
BACKUP LOG YOUR_DB_NAME to disk=N'PATH_TO_FILE\backup.trn'
```

2. 接著連線至鏡像主機，依序執行下列步驟

   1. 將資料庫還原至鏡像資料庫  

   <img  class="img-fluid" src="https://imgur.com/npw2mrI.png" width="400" height="400">
   
   2. 選擇檔案來源  
   <img  class="img-fluid" src="https://imgur.com/vjBuleg.png" width="600" height="600"/>
   
   3. 必須選擇 <u>RESTORE WITH NORECOVERY </u> 
   
   <img  class="img-fluid" src="https://imgur.com/RBdWe1W.png" width="600" height="400"/>
   
   4. 還原後，資料庫上會顯示**正在還原**  
    <img  class="img-fluid" src="https://imgur.com/D1UnNMR.png" width="400" height="400"/>
   5. 還原交易紀錄  
       <img  class="img-fluid" src="https://imgur.com/UVO22uP.png" width="600" height="400"/>
   6. 選擇來源  
       <img  class="img-fluid" src="https://imgur.com/LxpiaIT.png" width="600" height="400"/>
       
   7. 還原選項中必須選擇RESTORE WITH NORECOVERY  
       <img  class="img-fluid" src="https://imgur.com/TyIZv3Q.png" width="600" height="400"/>

### 4. 鏡像設定

1. 至主體資料庫(Principal) ，選擇鏡像  
      <img  class="img-fluid" src="https://imgur.com/YhDE88y.png" width="500" height="300"/>
      
2. 選擇設定安全性  
      <img  class="img-fluid" src="https://imgur.com/a2WqkKP.png" width="500" height="400"/>
3. 選擇下一步  
      <img  class="img-fluid" src="https://imgur.com/iIMDnlg.png" width="500" height="400"/>
      
4. 因為這次鏡像沒有見證(Witness)主機，故選擇否，並按下一步  
      <img  class="img-fluid" src="https://imgur.com/ycJthoK.png" width="500" height="400"/>
      
5. 確認主體資料庫資訊與使用之Port號  
      <img  class="img-fluid" src="https://imgur.com/yCTlzfa.png" width="500" height="400"/>
      
6. 設定鏡像資料庫主機使用之Port號與連線資訊，選擇好鏡像資料庫主機後點選連接，確保可憐至鏡像資料庫中  
      <img  class="img-fluid" src="https://imgur.com/wl0r1fZ.png" width="500" height="400"/>
      
7. 若未加入DOMAIN，則直接選擇下一步  
      <img  class="img-fluid" src="https://imgur.com/W1eebjh.png" width="500" height="400"/>
      
8. 最後確認資訊  
      <img  class="img-fluid" src="https://imgur.com/5Cgeh2n.png" width="500" height="400"/>
      
9. 按下完成後，會開始建立鏡像設定  
      <img  class="img-fluid" src="https://imgur.com/jgOcdFZ.png" width="600" height="500"/>
      
10. 等待設定完成  
      <img  class="img-fluid" src="https://imgur.com/Xe1GCcG.png" width="600" height="500"/>
    
11. 完成後，會詢問是否啟動鏡像，直接選擇啟動即可  
      <img  class="img-fluid" src="https://imgur.com/DuStyKZ.png" width="600" height="400"/>
    
12. 如果出現這個錯誤，表示當下使用的SSMS可能是 2016版本，這是一個存在於SSMS 2016 版的 bug，可以改使用其他版本，如SSMS 2014來執行就可以解決，或是改以指令方式執行即可。  
      <img  class="img-fluid" src="https://imgur.com/WGC9IY2.png" width="600" height="400"/>
    
13. 以指令方式執行，必須先至鏡像資料庫上輸入下列指令  
```sql
ALTER DATABASE YOUR_DB_NAME SET PARTNER = 'TCP://PRINCIPAL_DB_IP:5022'
```

14. 接著至主體資料庫上輸入下列指令   
```sql
ALTER DATABASE YOUR_DB_NAME SET PARTNER = 'TCP://MIRROR_DB_IP:5022'
```

15. 設定完成後可以看見資料庫上之訊息  
主體資料庫：  
      <img  class="img-fluid" src="https://imgur.com/Sr3zO1L.png" width="300" height="200"/>

鏡像資料庫：  
      <img  class="img-fluid" src="https://imgur.com/tkdVgRJ.png" width="300" height="200"/>
    

16. 鏡像設定檔會存在於**伺服器物件 -> 端點 -> 資料庫鏡像**  
      <img  class="img-fluid" src="https://imgur.com/ckBFHwa.png" width="300" height="300"/>

到這邊基本上已經設定完成，如果有遇到其他錯誤代碼為 1418 之錯誤，可參考下列連結之建議處理：  
[Suggestions for 1418](https://blog.sqlauthority.com/2010/01/11/the-server-network-address-tcpsqlserver5023-can-not-be-reached-or-does-not-exist-check-the-network-address-name-and-that-the-ports-for-the-local-and-remote-endpoints-are-operational-microso/)  
[How to resolve Error: 1418 in sql server while mirroring](https://stackoverflow.com/questions/11032937/how-to-resolve-error-1418-in-sql-server-while-mirroring)  

## 總結

不得不說，設定中最痛苦的就是找 1418這個代碼代表的問題與排除，更痛苦的是明明照著步驟作卻沒辦法設定成功，試了許久才查到原來是最新的SSMS功能有bug，用舊版的就沒問題，這邊實在耗掉太多時間，若不是幸運查到文章，現在可能還在悲劇之中。