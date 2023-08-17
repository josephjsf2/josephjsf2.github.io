---
title: 透過SAMBA讓CentOS與Windows共享目錄
layout: post
categories: linux
unsplashTag: 'code'
tags:

- windows
- samba
- centos
- linux


---

第一次設定SAMBA，讓CentOS與Windows共享目錄，因此簡單做一個筆記紀錄。

<!--more-->

## 前言

最近專案上需要做到Windows 系統將自CentOS上進行檔案寫入與讀取，做了簡單的研究，做了點簡單筆記記錄如何做到Windows與CentOS做檔案共享。

Windows 電腦間要共享檔案可以透過網路的芳鄰來做到，其概念主要是透過CIFS/SMB協定來達成，而Unix系統上要分享檔案，則是透過NFS，如果要讓Windows與Unix共享，則可以透過Samba。

下面針對一些名詞作介紹，這邊簡單做一個整理：

**SMB(Server Message Block)**：最初由IBM提出，經過微軟大幅改良，主要目的為允許電腦間可以透過LAN對遠端主機檔案或目錄進行讀寫。

**CIFS(Common Internet File System)**：基於SMB而發展出來的協定，允許網路上的機器分享檔案目錄文件與印表機，同時支援授權認證，能讓Windows系統互相溝通。

**NFS(Network Fils System)**：在Unix上互相分享檔案的系統稱為NFS，NFS僅允許Unix機器互相溝通。

**SAMBA**：可以讓UNIX系統與Windows系統進行SMB/CIFS 檔案共享的自由軟體，後續版本中除了分享檔案目錄與印表機外，也可以整入Windows AD。

所以這次的需求主要就是透過SAMBA來處理。

## 安裝SAMBA

CentOS上安裝Samba

```bash
yum install samba samba-client samba-common -y
```



## 建立欲共享之目錄建立

建立 /home/joseph/SharedFolder 目錄，表示要開放 /home/joseph/SharedFolder目錄，允許透過Samba存取



## 設定

安裝完成Samba後，samba的設定檔位置路徑為  **/etc/samba/smb.conf**，接著設定samba設定檔

```bash
vi /etc/samba/smb.conf
```

smb.conf

```properties
[global]
	workgroup = SAMBA
	netbios name = SAMBA_NETBIOS
	server string = SAMBA SERVER
 
	log file = /var/log/samba/log.%m   
	max log size = 500    
	 
	security = user
	passdb backend = tdbsam

 
[ShareFolder]
	comment =Shared
	path = /home/joseph/SharedFolder
	browseable = yes
	writable = yes
 
	create mode = 0660
	directory mode = 2770
 
	valid users = root   # 必須是Linux上存在的user
```

smb.conf還有相當多的設定可以調整，這邊僅列出一部份，有其他更細節之設定，因為沒用到就沒提到了，其餘可以參考官方上的說明，有所有smb.conf設定：[smb.conf](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html)，設定完成後，**可以透過testparm 測試 smb.conf設定檔**

針對上面設定，這邊做一個簡單的紀錄

>workgroup：工作群組名稱
>
>netbios name：SAMBA netBIOS name
>
>server string：Server 描述說明
>
>log file：SAMBA log 位置，可傳入參數，如%m表示Client端 NetBIOS名稱
>
>max log size：表示SAMBA log檔最大SIZE，單位為KB，Samba會定期檢查並將超過大小的檔案作rename的動作，如加上 .old extension
>
>security：user 表示需要在主機上有帳號才可登入使用
>
>passdb backend：決定使用何種方式儲存user與group資訊，設定為tdbsam表示以TDB based password方式儲存，可傳入第二個optional參數表示passdb.tdb檔案欲存放位置，預設會在private目錄(*${prefix}/private*)
>
>​	如passdb backend = tdbsam:/etc/samba/private/passdb.tdb 
>
>​	預設路徑於 /var/lib/samba/private/passdb.tbd，private目錄為隱藏目錄，且必需要有足夠權限才可進入與檢	視
>
>
>
>[SharedFolder]：表示在網路芳鄰上顯示的名稱，smb.conf中可以存在多個 [ ]區塊，表示不同的分享目錄與設定
>
>comment：註解
>
>path：欲共享的目錄路徑
>
>browsable：控制是否在網路上顯示
>
>writable：是否可寫入，如果設為no，Client端將沒辦法對資料夾或檔案做寫入的動作
>
>create mode：從Client端建立檔案後，建立之檔案權限會設定為create mode的值
>
>directory mode：從Client端建立目錄後，建立之目錄權限會設定為directory mode的值
>
>valid users：可登入SAMBA的帳號白名單列表，可直接指定帳號清單，或是以 @開頭表示可使用的群組，指定的user一定要是系統上存在的帳號，否則是無效的
>
>​	如 root, @user 表示允許root帳號與 @user群組中的帳號使用



##  建立 Samba 存取帳號之密碼

前面smb.conf中 passdb backend設定為tdbsam，這邊透過smbpasswd建立對應帳號，建立的同時會一併設定登入密碼，資料會寫入 passdb.tdb 中

```bash
smbpasswd -a joseph
```



設定為開機啟動

```bash
systemctl enable smb
systemctl enable nmb
```



啟動SAMBA服務

```bash
systemctl start smb
systemctl start nmb
```



新增防火牆規則，允許外部可透過SAMBA連線至主機內存取目錄

```bash
firewall-cmd --permanent --zone=public --add-service=samba
firewall-cmd --reload
```

關閉SELinux(目前尚未能清楚知道關閉之原因，或是其實需要做什麼設定，受陷於目前不清楚SELinux，故只能先關閉)，目前未關閉SELinux情況下，Windows會沒權限無法存取SharedFolder內容

```bash
setenforce 0  # 暫時關閉，會於下一次登入時失效

vi /etc/selinux/config

SELINUX=disable #永久關閉SELINUX
```

到這邊基本設定即完成了，再來可以測試本機上 smb服務是否有啟動，可以檢視CentOS上445 port是否已被使用

可以使用任一熟悉的方式偵測，這邊使用**<u>lsof</u>**指令查看445 port使用資訊：

<img class="img-fluid" src="https://imgur.com/yT9HfNA.png" />

只要確認服務啟動後，即可切換至Windows系統準備連至CentOS。

## 在Windows上存取CentOS目錄

首先必須先查到 CentOS上的IP位置：

<img class="img-fluid" src="https://imgur.com/4f6uCcj.png" />

接著至Windows上，開啟『我的電腦』-> 『連接網路磁碟』

<img class="img-fluid" src="https://imgur.com/cXjMASH.png"/>



點選後，磁碟可以任意選擇，主要是 Folder 位置，必須使用 [UNC](https://www.lifewire.com/unc-universal-naming-convention-818230)路徑來指向CentOS目錄，在目錄最後是SAMBA [] 區塊內的名稱，前面設定 [SharedFolder]，故這邊使用 ShareFolder，接著可以點選完成。如果連線設定正常，SAMBA服務也正常設定，則會出現要求輸入帳號密碼的視窗

透過NETBIOS_NAME來找尋SAMBA server：

<img class="img-fluid" src="https://imgur.com/57U1JW6.png"/>

<img class="img-fluid" src="https://imgur.com/I6GtRej.png"/>

透過IP找尋SAMBA server：

<img class="img-fluid" src="https://imgur.com/xhpEewn.png" />

<img class="img-fluid" src="https://imgur.com/xbHNZZY.png"/>

在這邊輸入 smb.conf中設定的一組帳號還有設定的密碼，驗證過了後，即會看見硬碟被掛載到Windows上

<img class="img-fluid" src="https://imgur.com/GA0O3gU.png"/>

回到我的電腦，可以看見目錄被掛載在R磁碟上

<img class="img-fluid" src="https://imgur.com/XLEttdP.png"/>



## 測試

可以嘗試在Windows中新增一個檔案，新增一個HELLO_WORLD.txt，並指定內容

<img class="img-fluid" src="https://imgur.com/08g2LCn.png"/>

接著可以至CentOS上看，檔案確實被新增進來，且權限為 660

<img class="img-fluid" src="https://imgur.com/H58b0Ih.png"/>

<img class="img-fluid" src="https://imgur.com/iFiFOyc.png"/>

如果是建立目錄，則權限會是 2770

<img class="img-fluid" src="https://imgur.com/GP0BjwX.png"/>

同樣的，在CentOS上新增也會立即反應至Windows上所見項目，因為存取的是同一個位置。

## 結語

一開始覺得建立SAMBA是有困難的，中間比較過許多文章，發現難度其實不高，smb的設定相當多，有許多是未探索的設定，可以自行試試看，記得有看到整入AD或是LDAP做SAMBA登入，因步驟比較複雜這邊就沒特別研究了。

這編寫一篇筆記，也是幫助未來遇到時可以更快速的找回遺失的記憶。

參考資料：

* [SAMBA](https://zh.wikipedia.org/wiki/Samba)

* [SMB](https://en.wikipedia.org/wiki/Server_Message_Block)

* [CIFS](https://www.digitimes.com.tw/tw/dt/n/shwnws.asp?cnlid=&id=0000124179_f7z0bebu895gfp8dfsvfx)

* [**鳥哥的Linux 私房菜** - SAMBA](http://linux.vbird.org/linux_server/0370samba.php#theory)

* [CentOS 6.5 【SAMBA Server】 提供 Linux 與 Windows 溝通的介面，網路上的芳鄰](https://shazi.info/centos-6-5-%E3%80%90samba-server%E3%80%91-%E6%8F%90%E4%BE%9B-linux-%E8%88%87-windows-%E6%BA%9D%E9%80%9A%E7%9A%84%E4%BB%8B%E9%9D%A2%EF%BC%8C%E7%B6%B2%E8%B7%AF%E4%B8%8A%E7%9A%84%E8%8A%B3%E9%84%B0/)

* [使用 Samba 在 Windows 內掛載 Linux 資料夾](https://noob.tw/samba/)

* [架設 Samba 伺服器](http://www.weithenn.org/2009/07/samba.html)

* [CentOS 7 安裝設定 Samba](https://www.opencli.com/linux/centos-7-install-samba)

