---
title: Cross-Site Scripting (XSS ) 攻擊
layout: post
categories: Security
tags:
- security
- javascript
- owasp
- xss
- cross site scripting
unsplashTag: 'hacker'
---

<!--more-->

##  前言

Cross-Site Scripting(XSS) 是一種常見的攻擊方式，而且方式相當多種變化，只要網頁上有 input欄位工使用者輸入，並且會在後續將使用者輸入資料呈現於網站上，就有很高的機會產生XSS弱點。

XSS主要原因為**<u>未正確的對使用者輸入資料作驗證與過濾</u>**，而允許攻擊者將程式注入至網站中，使得使用者連至網站時載入被植入的惡意代碼，而讓瀏覽器自動執行，後果可能導致使用者資料暴露或是下載其他惡意代碼。

## XSS類別

XSS 攻擊可以分為三種類型：

### 1. Stored XSS attacks

因為未正常過濾使用者所輸入的資料，導致攻擊的script被儲存至server端造成後續的攻擊，稱為 Stored XSS attack，常見的像是表單資料、系統記錄或是留言板回覆等欄位，惡意代碼輸入留言板，接著存入資料庫後，後續看見同一個留言資料的人都會受害。Stored XSS通常也稱為 Persistent  或是 Type-I XSS。

```jsp
<%
	 Statement stmt = conn.createStatement();
	 ResultSet rs = stmt.executeQuery("sql command");
	 if (rs != null) {
	  rs.next(); 
	  String title = rs.getString("title");
    String message = rs.getString("message");
%>
<!--從資料庫中取出之資料可能包含惡意代碼，未處理情況下直接輸出至畫面少可獳導致惡意代碼被執行-->
Title: <%= title %>
Message: <%= message%>
```

從DVWA提供之程式來測試，提供一個流言版，並將輸入之資料寫回資料庫中儲存，於下一個使用者進到該畫面時，在將資料抓出來呈現於畫面上：

<img class="img-fluid" src="https://imgur.com/ofURFUW.png"/>

當輸入欄位中加入了javascript程式後送出，資料將會儲存至資料庫中

<img class="img-fluid" src="https://imgur.com/w4mNSKC.png"/>

當其他人檢視到同一串留言時(或剛剛我留下的內容)，因為文字中含有javascript，瀏覽器會自動運行代碼。

<img class="img-fluid" src="https://imgur.com/a6ioeYn.png"/>

### 2. Reflected XSS attacks

與Stored XSS不同， Reflexted XSS不需要儲存於Server端。主要發生原因是Server端未正確過濾前端傳入內容，在未適當過濾使用者輸入資料後，直接將使用者輸入之資料寫回response中導致的漏洞。這類攻擊通常稱為 Non-Persistent或是Type-II XSS。這類攻擊經常會搭配社交工程，攻擊者將有問題連結編碼後，寄送給大量受害者，等待受害者自己點擊連結。

```jsp
<% String name = request.getParameter("name"); %> 
<!--未將name做適當過濾，而直接呈現於Response中-->
Your name: <%= name %>
```



這個攻擊方式可以直接從DVWA 這個程式來做實際操作：

在input上輸入名字後點 submit，畫面上會出現使用者輸入之內容

<img class="img-fluid"  src="https://imgur.com/ODRL8m4.png"/>

當輸入內容中加入javascript時

<img class="img-fluid" src="https://imgur.com/C9NgUch.png"/>

<img class="img-fluid" src="https://imgur.com/AC1Du6k.png"/>

可以看DVWA上提供的後端程式，會知道後端僅將輸入作為結果輸出至畫面上，這情況很容易被利用，：

```php
<?php

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Feedback for end user
    echo '<pre>Hello ' . $_GET[ 'name' ] . '</pre>';
}

?> 
```

所以在Response的內文上，看見寫入的script 直接作為 response 回傳回來，導致使用者端直接運行代碼

<img class="img-fluid" src="https://imgur.com/Z6cIkaN.png" />



### 3. DOM based XSS

所以與前兩項主要的差異在於，Stored XSS與Reflected XSS是Server端未正常處理資料，將惡意代碼一同返回Response中而導致使用者瀏覽器運行惡意代碼。

DOM based XSS則是Server端傳回正常代碼，傳回正常的HTML與javascript，但是前端javascript 未適當處理接收到的使用者輸入資料，在動態產生DOM的過程中，因為輸入內容中被植入惡意代碼，導致程式與資料以未預期的方式執行

從一個簡單的例子來看，下列一段完整HTML，當使用者輸入完name後，點選按鈕，會將name作為參數傳至同一個網頁呈現：

```html
<html>
<head>
  <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
</head>
Your name:
<span id="name">
  <script>
    if (document.location.href.indexOf("name=") !== -1) {
      const name = decodeURIComponent(document.location.href.substring(document.location.href.indexOf("name=") + 5));
      document.getElementById("name").innerHTML = decodeURI(name);
      // document.write(name) 
    }
  </script>
</span>

<form>
  <input type="text" name="test" value="">
  <button onclick="testClick()" type="button">Click</button>
</form>
<script>
  function testClick() {
    location.href = "/index.html?name=" + document.querySelector("input[name=test]").value;
  }
</script>
</html>
```

這時候請求 http://localhost/index.html?name=joseph<3，這段程式從location.search中取得 name參數，並呈現至畫面上

但一旦這個規則被知道了，請求上的參數可能會改變，可能的參數如

```http
http://localhost/index.html?name=<img src=x onError="alert('Hello')" />
```

這時重新進入網網頁，會發現html上不會出現參數中的內容，反而跳出一個alert

<img class="img-fluid" src="https://imgur.com/Y8Rc9RG.png" />

另一種寫法，將原本 innerHtml改為 document.write後，可以直接執行 script 語法

```html
http://localhost:12345/index.html?name=<script>alert('xxx')</script>
```

<img class="img-fluid" src="https://imgur.com/xAzVyFE.png" />

Dom-based XSS 與Reflected XSS最大的差異在於，對網頁做請求後<u>**產生的Response內容**</u>，Reflected XSS是已經將惡意代碼作為Response傳回使用者端，而 DOM-based XSS則是請求的Response內容正常，但是透過Javascript動態產生內容時，誤將惡意代碼放入DOM中，導致瀏覽器執行惡意代碼。



## XSS 影響

XSS 可能會造成使用找session cookie被揭露、 劫持  user session、將使用者轉導至惡意網站安裝木馬程式等，甚至可能更改使用者畫面上所見的內容，如更改藥局網站上藥品的劑量，導致服用過量藥物。



## XSS 防護方式

XSS 最根本防護方式，為對不受信任的資料作過濾與編碼，避免讓資料呈現於網頁上時有機會可以運行惡意代碼，輸出時針對下列幾個字元，統一做跳脫處理：

| 原字元 | 跳脫字元 |
| ------ | -------- |
| <      | \&lt;    |
| >      | \&gt;    |
| "      | \&quot;  |
| '      | \&#x27;  |
| &      | \&amp;   |
| /      | \&/#x2F; |

各個程式語言大多都有提供針對字元做Encode的套件，OWASP也提供了 XSS的 [Prevention Cheat Sheet](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.md)可供參考。

1. 在將不信任資料輸出到HTML前，必須先做適當的跳脫，包含前面表格所列項目

```html
<div>escape untrusted data here</div>

<span>escape unstructed data here</span>
```

2. 將不信任資料作為html attribute前，必須做適當的跳脫

```html
<!--盡量避免使用unquoted attribute，OWASP上提及unquoted attribute容易被其他符號打亂導致弱點產生-->
<div attr=escape unstructed data here></div>

<div attr='escape unstructed data here'></div>

<div attr="escape unstructed data here"></div>
```

3. 將不信任資料用於javascript前，需要做適當的跳脫

```html
<script>alert('escape unstructed data here')</script>

<div onclick="x='escape unstructed data here'"></div>
```

 setInterval與 setTimeout比較特別，OWASP中提到，即便是跳脫過的資料，放入setInterval與setTimout中都一樣危險，所以要盡量避免
```html
<!--避免在setInterval, setTimeout中使用不信任的資料，即便跳脫後仍然一樣-->
window.setInterval('escape or not escape unstructed data ')
```

同時也建議前後端分離的架構中，如果是透過ajax請求回來的資料， response中的 Content-Type 如果是json，則應該要呈現 application/json，而非 application/html，如此一來可以避免瀏覽器解讀錯誤。

4. 不信任資料作為CSS屬性使用前，需要嚴格驗證與跳脫

```css
<style>
	selector{ property: escape unstructed data here}
</style>

<a style="{property: escape unstructed data here}" ></a>
```

5. 將不信任資料作為網頁連結或網頁參數時，必須適當的跳脫，URL部分僅需要對網址後參數跳脫即可，如果整個網址都跳脫，瀏覽器會沒辦法正確的找到目標網址

```html
<a href="https://example.com?param1=escape unstructed data here" >Link</a>
```

其他建議的部分

1. 使用 httpOnly cookie，可以避免 cookie被 javascript 存取
2. 實做 Content Security Policy，透過設定 Content Security Policy，可以讓瀏覽器知道哪些javascript、css或圖片資源可以被載入與執行，瀏覽器支援部分可以參考[這邊](https://content-security-policy.com/)，主要是在Response header上加入CSP資訊即可，這部分可以透過程式加入，或是可以透過server一律在 response中加入

```nginx
# nginx setting 表示瀏覽器只允許從網頁 origin端與 example.com 端讀取資源
add_header Content-Security-Policy "default-src 'self' 'example.com';";
```

3. 使用自動跳脫字元的template程式，如angularjs、Go Templates
4. 加入 [X-XSS-Protection](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection) Header於Response中，加入後允許瀏覽器開啟內建的 XSS filter，但每個瀏覽器的設定都不太一樣，使用時需要參閱說明文檔
5. 前端適當的使用JS框架，如angularjs、Angular 2+或ReactJS等



## 總結

XSS 攻擊方式相當多元，也許這就是XSS一直留在 OWASP Top 10的原因，其中還包含自己尚未弄清原理的攻擊方式，不過稍微做了一些功課後才終於知道，為什麼之前專案在掃白箱時，自己寫的防XSS語法為什麼會被叛定無效，稍微知道攻擊的思路後，未來在開發上才會更加小心。

參考資訊

[OWASP - XSS](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS))

[XSS Filter Evasion Cheat Sheet](https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet)

[OWASP - Cross_Site_Scripting_Prevention_Cheat_Sheet](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.md)

[XSS攻擊的深入探討與防護之道](https://www.qa-knowhow.com/?p=2992)

[Web安全系列（四）：XSS 的防御](https://520mwx.com/view/7719)

[DVWA](http://www.dvwa.co.uk/)

[mutillidae](https://www.owasp.org/index.php/OWASP_Mutillidae_2_Project)