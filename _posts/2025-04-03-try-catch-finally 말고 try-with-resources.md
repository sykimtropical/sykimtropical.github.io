---
layout: post
title:  "try-catch-finally 말고 try-with-resources"
date:   2025-04-03 11:00:00 +0900
category: [Java]
tags: [Java]
lastmod : 2025-04-03 11:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

자바에서 커넥션이나 스트림 같은 자원은 사용 후 해제를 해주지 않으면 성능 문제가 발생할 수 있다.<br>
(이전에 PreparedStatement 를 사용하면서 자원 해제 누락으로 인해 커넥션 풀에서 커넥션이 하나씩 감소하는 경험을 해본 적 있음)<br>
<br>
기존 try-catch-finally 를 이용한 자원 해제 예시

```java
BufferedReader br = null;  
try {  
    br = new BufferedReader(new FileReader("C:/Users/Desktop/AppData/Roaming/JetBrains/IntelliJIdea2023.2/scratches/test.txt"));    
} catch (IOException e) {  
    e.printStackTrace();
} finally {  
    if( br != null ) {       
	    try {          
		    br.close();       
		} catch (IOException e){          
			e.printStackTrace();       
		}    
	}
}
```

<br>
br 객체가 null이 아님을 확인 해야 하고, `close()` 를 실행하다가 오류가 날 수 도 있으며 finally  구분을 누락 할 경우 `close()` 누락 될 수 있다.<br>
위 코드에서 finally 구문을 주석 할 경우 `close` 가 수행되지 않는 걸 확인하였다. (확인 방법 맨 아래)<br>
<br><br>

try-with-resources를 이용한 코드 변경점
```java
try(BufferedReader br = new BufferedReader(new FileReader("C:/Users/Desktop/AppData/Roaming/JetBrains/IntelliJIdea2023.2/scratches/test.txt"))) {  
    br.readLine();  
} catch (IOException e) {  
    throw new RuntimeException(e);  
}
```

<br>
기존 try-catch-fianlly의 자원 해제 방식의 불편함을 해소하고자 자원을 자동적으로 해제하는 기능이 java 7 부터 도입 되었다.<br>
AutoCloseable 인터페이스를 구현한 객체들은 `try()` 괄호 안에 변수를 선언해서 사용하면, `try` 구문이 끝나는 시점에 자동으로  `close()` 가 호출된다.<br>
위 예시에서 사용된 `BufferReader`와 `FileReader` 는 타고 들어가면 `Reader` 라는  추상 클래스를 이용해 구현했는데 상속받은 `Closeable` 인터페이스가 `AutoCloseable` 를 상속받은 인터페이스임을 확인 할 수 있다.<br>
<br>
테스트 결과 `br.readLine()` 이 수행된 다음 `close()`가 수행하는 걸 디버그모드를 통해 확인할 수 있었다.<br>
<br>

![](/assets/img/2025-04-03-try-catch-finally 말고 try-with-resources 이미지/AutoCloseable.png)
<br><br>



자원 해제가 정상적으로 수행되었는지는 `InputStreamReader` 객체내 `close()` 에 브레이크 포인트를 잡고 각 테스트마다 확인하였음<br>
![](/assets/img/2025-04-03-try-catch-finally 말고 try-with-resources 이미지/자원 해제 실행.png)
<br/><br>
