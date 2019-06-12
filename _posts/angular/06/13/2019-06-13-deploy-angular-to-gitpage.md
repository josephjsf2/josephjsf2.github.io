---

title: Deploy Angular 程式到 Github page
layout: post
categories: angular
tags:

- angular
- github page
- ngh

---

我到今天才知道原來Angular專案也可以放上 github page！

<!--more-->

### 1. 前置作業
準備一組 Github帳號

### 2. 在github上建立 repo
在 github上建立一個 repository，這邊取名叫 github-page-demo

<img src="https://imgur.com/lzpMwRT.png" width="600px" height="400px"/>

  

### 3. 建立Angular專案
在本機上建立 Angular專案，或是用已經存在的專案也可

```bash
ng new github-page-demo
```

<img src="https://imgur.com/lzpMwRT.png" width="600px" height="400px"/>

### 4. 程式加入 remote設定

```bash
git remote add origin YOUR_GITHUB_URL
```

Github Url 可以在剛剛建立的 repo上找到。

<img src="https://imgur.com/rzO5wLZ.png"/>



### 5. 啟動Angular程式

```bash
ng serve
```

接著打開瀏覽器，應可以看見網頁正常運作。

<img src="https://imgur.com/9lGiqTX.png" width="700px" height="300px"/>

  

### 6. 安裝 angular-cli-ghpages 套件

```bash
npm i -g angular-cli-ghpages
```

angular-cli-ghpages套件是可以讓 Angular程式佈到 github的主要套件。

  

### 7. 打包程式

```bash
ng build --prod --base-href=/github-page-demo/
```

因為後面放上 github page後，路徑不是從 / 開始，故要在打包時加入 —base-href 參數，會在打包時修改 index.html上 base href設定。

或是另一種設定方式也可：

```bash
ng build --prod --base-href=https://YOUR_GITHUB_ACCOUNT.github.io/github-page-demo/
```



程式目錄下 index.html base href 設定：

<img src="https://imgur.com/yrA8c6T.png" />

打包後 index.html base href 設定，因為指定了 --base-href，所以改變了設定：

<img src="https://imgur.com/nl5k2Mf.png"/>

### 8. 發佈至 github

```bash
[sudo] npx ngh --dir=dist/github-page-demo --no-silent
```

* --dir目的是指定要部屬的檔案來源，因為程式打包後預設會產生在 dist目錄，故指定在 dist/PROJECT_NAME目錄下。
* --no-silent 目的是除錯用，可以不加也不影響運作，加入後可以看見 cli 將程式送上 github的紀錄

如果遇到 permission問題，可以在指令前方加入 sudo指令。

<img src="https://imgur.com/TkLkY9Z.png"/>

### 9. 瀏覽網頁

完成步驟8後即完成佈署，大概需要等1-3分鐘的時間才會生效，接著就可以連至 https://YOUR_GIT_ACCOUNT.github.io/YOUR_REPO_NAME就可以看到網頁。

如我這邊的url就是 http://josephjsf2.github.io/github-page-demo

<img src="https://imgur.com/yqREqNX.png"/>



### 結論

以後 Angular 的專案如果要 demo，又多了一個新的選項了！而且用起來也不複雜。

參考文章：

[Deploying an Angular App to Github Pages](https://alligator.io/angular/deploying-angular-app-github-pages/)

[免費在 GitHub Pages 執行你的 Angular 應用程式](https://poychang.github.io/publish-angular-app-to-github-pages/)

