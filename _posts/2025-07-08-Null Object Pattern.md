---
layout: post
title:  "Null Object Pattern"
date:   2025-07-08 11:00:00 +0900
category: [Java]
tags: [Java, NPE]
lastmod : 2025-07-08 11:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

널 오브젝트 패턴이란 아래 그림과 같이 Interface를 만들고 상속받는 클래스를 두가지로 만들어 <br>
하나는 Null 이 아닌 객체로, 하나는 Null 일때의 객체를 만든다.  <br>

![](/assets/img/2025-07-08-nullObjectPattern/NullObjectPattern.png) <br/>


이후 User가 Null 일 경우 NullMember 객체내에서 null을 반환하지 않고 처리할 수 있도록 메서드를 정의해둔다. <br>

```java
public interface User {  
    String getName();  
}
```

```java
@Getter @Setter  
@ToString  
public class Member implements User {  
    private String name;  
  
    @Override  
    public String getName() {  
        return name;  
    }  
}
```


```java
public class NullMember implements User {  
  
    @Override  
    public String getName() {  
        return "Unknown";  // null을 반환하지 않는다.
    }  
}
```


이러한 패턴을 사용 할 경우 아래와 같이 별도의 null 처리가 필요 없어지며 User라는 공통 인터페이스를 받아와 처리할 수 있게 된다. <br>

```java
public static void main(String[] args) {  
	// case 1 : User not Null  
	Member member = new Member();  
	member.setName("ksy");  
	User user = member;  

	System.out.println("user.getName() = " + user.getName());  

	// case 2 : User is Null  
	User nullUser = new NullMember();  
	System.out.println("nullUser.getName() = " + nullUser.getName());  

	// Null 여부에 따른 예외처리가 필요 없어짐.  
	// 간단한 예시라 와닿지 않을 수 있지만 service단에서 User 객체를 위와 같이 이용하게 되면  
	// NPE 에서 유리해질 수 있음.  
}  
```

<br>
Optional과 같이 NPE 방지를 위한 추가 기술들이 존재하므로 작업에서 좀 더 유리한 패턴과 기술을 적용하도록 하자. <br>
<br>

git : [https://github.com/sykimtropical/nullObjectPattern](https://github.com/sykimtropical/nullObjectPattern){:target="_blank"}
