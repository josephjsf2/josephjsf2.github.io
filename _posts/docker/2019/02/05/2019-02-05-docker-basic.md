---
title: Docker學習筆記 - Docker基礎指令
layout: post
categories: docker
tags:
- docker
- images
- container
---

Docker 安裝完之後，可以透過 terminal 操作相關指令，這邊簡單筆記幾項簡單的操作：

<!--more-->

# Image：

## 尋找images

### Docker Hub 

可以在[Docker Hub](https://hub.docker.com)上找尋要使用的Images，可以透過詳細資訊知道如何透過 docker pull 指令。

### Docker Search

或直接透過docker內建指令查詢：

```bash
docker search imageName
```

Image 將會列出所有尋找之結果，並會標注是否為官方發佈版本

<img  class="img-fluid" src="https://i.imgur.com/rgo82ba.png" width="60%"alt="搜尋結果" title="搜尋結果"/>



<img  class="img-fluid" src="https://i.imgur.com/o8veP06.png" width="60%"/alt="搜尋結果" title="搜尋結果">

## 下載 images

```
docker pull imageName[:tag]
```

將所指定的image抓回本機，可以指定版本號，若未指定，預設則會抓 latest版本回來。

如果需指定版本，則在imageName後方加入 :Tag，如 

```
docker pull ubuntu:14.3.1
```

## 檢視已下載 images

```
docker images
```

## 刪除images

```
docker rmi imageName[:tag]
```

在刪除image之前，必須先確定沒有container用到要刪除的image，才可以正常刪除。

# Container

## Container指令

```bash
# 建立 Container
docker create imageName
# 啟動 Container
docker start imageName
# 停止 Container
docker stop imageName
# 刪除Container
docker rm imageName
```

## 實際建立Container (建立 Nginx)

這邊透過 docker run 來執行，docker run 等同於執行 docker create 與 docker start，執行時會先檢查本機是否有指定的 image，若沒有，將會自動從 repo下載image到本機，接著建立 container，執行應用程式，最後終止。

``` bash
docker run --name MyNginx -dt nginx
```

* --name  container名稱
* -d 讓container 背景執行
* -t 啟動後建立一個虛擬的終端機，讓container上process執行，避免container process執行完畢而終止。
* 因為docker 上每一個container只能由一個 process執行，且process在執行完之後，container 就會自動釋放，所以在執行完start 之後，因為沒有其他執行的process，所以container就會自動釋放，故在指令上加上 -t 參數，讓docker container預設執行bash 不讓container被釋放

建立完成後可以查看是否成功新建：

```
docker ps
```
<img  class="img-fluid" src="https://i.imgur.com/BCx3W8h.png" width="60%" alt="執行結果" title="執行結果"/>


## 進入 Container

```
docker exec -it MyNginx bash
```

* -i  可以想為如果希望以透過terminal方式，輸入指令得出結果這種互動方式執行，則必須加上這參數
* -t 讓docker以虛擬 terminal 方式執行

上面文字可能很難懂，如果仍不知道實際上的用途，這邊建議是直接透過指令來觀察差異：

```bash
# 同Terminal方式執行
docker exec -it MyNginx bash

# 單純透過指令與回應方式執行，一樣可執行，但畫面相當不容易看
docker exec -i MyNginx bash

# 輸入指令後沒回應之 Terminal
docker exec -t MyNginx bash
```

如果要退出 container，則執行 **exit** 即可。



這邊僅記錄一小部分指令，docker 提供了相當多的指令，同時還有 optional的參數可供輸入，在使用時可以多搭配 --help 來查看docker提供的功能。，