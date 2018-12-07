---
title: Java Spring Framework 筆記 - 初探Spring Web 學習筆記(1)
layout: post
categories: Java
tags:
- Java
- 筆記
- Spring Framework
---
這邊要實際透過Spring Framework來實做一個可以增刪改查的Web 程式，基本功能會是人員的新增、更新、查詢與刪除，並且會將資料寫進 MSSQL中，與一般的Todo清單功能類似，只是改為人員帳號而已。

這邊是事前作業，順便記錄下我初探基本 Spring Web的筆記，要時機操作到增刪改查要到下一篇文章。

<!--more-->

## 資料庫建立

首先資料庫的部分，我是透過Docker安裝了MSSQL，安裝好後建立Database與一個Person Table：

### 1. 建立Server

```sql
CREATE DATABASE Spring
```

### 2. 建立Person Table

```SQL
CREATE TABLE Person (
    uuid UNIQUEIDENTIFIER not null,
    firstName VARCHAR(30),
    lastName VARCHAR(30),
    gender CHAR(1),
    email VARCHAR(320)
    PRIMARY KEY (uuid)
)

```

### 3. 新增資料

這邊新增兩筆測試用的資料。

```sql
INSERT INTO dbo.Person values(
    NEWID(),
    'Mary',
    'Huang',
    'F',
    'mary@example.com.tw'
);

INSERT INTO dbo.Person values(
    NEWID(),
    'Tom',
    'Lin',
    'M',
    'tom@example.com.tw'
);
```

## Spring Web 基礎

接著建立一個Spring Website，先透過maven加入 dependency：


```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.joseph.example</groupId>
  <artifactId>spring-website</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <dependencies>
  	<dependency>
  		<groupId>org.springframework</groupId>
  		<artifactId>spring-core</artifactId>
  		<version>5.0.8.RELEASE</version>
  	</dependency>
  	<dependency>
  		<groupId>org.springframework</groupId>
  		<artifactId>spring-beans</artifactId>
  		<version>5.0.8.RELEASE</version>
  	</dependency>
  	<dependency>
  		<groupId>org.springframework</groupId>
  		<artifactId>spring-context</artifactId>
  		<version>5.0.8.RELEASE</version>
  	</dependency>
  	<dependency>
  		<groupId>org.springframework</groupId>
  		<artifactId>spring-jdbc</artifactId>
  		<version>5.0.8.RELEASE</version>
  	</dependency>
  	<dependency>
  		<groupId>org.springframework</groupId>
  		<artifactId>spring-web</artifactId>
  		<version>5.0.8.RELEASE</version>
  	</dependency>
  	<dependency>
  		<groupId>org.springframework</groupId>
  		<artifactId>spring-webmvc</artifactId>
  		<version>5.0.8.RELEASE</version>
  	</dependency>
  	<dependency>
  		<groupId>jstl</groupId>
  		<artifactId>jstl</artifactId>
  		<version>1.2</version>
  	</dependency>
  </dependencies>
</project>
```
<img src="https://i.imgur.com/OlNxFcJ.png" alt="pom.xml設定" title="pom.xml設定">

另外在專案上點**右鍵 ->  properties -> Deployment Assembly **，須要將Maven 加到路徑上

<img src="https://i.imgur.com/2JpmHbe.png" alt="Deployment Assembly設定" title="Deployment Assembly設定">

### 1. 建立Dispatch Servlet
建立Dispatch Servlet的目的在於過濾所有的請求，所有送往 Server端的請求，都會由Spring將請求送往Dispatch Servlet，再由Dispatch Servlet尋要對應的controller做處理。

在專案上點右鍵新增 Servlet，接著將 **Use an existing Servlet class or JSP** 選項勾起，接著選 Browse，找到Spring 內的 **Dispatch Servlet**

<img src="https://i.imgur.com/tkYTk1s.png" alt="新增DispatchServlet" title="新增DispatchServlet">

<img src="https://i.imgur.com/KJa8Scw.png" alt="選擇DispatchServlet" title="選擇DispatchServlet">

如果在選擇Dispatch Servlet時出現沒有可用的Servlet訊息時，可以嘗試到**專案 -> Properties -> Targeted Runtimes**，將要佈屬的 Tomcat勾起來，如果沒有則自己建立一個，在新增完畢後，應該就可以選擇DisptachServlet類別：

<img src="https://i.imgur.com/rHv6EKK.png" alt="設定Targeted Runtimes" title="設定Targeted Runtimes">


完成後會在web.xml內看見所加入的 Dispatch Servlet設定：

<img src="https://i.imgur.com/R3wWYUc.png" alt="web.xml設定" title="web.xml設定">

接著我們調整Servlet設定：

web.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" id="WebApp_ID" version="3.1">
  <display-name>spring-website</display-name>
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
    <welcome-file>default.html</welcome-file>
    <welcome-file>default.htm</welcome-file>
    <welcome-file>default.jsp</welcome-file>
  </welcome-file-list>
  <servlet>
    <description></description>
    <display-name>persons</display-name>
    <servlet-name>persons</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>persons</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```
這邊改掉Servlet名稱，同時在servlet-mapping的地方，將url-pattern 改為 /，表示所有符合的請求都會進入 DispatchServlet中。

### 2. 設定 bean configuration file
web.xml中的 DispatchServlet須要讀取 Bean的設定檔，沒有設定的情況下，Spring在啟動時會自動找 WEB-INF目錄下**servletName-servlet.xml**，以這邊的例子來看就是 person-servlet.xml。
因此我們在 WEB-INF目錄下建立一個 spring-bean-configuration-file，名稱為 persons-servlet.xml。

若不希望以這種方式讀取bean設定檔，可以在web.xml Servlet設定的地方，加上init-param設定contextConfigLocation，指明對應的bean設定檔放在哪一個目錄下：

```xml
<servlet>
		<description></description>
		<display-name>persons</display-name>
		<servlet-name>persons</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<load-on-startup>1</load-on-startup>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/path/to/config</param-value>
		</init-param>
	</servlet>
```

接著在person-servlet.xml設定檔中，namespace中須還要額外啟用 context與 mvc 兩項：

<img src="https://i.imgur.com/CI1pEin.png" alt="bean設定" title="bean設定">

接著加入下列設定：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">

	<context:component-scan
		base-package="org.joseph.example.controller">
	</context:component-scan>
	<mvc:annotation-driven></mvc:annotation-driven>
</beans>
```

**context:component-scan**：目的在於掃描base-package特定的annotation，將其註冊為bean，以作為autowire物件管理。
**mvc:annotation-driven**：就網路上查到的部分，簡單來說是透過這個設定，Spring會為程式自動註冊 8 個類別，分別有不同用途，而這次程式須要用到其中兩項：分別為 RequestMappingHandlerMapping與 BeanNameUrlHandlerMappring，分別用途為：

1. RequestMappingHandlerMapping：用來處理@RequestMapping annotation
2. BeanNameUrlHandlerMappring：處理 request url，使其對應到 controller


### 3. 建立Controller
在程式目錄下建立一個package：org.joseph.example，並在package下建立一個PersonController.java檔：

PersonControl.java：
```java
package org.joseph.example.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class PersonController {

	@RequestMapping("/")
	public String showIndex() {
		return "index";
	}
}
```

Controller建立好後，最後就是建立 View。

### 4. 將Controller結果對應到 View

在Controller處理完後，我們會須要決定結果要呈現什麼，透過哪一個jsp 檔案來呈現，
而負責處理的稱為 ViewResolver，必須在bean configuration file中加入一個ViewResolver的bean。在Spring中有多個不同的 ViewResolver，這邊用到的是**InternalResourceViewResolver**。

其中我們會設定兩個property：prefix與suffix，設定如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">

	<bean id="jspViewResolver"
		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/jsps/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>

	<context:component-scan
		base-package="org.joseph.example.controller">
	</context:component-scan>
	<mvc:annotation-driven></mvc:annotation-driven>
</beans>

```
當Controller處理完後，會回傳一個對應的字串回來，如index字串，接著會由 InternalResourceViewResolver處理，會依據所設定的 prefix與suffix，對應到 **prefix + controller回傳字串 + suffix**，即最後的結果會對應到 **/WEB-INF/jsps/index.jsp**。

因此現在差最後一步，我們最後在 WEB-INF下新增一個目錄叫 jsps，並且加入任意一個JSP檔，檔名叫 index.jsp，到這邊基本的架構就完成了。

### 5. 測試
接著就可以開啟瀏覽器，輸入 localhost:8080/spring-website/，就會看見自己的JSP檔案呈現在畫面上。

整個流程會變成：

1. 程式收到請求
2. / 路徑會對應到 person servlet，由person servlet來對應到所負責的 controller
3. person controller收到請求，接著比對 RequestMapping路徑，找到RequestMapping("/")，故由showIndex method處理
4. 處理完後回傳 index
5. 接著由ViewResolver處理，這邊設定為 InternalResourceViewResolver，並設定了 prefix與suffix，所以最後View路徑會在 /WEB-INF/jsps/index.jsp
6. 程式會到 /WEB-INF/jsps/index.jsp，將JSP呈現在畫面上。

## 資料傳遞

接著是紀錄如何讓controller中處理好的資料呈現到網頁上，做到動態資料的效果。

### Session 資料處理

第一個方式是透過 session來做到。

RequestMapping還有其他參數可傳入，參考[Method Arguments](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-arguments) 可以知道，還可以傳入 session、request、response等資料，所以我們可以在controller中注入這些資料，這邊可以將資料放入session中，讓JSP從session將資料取出並顯示在畫面上：

```java
package org.joseph.example.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class PersonController {

	@RequestMapping("/")
	public String showIndex(javax.servlet.http.HttpSession session) {
		session.setAttribute("name", "Joseph");
		return "index";
	}
}
```
在session內放入 name為 Joseph字串，接著到 index.jsp中，使其顯示出來：

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	Hello
	<%=session.getAttribute("name")%>
</body>
</html>
```

### ModelAndView
另外還有透過ModelAndView來做到，ModelAndView中有一個Map物件，scope只有在 request階段有效，我們也可以將資料放到ModelAndView中的Map，並在JSP的地方將資料取出並印在畫面上。

前面Controller的方法是透過回傳字串的方式給ViewResolver處理，ModelAndView可以想為是同一個方式的加強版。

以程式碼來看：

```java
package org.joseph.example.controller;

import java.util.Map;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class PersonController {

	@RequestMapping("/")
	public ModelAndView showIndex(javax.servlet.http.HttpSession session) {
		session.setAttribute("name", "Joseph");
		
		ModelAndView mv = new ModelAndView("index");
		// Only persist in request scope
		Map<String, Object> map = mv.getModel();
		map.put("name", "Tiffany");
		return mv;
	}
}
```

原本showIndex方法會回傳String，現在改為回傳ModelAndView類別，在method內透過new方法，將原本要回傳的String作為參數傳入即可，效果與直接回傳String相同，差異在於ModelAndView有一個Map物件，**只存在與 request scope**。

因此在map內一樣放入 name屬性物件，接著在JSP的地方呈現：

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	Hello Session:
	<%=session.getAttribute("name")%><br /> Request:
	<%=request.getAttribute("name")%>
</body>
</html>
```

<img src="https://i.imgur.com/cWZnfOb.png" alt="執行結果" title="執行結果">

放在 ModelAndView的Model Map內的物件，可以在最後直接透過getAttribute存取。

這邊為了傳遞資料讓View可已呈現，所以建立了ModelAndView，不過參考官方文件，其實還可以讓程式碼更簡單，就是讓Spring在 method內注入 Model物件，先看程式：

這邊改寫剛剛的 showIndex method：

```java
	@RequestMapping("/")
	public String showIndex(javax.servlet.http.HttpSession session, Model model) {
		session.setAttribute("name", "Joseph");
		
		model.addAttribute("name", "Tiffany");
		return "index";
	}
```
第一個是回傳刑別改為String，第二個是這邊注入了一個 model物件，原本設在 Map內的key and value，現在改設在Model物件內，之後執行程式，可以得到相同的結果。

**雖然寫法不同，但是做到的效果都是一樣的。**

## 資料呈現


1. JSP中JAVA專用的 <%=variableName%>
2. Spring Expression Language
3. Spring Expression Language 搭配 JSTL (須要jstl.jar須要jstl.jar) [Official](https://docs.oracle.com/javaee/5/jstl/1.1/docs/tlddocs/)

這邊直接透過程式來展示：

PersonController：

```java
package org.joseph.example.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class PersonController {
	
	@RequestMapping("/")
	public String showIndex(Model model) {		
		model.addAttribute("hello", "<b>Hello World</b>");
		return "index";	
	}
}
```

index.jsp：

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	Request: <%=request.getAttribute("name")%><br>
	SPEL:${name}<br/>
	JSTL: <c:out value="${name}" /><br/>
</body>
</html>
```

**如果要在JSP中使用JSTL tag，則在最上方必須加入一行宣告。**
執行結果如下：

<img src="https://i.imgur.com/LQWvJmR.png" alt="執行結果" title="執行結果">

這邊在model中放入一個特殊字串，然後分別透過三種方式呈現出來，
SPEL可以直接取得 model 中的資料，所以直接透過 ${name}即可以取出資料；JSTL 的好處是預設就會過濾掉字串中的特殊字元，所以\<b\>與\</b\>字串就不會被視為html tag。

一般來說，若讓字串中特殊字元在網頁上直接被直接呈現，可能會有安全性問題，所以建議在呈現資料到畫面上時，要對字串做跳脫處理，除了JSTL 預設就會做到外，還可以考慮透過其他第三方Library 來協助。

## JNDI(Java Naming and Directory Interface)

詳細的說明可以參考[Tomcat JNDI](https://tomcat.apache.org/tomcat-8.5-doc/jndi-datasource-examples-howto.html)的說明

1. 開啟 contex.xml：在 tomcat_dir/conf目錄下，可以找到 context.xml，我們可以在這個檔案內定義JNDI的來源。

<img src="https://i.imgur.com/G35YqHn.png" alt="context.xml" title="context.xml">
2. 加入 MSSQL JDBC dependency
要連MSSQL資料庫，必須先將MSSQL用的JDBC Driver加入 pom設定檔中

```xml
  	<dependency>
  		<groupId>com.microsoft.sqlserver</groupId>
  		<artifactId>mssql-jdbc</artifactId>
  		<version>6.4.0.jre8</version>
  	</dependency>
```

3. 加入設定：
這邊可以參考官網上的語法，不過官網是用MySQL作為範例，我這邊是改為MSSQL，所以須要調整Driver類別與Url寫法，所使用的Driver類別必須另外下載 mssql-jdbc-6.x的jar檔才可以找到，這邊用到是**SQLServerDriver.class**：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<Context>

	<!-- Default set of monitored resources. If one of these changes, the -->
	<!-- web application will be reloaded. -->
	<WatchedResource>WEB-INF/web.xml</WatchedResource>
	<WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>

	<!-- Uncomment this to disable session persistence across Tomcat restarts -->
	<!-- <Manager pathname="" /> -->

	<Resource name="jdbc/mydb" auth="Container"
		type="javax.sql.DataSource" maxTotal="100" maxIdle="30"
		maxWaitMillis="10000" username="sa"
		password="qgTVwxQ9uLmv9KxnaFex9GMGXPLTBd@a2DcxmDM."
		driverClassName="com.microsoft.sqlserver.jdbc.SQLServerDriver"
		url="jdbc:sqlserver://127.0.0.1:1433;databaseName=SpringDB;" />
</Context>
```
4. 設定 web.xml
開啟web.xml，在檔案中加入JNDI的參考：

```xml
	<resource-ref>
      <description>DB Connection</description>
      <res-ref-name>jdbc/mydb</res-ref-name>
      <res-type>javax.sql.DataSource</res-type>
      <res-auth>Container</res-auth>
  </resource-ref>
```

5. 測試連線
可以參考官網的例子，然後調整語法：
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/sql" prefix="sql" %>

<sql:query var="rs" dataSource="jdbc/mydb">
select * from Person
</sql:query>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<c:forEach var="row" items="${rs.rows}">
    FirstName: ${row.firstName}<br/>
    LastName: ${row.lastName}<br/>
    Gender: ${row.gender}<br/>
    Email: ${row.email}<br/>
</c:forEach>
</body>
</html>
```

<img src="https://i.imgur.com/Gygxv6w.png" alt="連線測試" title="連線測試">

到這邊後，從接收請求到畫面呈現，與連至資料庫的部分都沒問題了，下一篇會加入增刪改查的功能。