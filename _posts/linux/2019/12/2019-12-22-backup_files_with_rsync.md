---
title: 使用Rsync進行檔案異地備份
layout: post
categories: linux
unsplashTag: 'code'
tags:

- rsync
- crontab
- centos
- linux


---

<!--more-->

## 前言

專案上需要做到將CentOS主機上檔案同步到其他CentOS主機做備援，得知當中可以使用Linux下的 Rsync來實做，變嘗試了一次，發現設定難度不高，所以做了一個簡單的筆記。

## Rsync 介紹

網路上都有介紹到，Rsync有點類似 cp或 scp指令功能，可以將目錄與檔案複製到另一個位置上，並且支援跨主機檔案傳輸與同步，傳輸過程支援壓縮功能。

Rsync 同步的機制僅有在第一次同步時需要以鏡像的方式同步到目的位置，後續的同步會檢查差異部分，僅同步異動的檔案，可以大幅省下同步的時間。

## 安裝Rsync 

Linux 預設應該已經有安裝Rsync軟體，如果沒有則可以透過下列指令安裝

```bash
sudo yum install rsync
```



## Rsync Server 端設定

首先調整Server端設定，允許 Client端可以同步檔案至Server端，或從Server端同步檔案回Client端

修改Rsync設定檔，在修改前建議先備份

```bash
sudo cp /etc/rsyncd.conf /etc/rsyncd.conf_backup
```

接著開啟檔案

```bash
sudo vi /etc/rsyncd.conf
```

調整設定檔上內容，情境如下：

第一個設定 SharedFolder 允許讓其他主機以 joseph帳號進行同步

第二個設定 FTP 允許讓其他主機以 ftpuser 帳號同步

第三個設定 www 允許任意任意存取

所以系統上共需要存在joseph 與 ftpuser帳號才可設定，並且於設定完後，建議於設定檔案時先將所有上面有列到的目錄都建立完成，否則會在執行時遇到一些不容易查的問題。

**如果要測試，建議先取一個設定來測試即可，這邊是為了做一些簡單測試才分為三組**

```properties
# 看說明不是很清楚，網路其他文章是寫啟動rsync server的帳號，與同步權限有關。測試後，設定為nobody時，允許client端同步目錄，但無法讓client端同步回Server端，會出現 permission denied訊息，除了設定帳號外，也可以設定為%RSYNC_USER_NAME% 表示當前的登入者，設在這邊的uid與 gid會套用到下方工作目錄區設定區塊(如果下方工作目錄區塊未設定)
uid = nobody     
# 同上
gid = nobody     
# 執行rsync 前會執行chroot指令到 "path" 路徑下
use chroot = yes   
# 最大同時連線數
max connections = 4   
# log 記錄位置
log file=/root/rsyncd.log  
# rsync pid檔案路徑
pid file = /var/run/rsyncd.pid  
# 同步時排除的檔案
exclude = lost+found/  
# 傳輸過程式否要輸出log
transfer logging = yes   
# 連線逾時時間，單位為秒
timeout = 900    
# 忽略沒有讀取權限檔案
ignore nonreadable = yes   
# 指定之檔案不再壓縮傳輸
dont compress   = *.gz *.tgz *.zip *.z *.Z *.rpm *.deb *.bz2  

[SharedFolder] 
	# 表示uid 會是登入者，如果這邊沒有設定uid資訊，則會套用到上方 nobody
	uid = %RSYNC_USER_NAME%
	# 該組設定的目錄路徑
    path = /home/joseph/SharedFolder    
    # 說明
    comment = Shared Folder    
    # 是否read only
    read only = no     
    # 允許連線的帳號，必須存在於linux 系統內
    auth users = joseph    
    #紀錄帳號密碼的檔案位置
    secrets file = /home/joseph/rsync/rsync.secret   
    # 記錄log 位置
    log file = /home/joseph/rsync/log/rsync.log   

[ftp]
	# 這邊沒設定uid或gid，故套用 nobody權限，所以client端沒辦法透過nobody帳號對FTPFolder目錄執行寫入權限(除非FTPFolder有開放)
	path = /home/ftpuser/FTPFolder
	comment = FTP file backup
	read only =no
	auth users = ftpuser
	secrets file = /home/ftpuser/rsync/rsync.secret
	log file = /home/ftpuser/rsync/log/ftp.log

[www]
	path = /home/ftpuser/public/www
	comment = public files no need to authenticate
	read only = yes
	log file = /home/ftpuser/rsync/log/www.log
```

<img class="img-fluid" src="https://imgur.com/4IEq36C.png" />

接續建立一個用來紀錄登入遠端主機的帳號與密碼檔，**密碼檔的 owner必須是 root，這些secret檔不可以讓 others擁有read權限**，否則執行時可能會出現錯誤

```bash
# SharedFolder設定
sudo vi /home/joseph/rsync/rsync.secret
# 檔案內輸入 joseph:your_password 即可
chmod 600 /home/joseph/rsync/rsync.secret # 更改權限

# ftp 設定
sudo vi /home/ftpuser/rsync/rsync.secret
# 檔案內輸入 ftpuser:your_password  即可

chmod 600 /home/ftpuser/rsync/rsync.secret # 更改權限
```

下列密碼皆為測試用

<img class="img-fluid" src="https://imgur.com/iOhYrlf.png" />

接著調整服務設定，設定rsync設定開機後自動啟用服務

```bash
systemctl enable rsyncd   
systemctl start rsyncd
```

Server端需要開啟防火強設定，允許Rsync連線至主機端，可以選擇開通 873 Port連線，或是允許Rsync程式連線：

```bash
firewall-cmd --permanent --add-service=rsyncd    # 防火牆上新增Rsync連線規則
firewall-cmd --reload	   # 重新讀取設定
```

最後是同SAMBA文章一樣，需要關閉SELinux設定，目前還不確定怎麼設定，如果沒關閉會無法使用

```bash
setenforce 0    # 暫時關閉
vi /etc/selinux/config   # 開啟SELinux設定

SELINUX=disable  # 將config內設定設定為disable，如此一來在後續重新啟動Linux時就會不會啟動SELinux
```

到這邊Server 端算就設定完成了



## Rsync Client 端設定

首先也需要有Rsync軟體，如果client端沒有安裝，則也需要先安裝才可

```bash
sudo yum install rsync
```

接著是client端會用到的設定

首先是要同步SharedFolder ，這邊會在 client端建立 Rsync與SharedFolder目錄，並於rsync目錄內建立一組密碼設定，目的是不要在每次同步時都手動輸入指令，而改以讀取密碼檔方式驗證

```bash
mkdir rsync  #用來存放rsync設定的目錄

vi ~rsync/SharedFolder.passwd
# 輸入Server端/home/joseph/rsync/rsync.secret joseph 設定之 密碼儲存即可

mkdir SharedFolder  # 要將Server端目錄資料同步至SharedFolder，名稱隨意即可
```

如下列圖片，SharedFolder中僅放入密碼資訊(測試用)

<img class="img-fluid" src="https://imgur.com/tSllRZh.png" />

接著建立FTP備份用目錄

```bash
mkdir FTPBackup  #備份FTP檔案之目錄

vi ~rsync/FTP.passwd
# 輸入/home/ftpuser/rsync/rsync.secret ftpuser設定之密碼儲存即可

```

同樣的FTP.passwd內也僅存放密碼檔(測試用)

<img class="img-fluid" src="https://imgur.com/joBmPJ8.png" />

最後建立一個用來同步 www 之目錄

```bash
mkdir www
```

到這邊 Client環境即準備完成。

## Rsync 同步測試

### 1. SharedFolder區塊測試

首先測試第一組 SharedFolder 同步，語法並不複雜，最單純為 **rsync src dest**

```bash
rsync 登入帳號@SERVER_IP_ADDRESS::設定區塊 同步目的位置
```

這邊使用的語法如下，會多加上一些參數：

```bash
rsync -arvzh joseph@SERVER_IP_ADDRESS::SharedFolder /home/joseph/SharedFolder/ --password-file=/rsync/SharedFolder.passwd --delete
```

這時候如果設定沒有錯誤，應該可以看見一些檔案同步的訊息

<img class="img-fluid" src="https://imgur.com/slveTwO.png" />

各個參數用途

>-a：archive mode，允許同步目錄或檔案
>
>-r：遞迴同步目錄
>
>-v：在同步時，rsync在同步時預設不會顯示任何資訊，在傳輸過程中會顯示更多資訊，如哪些檔案被同步與一些簡單的summary
>
>-z：壓縮傳輸
>
>-h：輸出較為容易閱讀的資訊
>
>--delete：加上這參數後，才會執行刪除的動作。如從Server同步至Client中，Client目錄多了tmp 目錄，tmp目錄在Server端不存在，如果有加上delete參數，則rsync過程會將 Client端的 tmp 目錄刪除。
>
>--password-file：密碼檔案的位置

除了將檔案同步回來，也可以將檔案同步至Server端，實際上，語法的差異只在調換了來源與目的的位置，其餘皆相同

```bash
rsync -arvzh /home/joseph/SharedFolder/ joseph@SERVER_IP_ADDRESS::SharedFolder --password-file=/rsync/SharedFolder.passwd --delete
```

這邊測試在Client端新增一個檔案，client_new_file.txt，接著執行rsync同步至Server

```bash
vi client_new_file.txt
# 輸入任意內容後儲存

rsync -arvzh /home/joseph/SharedFolder/ joseph@SERVER_IP_ADDRESS::SharedFolder --password-file=/rsync/SharedFolder.passwd --delete
```

可以看見檔案被同步回 Server端

<img class="img-fluid" src="https://imgur.com/2Trwfho.png" />

如果這時候將Server端 rsyncd.conf內 [SharedFolder] 區塊中的 uid資訊移除，接著在client端建立一個新的檔案同步至Server端，就會出現 Permission denied，因為登入是以 nobody帳號權限執行。

```bash
# Server端設定
vi /etc/rsyncd.conf
# 移除 [SharedFolder] 區塊中 uid
```

```bash
#Client端設定
vi new_client_file2.txt

rsync -arvzh /home/joseph/SharedFolder/ joseph@SERVER_IP_ADDRESS::SharedFolder --password-file=/rsync/SharedFolder.passwd --delete
```

<img class="img-fluid" src="https://imgur.com/pe2TQhy.png"/>

### 2. ftp區塊測試

測試與前面情境相同，嘗試將Server端檔案同步至Client端，參數設定都是一樣的

```bash
rsync -arvzh ftpuser@SERVER_IP_ADDRESS::ftp /home/joseph/FTPBackup/ --password-file=/rsync/FTP.passwd --delete
```

<img class="img-fluid" src="https://imgur.com/1N5fXtK.png" />

如果要將client端檔案同步回Server，目前設定會無法同步，如果有需要，則在 Server端 rsyncd.conf內 [ftp] 區塊加上 uid設定即可。

### 3. www 目錄測試

同步方式相同，但因為不需要認證即可存取，故可以減少一部份參數

```bash
rsync -arvzh SERVER_IP_ADDRESS::www /home/joseph/public/www --delete
```

<img class="img-fluid" src="https://imgur.com/glgg0FO.png" />

反過來測試從Client端同步至Server端，會發現沒有辦法寫入，這邊可能是因為 [www] 設定了read only=yes

<img class="img-fluid" src="https://imgur.com/HDGwlWV.png" />

如果希望有寫入權限，除了將 read only 設定為 no 以外，還需要將 uid 設為擁有寫入權限的帳號才可以(預設為 nobody)

如果設定了一個擁有寫入權限的 uid，卻沒有將read only設定為 no，同樣無法寫入

#### 這邊僅為了做測試才開啟這些設定，一般沒有用帳號密碼認證的強烈不建議這樣做，會增加主機上的風險！

<img class="img-fluid" src="https://imgur.com/DEsxRBV.png" />

<img class="img-fluid" src="https://imgur.com/8v2ap29.png" />

## 透過 crontab 定期執行同步

Crontab的設定可以參考網路上文章，這邊不做介紹，這邊可以設定透過 crontab來定期執行 rsync 指令

這邊情境為Client端每1分鐘同步一次 ftp 目錄檔案

首先建立一個 bash檔，把剛剛使用的指令複製貼上即可

```bash
vi rsync_task.sh

# 完成後記得賦予 x 權限
chmod u+x rsync_task.sh
```

rsync_task.sh內容如下：
```bash
#!/bin/bash
rsync -arvzh ftpuser@10.0.2.15::ftp /home/joseph/FTPBackup/ --password-file=/rsync/FTP.passwd --delete
```

接著編輯  /etc/crontab檔案

```bash
sudo vi /etc/crontab

# 輸入完成後儲存離開
* * * * * joseph /home/joseph/rsync/rsync_task.sh

# 重啟 crontab服務
systemcrl restart crond.service

# 可以檢查crontab服務是否有在執行
service crond.service status
```

<img class="img-fluid" src="https://imgur.com/61hJBKx.png" />

這時候在Service端有任何異動，則會每分鐘自動同步一次。

## 結語

其實設定Rsync不難，但設定檔中有些設定在最初實做時沒有特別去查用途，在做筆記時大多時間都是在查看每個參數的目的與用途，反而是最耗時間的，而且查了還有些沒把握是否正確，所以才搭配上多個測試，目的是希望後續回來參考文章時可以更快回憶起來。

## 參考文章

[CentOS 7.6 上安裝 Rsyncd 遠端檔案同步伺服器](https://blog.tomy168.com/2019/01/centos-76x64-rsync-daemon.html)

[Linux 使用 rsync 遠端檔案同步與備份工具教學與範例](https://blog.gtwang.org/linux/rsync-local-remote-file-synchronization-commands/)

[Rsync 基本設定教學 ](https://ithelp.ithome.com.tw/articles/10081360)

[rsyncd.conf](https://download.samba.org/pub/rsync/rsyncd.conf.html)

[Linux rsync command](https://www.computerhope.com/unix/rsync.htm)

