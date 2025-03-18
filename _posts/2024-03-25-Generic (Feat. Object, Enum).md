---
layout: post
title:  "Generic (Feat. Object, Enum)"
date:   2024-03-25 21:00:00 +0900
category: [Java]
tags: [generic,Object,Enum]
lastmod : 2024-03-25 21:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

# Generic이란?

JDK 1.5 부터 지원된 기능으로 클래스나 메서드에서 사용할 **내부 데이터 타입을 컴파일 시 미리 지정** 할 수 있는 방법이다. <br>
예를 들어 `ArrayList<E>` 는 ArrayList의 요소 타입을 제네릭으로 처리하기 때문에 요소에 String, Integer 또는 커스텀 객체 등 데이터 타입을 다양하게 담을 수 있다. <br>
<br>
<br>
제네릭 타입 변수가 들어갈 자리에는 기본 타입은 사용이 불가능하다.<br>
래퍼 클래스(Wrapper Class)를  이용해야한다.<br>

```java
ArrayList<int> list = new ArrayList<int>(); // Type argument cannot be of primitive type 빌드 오류 !!
ArrayList<Integer> list2 = new ArrayList<>();
```

<br>

오라클에서 얘기하는 왜 제네릭을 사용해야하는가 : https://docs.oracle.com/javase/tutorial/java/generics/why.html <br>

<br>

# Generic 사용법

## 파라미터 사용법
클래스에 `<>` 안에 받고자 하는 타입 파라미터의 개수를 지정한다.(파라미터 명명규칙 참고) <br>

```java
class Product<K, V> {  // 꺽쇠 안에 받은 타입 파라미터가 아래 타입 변수로 선언된다
    private K key;
    private V value;

    public Product(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey(){  return key;  }

    public V getValue(){  return value;  }
}

...

Product<Integer, String> product = new Product<>(1, "sogogi");
Product<String, String> strProduct = new Product<>("d-01", "darkgogi");

```

## 메소드 사용법
메소드에서 제네릭을 사용할땐 아래와 같이 사용할 수 있다.<br>

```java
class Response<T> {
    T list;

    public void setList(T list){  // 제네릭을 파라미터로 받아온다
        this.list = list;
    }
    public T getList(){  // 제네릭을 리턴한다.
        return list;
    }
}
```

<br>
※ static 은 제네릭(T-타입변수)에 사용할 수 없다. <br>
<br>

# Generic 파라미터 명명규칙

| 인자  | 설명      |
| --- | ------- |
| T   | Type    |
| E   | Element |
| K   | Key     |
| V   | Value   |
| N   | Number  |
| R   | Result  |


# Generic 공변성 반공변성

간혹 아래와 같이 `<? super ..>` 또는 `<? extends ...>` 가 보이는데 무슨 의미일까? <br>

```java
-- Function interface 내부 메서드 중 발췌

default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
    Objects.requireNonNull(before);
    return (V v) -> apply(before.apply(v));
}
```

<br>

java 공변성과 반공변성에 대해 우선 알아야 한다. <br>
*직접 다루지 않음* <br>

<br>

결론 : `<? super T>` 는 `T` 에 넣는 타입 이상의 타입 자료로만 제한, `<? extends T>` 는 `T` 에 넣는 타입 이하의 타입 자료로만 제한한다는 의미 <br>

<br>

`<?>`  : 타입 변수에 모든 타입을 사용 가능 <br>
`<? extends T>` : T 타입과 T 타입을 **상속받는 자손 클래스 타입**만을 사용 가능 <br>
`<? super T>` : T 타입과 T 타입이 **상속받은 조상 클래스 타입**만을 사용 가능 <br>

<br>
<br>


# Generic VS Object

❓이전에도 Object가 최상위 객체 타입이기 때문에 Object를 이용하면 모든 객체 타입을 담을 수 있었는데? <br>

<br>
사용은 가능하다만 아래와 같은 차이점이 존재한다. <br>
<br>
Object를 이용할 경우 어떤 데이터 타입이든 다 담을 수 있다. <br>
그러나 반대로 담겨있는 데이터의 타입이 어떤 건지 알기 어렵다. <br>


```java
ArrayList<Object> objList = new ArrayList<>();
objList.add("String Data"); // String
objList.add(1);             // Integer
objList.add('Y');           // Char

int element = (int) objList.get(0); // 런타임 오류!! String 을 int로 캐스팅 시도 했으니 오류
```

<br>
Generic을 이용하면 지정한 데이터 타입만 담을 수 있게 된다. <br>

즉 꺼내오는 데이터 타입도 컴파일 단계에서 지정되므로 잘못된 데이터 타입을 담으려 하면 **컴파일 오류가 발생**한다. <br>


```java
ArrayList<String> genericList = new ArrayList<>();
genericList.add("String Data");
genericList.add(1);     // Required type: String 컴파일 오류!!

int element = genericList.get(0);   // Required type: int, Provided: String 컴파일 오류!! genericList에 담긴건 String이니 int element에 담을 수 없다는 뜻
```

<br>
실무에서 개발을 진행하다 보면 모호한 정의가 얼마나 위험한지 모두 알 것이다. <br>
<br>
두 번째는 Object를 이용 할 경우 최상위 타입이므로 사용하고자 하는 타입으로의 형변환이 필요하다. (Boxing, Unboxing) <br>

>boxing : 기본타입 -> 래퍼 클래스 <br>
>unBoxiong : 래퍼클래스 -> 기본타입

<br>

List 요소를 Object로 처리 한 list는 String 타입을 담은 뒤 꺼내올 때 UnBoxing 없이 꺼내올 수 없다. <br>
이러한 언박싱 과정에서 소비되는 리소스는 데이터가 많을수록 무시할 수 없는 수준이 될 수 있다. <br>

```java
ArrayList<Object> objList = new ArrayList<>();
objList.add("String Data"); // String

String data = objList.get(0); // cast expression to 'String'  컴파일 오류!!
String data = (String) objList.get(0); // 담긴 데이터의 자료형이 String 이 맞다면 성공
```


# Enum

열거형 상수로 class 위치에 대신 enum 을 작성하여 생성할 수 있다. <br>

```java
enum Status {
	WAIT, ING, SUCC, FAIL, ERROR
}
```

enum을 사용하여 정의 할 경우 자동으로 싱글톤 패턴으로 구현된다. <br>

## 사용법

텍스트 그대로 추출하기 <br>

```java
enum Status {
	WAIT, ING, SUCC, FAIL, ERROR
}

System.out.println("Status = " + Status.ERROR);
// Status = ERROR
```

각 상수별 추출할 텍스트 지정하기 <br>

```java
enum Status {
    WAIT(1),
    ING(2),
    SUCC(3),
    FAIL(4),
    ERROR(5);

    private final int value;

    Status(int value) {
        this.value = value;
    }

    public int value(){
        return value;
    }

}

System.out.println("Status = " + Status.ERROR);
System.out.println("Status = " + Status.ERROR.value());
// Status = ERROR
// Status = 5
```


