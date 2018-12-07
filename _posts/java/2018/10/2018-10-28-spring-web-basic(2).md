---
title: Java Spring Framework 筆記 - 初探Spring Web 學習筆記(2)
layout: post
categories: Java
tags:
- Java
- 筆記
- Spring Framework
---
這篇將延續前一篇文章，建立起DAO層與Service層程式，並將資料呈現到畫面上。

<!--more-->

## 建立 Person Model
在前一篇在資料庫建立了 Person的Table， Person Model是為了可以對應到 Person Table的各個欄位，等於是 DB與 Java 物件的對應：
Person.java：

```java
package org.joseph.example.model;

public class Person {

	private String uuid;
	private String firstName;
	private String lastName;
	private String gender;
	private String email;

	public String getUuid() {
		return uuid;
	}

	public void setUuid(String uuid) {
		this.uuid = uuid;
	}

	public String getFirstName() {
		return firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

	public String getLastName() {
		return lastName;
	}

	public void setLastName(String lastName) {
		this.lastName = lastName;
	}

	public String getGender() {
		return gender;
	}

	public void setGender(String gender) {
		this.gender = gender;
	}

	public String getEmail() {
		return email;
	}

	public void setEmail(String email) {
		this.email = email;
	}

}
```

## 建立 Person DAO

首先先新增一個 PersonDAO.java：

```java
package org.joseph.example.dao;

@Component("personDAO")
public class PersonDAO {
	
	private NamedParameterJdbcTemplate jdbc;
	
    @Autowired
	public void setDataSource(DataSource jdbc) {
		this.jdbc = new NamedParameterJdbcTemplate(jdbc);
	}
	
	public List<Person> listPersons(){
		return this.jdbc.query("SELECT * FROM PERSON", new RowMapper<Person>() {
			public Person mapRow(ResultSet rs, int rowNum) throws SQLException {
				// TODO Auto-generated method stub
				Person person = new Person();
				person.setUuid(rs.getString("uuid"));
				person.setFirstName(rs.getString("lastName"));
				person.setFirstName(rs.getString("firstName"));
				person.setGender(rs.getString("gender"));
				person.setEmail(rs.getString("email"));
				return person;
			}
		});
	}
}
```

在這邊於類別名稱上加上 Component，目的是為了讓Spring 可以自動掃描到這個檔案，並允許作為注入來源。同時，加上了一個 listPersons的方法，用來將資料表內所有資料都列出來，透過NamedParameterJdbcTemplate 提供的對資料庫存取方式來取得資料表內資料，並透過**RowMapper將取出來的原始資料轉為 Person java物件**。

建立好 PersonDAO後，接著要建立對應的 bean configuration file：

dao-context.xml：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">


	<context:component-scan base-package="org.joseph.example.dao"></context:component-scan>
</beans>
```

建立一個基本的 bean configuration file，並啟用 context namespace，同時加上 component-scan的 element，**指定basic-package為DAO的 package：org.joseph.example.dao**。

同時為了要可以透過Autowired方式注入JNDI的 DataSource，須要額外啟用 JEE namespace：

<img src="https://i.imgur.com/OqQ0pcp.png" alt="啟用jee" title="啟用jee">



接著在切換到JEE的頁簽，加上**jee:jndi-lookup**：

<img src="https://i.imgur.com/tE51No2.png" alt="jee設定" title="jee設定">

<img src="https://i.imgur.com/Lkl7CdG.png" alt="jee設定" title="jee設定">

jndi-name設定可以參考web.xml中對應到 Tomcat context.xml JNDI的設定，目的是讓Spring可以將JNDI 資源作為可注入的來源。

接著要在專案中加入ContextLoaderListener，目的在讓Spring可以讀取自己定義的Container，
**預設會讀取application-context.xml**，而這邊要讀取的是dao-context.xml，因此要將這個設定加入到ContextLoaderListener的設定中，須要透過加上contextConfigLocation設定，告訴Spring要讀取的檔案路徑，讓Spring在啟動後讀取這個Container設定。

在spring-web.jar中可以找到**org.springframework.web.context.ContextLoaderListener.class**，並且加上**contextConfigLocation**設定。

web.xml：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://xmlns.jcp.org/xml/ns/javaee"
	xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
	id="WebApp_ID" version="3.1">
	<display-name>spring-website</display-name>
	<welcome-file-list>
		<welcome-file>index.html</welcome-file>
		<welcome-file>index.htm</welcome-file>
		<welcome-file>index.jsp</welcome-file>
		<welcome-file>default.html</welcome-file>
		<welcome-file>default.htm</welcome-file>
		<welcome-file>default.jsp</welcome-file>
	</welcome-file-list>
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:org/joseph/example/config/dao-context.xml</param-value>
	</context-param>

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
	<resource-ref>
		<description>DB Connection</description>
		<res-ref-name>jdbc/mydb</res-ref-name>
		<res-type>javax.sql.DataSource</res-type>
		<res-auth>Container</res-auth>
	</resource-ref>

</web-app>
```

如果要測試是否有在啟動時讀取 dao-context.xml設定，這邊可以在PersonDAO上加入一個constructor來測試：

personDAO：
```java
	PersonDAO(){
		System.out.println("PersonDAO loaded.");
	}
```

<img src="https://imgur.com/T2aG0bM.png" alt="啟動設定" title="啟動設定">




## 建立 Person Service

定義一個Service讓 Controller呼叫，呼叫的流程為 Controller -> Service -> Dao -> DataSource。

建立Service的步驟與建立DAO的部分相當類似，首先先建立一個PersonService.java：

```java
@Service("personService")
public class PersonService {

	@Autowired
	private PersonDAO personDao;
	
	public List<Person> getPersonsList(){
		return this.personDao.listPersons();
	}
}
```

因為PersonService須要使用 PersonDAO，所以可以透過Autowired的方式注入。

並定義一個getPersonsList方法，透過這個方法調用personDao，並回傳資料

接著建立service-context.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">


	<context:component-scan base-package="org.joseph.example.service"></context:component-scan>
</beans>

```

同時指定component-scan目錄為service之目錄。

建立好component-scan.xml後，同樣要在web.xml內的 contextConfigLocation設定內加上路徑，如果contextConfigLocation內有多個路徑，以逗號最為區隔：

```xml
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>
		classpath:org/joseph/example/config/dao-context.xml,
		classpath:org/joseph/example/config/service-context.xml</param-value>
	</context-param>

```



## 將資料呈現至網頁

這邊調整原本的PersonController，將原本測試的程式移除，改為呼叫PersonService來取得資料，而 PersonSerice透過 Autowired方式注入：

PersonController：

```java
package org.joseph.example.controller;

import java.util.List;

import org.joseph.example.model.Person;
import org.joseph.example.service.PersonService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class PersonController {

	@Autowired
	private PersonService personService;

	@RequestMapping("/")
	public String showIndex(Model model) {
		List<Person> personsList = this.personService.getPersonsList();
		model.addAttribute("persons", personsList);
		return "index";
	}
}
```

這邊將透過Service取出之資料方入 Model的 Map中，接著會回到 index.jsp，讓資料呈現在畫面上。

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
<c:forEach var="row" items="${personsList}">
    FirstName: ${row.firstName}<br/>
    LastName: ${row.lastName}<br/>
    Gender: ${row.gender}<br/>
    Email: ${row.email}<br/>
</c:forEach>
</body>
</html>
```



執行結果：

<img src="https://i.imgur.com/uUnVDPc.png" alt="執行結果" title="執行結果">

到這邊已經基本串起整個資料流程了，由請求進入personController後，接著透過personService呼叫personDao將資料取出來，最後在personController內將所取出之資料寫入Model的Map中，在JSP由SPEL將資料取出並呈現在畫面上。



在下一篇將會完成整個資料的新增、刪除與修改功能。