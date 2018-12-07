---
title: Java Spring Framework 筆記 - Bean configutation file 設定
layout: post
categories: Java
tags:
- Java
- 筆記
- Spring Framework
---

當Java工程師也已經好一段時間，這段時間接了許多用了Spring的案子，寫了許多程式，雖然有能力可以改動程式，網路上一查都可以查到相關文章，程式照著複製貼上也可以運作，但寫這種程式的感覺相當不實際，所以還是決定好好的將Spring的知識補起來。

<!--more-->

首先是前置工程，過程中會用到的程式，設定 pom.xml，由maven來協助管理library的dependency

<img src="https://i.imgur.com/cymLUci.png" alt="drawing" width="60%"/>

建立Person.java：

```java
package spring.sample;

public class Person {

	private String name;
	private String gender;
	private Hair hair;
	private Map hairMap;
	private List hairList;

	public List getHairList() {
		return hairList;
	}

	public void setHairList(List hairList) {
		this.hairList = hairList;
	}

	public Person() {
	}

	public Person(String name, String gender, Hair hair) {
		this.name = name;
		this.gender = gender;
		this.hair = hair;
	}

	public static Person getInstance(String name, String gender, Hair hair) {
		return new Person(name, gender, hair);
	}

	public Map getHairMap() {
		return hairMap;
	}

	public void setHairMap(Map hairMap) {
		this.hairMap = hairMap;
	}

	public void init() {
		System.out.println("Person init");
	}

	public void destroy() {
		System.out.println("Person destroy");
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getGender() {
		return gender;
	}

	public void setGender(String gender) {
		this.gender = gender;
	}

	public Hair getHair() {
		return hair;
	}

	public void setHair(Hair hair) {
		this.hair = hair;
	}

	@Override
	public String toString() {
		return "Person [name=" + name + ", gender=" + gender + ", hair=" + hair + ", hairMap=" + hairMap + ", hairList="
				+ hairList + "]";
	}
}
```

Person中的Hair 屬性為interface 

Hair.java

```java
package spring.sample;
 
public interface Hair {
 
	String getHairColor();
}

```

定義兩種繼承Hair的物件，分別為兩種不同顏色 

RedHair.java

```java
package spring.sample;

public class RedHair implements Hair {

	private String color;

	public RedHair(String color) {
		this.color = color;
	}

	public String getHairColor() {
		// TODO Auto-generated method stub
		return this.color;
	}
}
```

BlueHair.java

```java
package spring.sample;

public class BlueHair implements Hair {
	private String color;

	public BlueHair(String color) {
		this.color = color;
	}

	public String getHairColor() {
		// TODO Auto-generated method stub
		return this.color;
	}

}
```

最後是要執行的主程式檔： 

App.java

```java
package spring.sample;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {

	public static void main(String[] args) {
		ApplicationContext context = new ClassPathXmlApplicationContext("spring/sample/beans/beans.xml");
		Person person = context.getBean(Person.class);
		System.out.println(person);
		((ClassPathXmlApplicationContext) context).close();

	}
}
```

ApplicationContext可以視為是 Spring的context，ClassPathXmlApplicationContext傳入定義beans.xml的class path。程式可以透過這個context取得所定義的bean內容。



最後建立spring-bean-configuration-file：beans.xml，如果eclipse內找不到，可能需要額外安裝spring plugin 就可以看見了，過程會藉由 beans.xml來設定Person 中的物件注入，好處是任何定義好的bean object的dependency可以透過 beans.xml設定，變更設定時不用修改程式並重新compile。

最後專案結構：

<img src="https://i.imgur.com/V9yl9Ng.png" width="60%"/>



接著就要進入Spring的世界了



## 1. constructor 參數注入

可以透過 spring-bean-configuration-file，也就是beans.xml來設定，在之中定義了Person、RedHair與BlueHair三個bean，上面的定義皆可以透過 eclipse用 GUI的介面產生：

beans.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean class="spring.sample.Person">
		<constructor-arg name="name" value="Joseph"></constructor-arg>
		<constructor-arg name="gender" value="Male">
		</constructor-arg>
		<constructor-arg name="hair" ref="blueHair"></constructor-arg>
	</bean>
	<bean id="redHair" class="spring.sample.RedHair">
		<constructor-arg name="color" value="RED"
			type="java.lang.String">
		</constructor-arg>
	</bean>
	<bean id="blueHair" class="spring.sample.BlueHair">
		<constructor-arg name="color" type="java.lang.String"
			value="BLUE">
		</constructor-arg>
	</bean>
</beans>
```

要使用constructor來注入，我們可使用<constructor-arg> tag來做到，要在bean內使用<constructor-arg>，**對應的bean內必須有對應的constructor才可以使用**，否則會拋出錯誤。 簡單記錄constructor-arg中可用屬性：

1. name：constructor中要注入的變數名稱
2. value：要注入的內容值
3. ref：要注入的內容**參考到其他的bean id**
4. type：注入內容的型態
5. index：要注入的變數的index，依照constructor定義的順序來注入

雖然有多個屬性，但只要設定其中幾種就可以work了。

以RedHair物件來看，constructor-arg內定義了 name即為 RedHair constructor參數名稱，並指定了注入的 value為RED，對應的即為RedHair物件內的constructor，在runtime時，RedHair內的color就會因為constructor注入，被設為 RED。

而Person內的hair 屬性可以設定要參考到 Blue還是Red，在runtime就會有不同的結果。

執行後結果：

<img src="https://i.imgur.com/AgQ7MTn.png" width="60%"/>

可以看到Person內的name與gender分別為Joseph與Male，而Hair則是注入了RedHair物件，所以透過getColor會拿到 RED；如果在beans.xml內，將原本hair的ref改為blueHair，則結果就會變成BLUE。

## 2. 透過 bean property注入

透過 property注入，其實就是使用物件的 set method來做到注入。

舉例來說，如果要注入 Person物件的 name屬性，那麼在 bean 內就必須要有 setName method，這樣才可以做到 bean的 property 注入。



這邊調整了 Person的注入方法，原本透過constructor注入，改為property注入：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean class="spring.sample.Person">
		<property name="name" value="Joseph"></property>
		<property name="gender" value="Male"></property>
		<property name="hair" ref="redHair"></property>
	</bean>
	<bean id="redHair" class="spring.sample.RedHair">
		<constructor-arg name="color" value="RED"
			type="java.lang.String">
		</constructor-arg>
	</bean>
	<bean id="blueHair" class="spring.sample.BlueHair">
		<constructor-arg name="color" type="java.lang.String"
			value="BLUE">
		</constructor-arg>
	</bean>
</beans>
```

<img src="https://i.imgur.com/oT1JmjQ.png" width="60%"/>

而 Person物件內需要稍微做一些調整，程式原本是透過 constructor注入，所以定義了一個 constructor並傳入三個變數，這樣明確定義constructor的方式，表示 **Person物件只有這樣一個constructor**( 若物件未定義任何constructor，則預設會有一個沒有參數的constructor)，而Spring在建立物件時，實際上也是透過constructor來建立，這時候執行程式會拋出BeanCreationException Error，並指明<font color="red">**No default constructor**</font>的錯誤。

所以這邊需要小調整程式，有兩個方式，一個是加入一個沒有帶參數的constructor，另一個是移除原本有三個參數的constructor。

Person.java：

```java
public class Person {

	private String name;
	private String gender;
	private Hair hair;

	public Person() {
	}

	public Person(String name, String gender, Hair hair) {
		this.name = name;
		this.gender = gender;
		this.hair = hair;
	}
	// getter and setter for all attribute
}
```

調整後執行程式，可以得到與之前相同之結果。



## 3. Bean Scope

透過Spring注入的Bean**預設都是 singleton**，表示在runtime期間**只會有一個實體產生**，舉例來說：

App.java：

```java
public class App {

	public static void main(String[] args) {
		ApplicationContext context = new ClassPathXmlApplicationContext("spring/sample/beans/beans.xml");
		Person person1 = context.getBean(Person.class);
		Person person2 = context.getBean(Person.class);
		System.out.println(person1 == person2);
		((ClassPathXmlApplicationContext) context).close();
	}
}
```

得到的結果會是 true，
這個設定可以在beans.xml中調整，總共有四種模式，session與request還沒學到先略過，至於另外兩個：

singleton：runtime只會建立一個實體物件，bean的建立與摧毀都由Spring管理
prototype：每一次參考到bean，實際上都是 new 一個新物件出來，物件的摧毀要由**自己管控**

<img src="https://i.imgur.com/kJaj9Gn.png" width="60%"/>

## 4. **Bean 的 init method與destroy method**

在每個bean都可以設定在初始化時要執行bean中的某個method；在被destroy時要執行另一個method，只要在beans.xml中設定即可。

我們可以在 root beans的地方，設定 default-init-method方法，所有定義的 bean在建立時都會執行 default-init-method指定的方法，比如 init method，如果bean內沒有init方法，就會略過。

init方法會在bean建立時執行，如果 bean建立時需要參考到其他的bean，**Spring會優先建立其他被參考到的bean，**最後才建立當前的bean。
destroy在執行applicationContext.close()時會調用，表示摧毀物件。



<img src="https://i.imgur.com/eiTPY3w.png" width="60%"/>



舉例來說，設定了default init與destroy method，如果Person中沒有init或 destroy method，也不會影響運行，會被Spring略過。

Spring也可以針對個別的bean設定 init與 destroy方法，**如果default與init同時存在，則只會執行 個別設定的method，不會執行default method**，舉例來說



<img src="https://i.imgur.com/NYUCyBM.png" width="60%"/>

Perosn.java：

```java
public class Person {

	public void init() {
		System.out.println("Person init");
	}

	public void destroy() {
  System.out.println("Person destroy");
 }

// skip
```

RedHair.java：

```java
public class RedHair implements Hair {

	public void init() {
		System.out.println("RedHair init");
	}

	public void destroy() {
  System.out.println("Red hair destroy");
 }
//skip
```

BlueHair.java：

```java
public class BlueHair implements Hair {

	public void init() {
		System.out.println("Blue hair init");
	}

	public void destroy() {
  System.out.println("Blue hair destroy");
 }

// skip
```

執行結果：

<img src="https://i.imgur.com/CyxcE3y.png" width="60%"/>

可以觀察到，RedHair一定會優先於Person被建立；在執行到close時，bean會分別調用destroy方法。

這邊調整beans.xml，在BlueHair物件中另外設定init與destroy method：

<img src="https://i.imgur.com/wIHqRd7.png" width="60%"/>

BlueHair.java：

```java
public class BlueHair implements Hair {

	public void burnHair() {
		System.out.println("Blue hair burn");
	}

	public void create() {
		System.out.println("Blue hair created");
	}

	public void init() {
		System.out.println("Blue hair init");
	}

	public void destroy() {
  System.out.println("Blue hair destroy");
 }
//...skip
```
最後運行程式：

<img src="https://i.imgur.com/ftujlkx.png" width="60%"/>

BlueHair中的 init與destroy方法不會被調用，相對的只會調用 create與 burnHair method。

## 5. FactoryBean & FactoryMethod

有兩種方式可以做到，第一種是透過類別內定義的方法，另一種是定義一個專門的Factory來做到，在專案逐漸變大後，可以考慮改以Factory-pattern方法來實做。

**factory-method**：
這邊調整Person的建立方式，由原本透過constructor建立改為透過Factory Method建立，因此在Person類別內加入一個靜態方法： getInstance()，並允許傳入參數作為初始化

Person.java：

```java
public class Person {

	private String name;
	private String gender;
	private Hair hair;

	public Person() {
	}

	public Person(String name, String gender, Hair hair) {
		this.name = name;
		this.gender = gender;
		this.hair = hair;
	}

	public static Person getInstance(String name, String gender, Hair hair) {
  return new Person(name, gender, hair);
 }

...skip
```

接著在bean.xml內的Person Bean內找到factory-method，加上getInstance方法，如果getInstance需要傳入參數，則以 constructor傳入參數的方式傳入即可：

<img src="https://i.imgur.com/oT23TVa.png" width="60%"/>



<img src="https://i.imgur.com/YDnYPsc.png" width="60%"/>

跑起來後與透過constructor建立的Bean有一樣的效果

**factory-bean & factory method**

透過定義一個專門的Factory 類別，並指定Factory類別中的方法來執行

首先定義一個Factory類別，而其中factory-method不可以是static，否則spring會出錯：

```java
package spring.sample;

public class PersonFactory {

	public Person getInstance(String name, String gender, Hair hair) {
		return new Person(name, gender, hair);
	}

}
```

接著在beans.xml內加入一個bean定義，

<img src="https://i.imgur.com/d4ZAFaZ.png" width="60%"/>

完成後在Person的Bean設定上加入 factory-bean，參考到剛剛建立的personFactory，並指明factory-method為PersonFactory內的getInstance方法，傳入的方式與先前相同

<img src="https://i.imgur.com/v98o9pz.png" width="60%"/>

最後執行的結果會與之前相同。

## 6. p namespace

Spring的 bean定義檔中還有提供另一個注入方法，叫 p-namespace，預設這個功能是沒打開的，可以在下列的地方，將 p勾選起來， eclipse就會自動在xml內加入 namespace。

<img src="https://i.imgur.com/llMNAIj.png" width="60%"/>

p namespace可以讓我們在 tag裡面設定注入的內容，如果是參考到其他bean，則要在屬性後面加上 -ref，下面改寫Person的 property注入方法：

<img src="https://i.imgur.com/ae6mC8F.png" width="60%"/>

其中name與gender都是直接輸入，hair的部分參考到其他bean，所以會用hair-ref來指向另一個bean，執行後如下，等同事先前的property 注入結果，效果是一樣的。

<img src="https://i.imgur.com/ndy36fm.png" width="60%"/>

## 7. Map & List注入方法

前面設定的property injection都是針對比較單純的屬性類別作注入，其他還有Map/List與InnerBean注入，也可以透過Bean設定檔來設定注入。

### 7.1 Map

Spring bean configuration file提供了gui介面可供設定，以Person的 hairMap來看，接受 key為String，value為Hair類別，同樣可以透過constructor或是property方式注入，如下圖所示：

<img src="https://i.imgur.com/dNofWmI.png" width="60%"/>

在點選constructor或是 property後按下右鍵，eclipse就會有輔助視窗，提示可供使用的類型，在這邊加入 map

<img src="https://i.imgur.com/5k9PGW6.png" width="60%"/>

接著點選map後，右邊可以設定key與value類別，按下右鍵可以再繼續新增entry，也就是設定key與value

<img src="https://i.imgur.com/40m1SG2.png" width="60%"/>

設定介面上，有ref的通常都是指向另一個bean設定，其餘的按照字面上意思設定即可，最後xml大致如下：

```xml
<property name="hairMap">
	<map key-type="java.lang.String" value-type="spring.sample.Hair">
		<entry key="blue" value-ref="redHair"></entry>
		<entry key="red" value-ref="redHair"></entry>
	</map>
</property> 
```

如果是設定比較單純的Map，如String/String，還可以考慮另一種作法，透過props來做到：

<img src="https://i.imgur.com/Inj2JmO.png" width="60%"/>

```xml
<property name="hairMap">
	<props>
		<prop key="blue">blueStr</prop>
		<prop key="red">redStr</prop>
	</props>
</property>
```

### 7.2 List

方法與Map大同小異，可以加入多種類別：

<img src="https://i.imgur.com/zkbyifS.png" width="60%"/>

在這邊是為了加入Hair List，所以參考到另外兩個bean，要把另外兩個bean加入List中，可以選擇加入ref(這邊可以insert不同 element，都會視為 list內的物件)，指向red與blue兩個bean

```xml
<property name="hairList">
	<list value-type="spring.sample.Hair">
		<ref bean="redHair" />
		<ref bean="blueHair" />
	</list>
</property>
```

要直接在list內建立新的bean也是可以的：

<img src="https://i.imgur.com/Ff27Dtq.png" width="60%"/>

還可以直接在內部使用property或是constructor注入，結果都視為List內的一個物件。

這邊僅提 Map與List兩種，其餘的設定方式都差不多，仰賴eclipse上的UI操作對一般來說已經相當足夠。

Spring 的Bean管理大致上到這邊，雖然現在許多都是透過annotation來做到，這部分會慢些補上來，接下來會筆記關於 autowire的部份，如果有興趣深入，可以參考這堂 udemy上的spring課程，雖然上課程非常快，但是記錄成筆記相對來說會花上三到四倍的時間來釐清一些概念。

