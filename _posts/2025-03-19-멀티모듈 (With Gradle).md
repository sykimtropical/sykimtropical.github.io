---
layout: post
title:  "멀티모듈 (With Gradle)"
date:   2025-03-19 11:00:00 +0900
category: [Java]
tags: [Java, 멀티모듈]
lastmod : 2025-03-19 11:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

# 멀티모듈이란?

멀티모듈이란 **하나의 단일 프로젝트를 여러개의 모듈로 분리해서 구성**하는 기법이다.  <br>
각 모듈은 개별적으로 컴파일되어 라이브러리 또는 실행가능한 파일로 생성된다.<br>
<br>

기존 싱글 모듈로 구성된 프로젝트의 경우 아래와 같은 구조를 가지고 있다.<br>

![](/assets/img/2025-03-19-MultiModule/멀티모듈-각 모듈별 프로젝트 구조.png)

`Member`라는 객체를 각각 모듈이 가지고있기 때문에 중복되는 코드가 많으며, 유지보수 시 Member 객체에 변경이 있으면 3개의 모듈에 싱크를 맞추는데 신경써야한다.<br>

위 구조를 해결하기 위해 아래와 같이 사설 저장소를 이용하는 방법이 있다.<br>

![](/assets/img/2025-03-19-MultiModule/멀티모듈-프로젝트 모듈화 기존.png)

이제 사설 저장소에 공통된 코드를 라이브러리화 하여 배포하기 때문에 각 모듈마다 싱크를 신경쓸 필요가 없어졌다.<br>

그러나 공통된 코드(Member)를 수정하게 되면 매번 빌드하고 배포하는 사이클이 추가된다.<br>

이것은 코드가 단 한줄만 수정되더라도 해야하는 작업이기 때문에 번거로워진다.<br>

위 불편함들을 해결하기 위한 방법으로 멀티 모듈이 존재한다.<br>

![](/assets/img/2025-03-19-MultiModule/멀티모듈-멀티모듈 이용시 변화.png)

위에서 보던 2가지 방법과 다르게 **하나의 프로젝트**에서 모듈(서브 프로젝트)이 여러개로 변경되었다.<br>

그로인해 공통된 코드(Member)의 변경이 있어도 빌드/배포 가 필요 없어지며, 각 모듈마다 하나의 공통된 코드를 참조하기 때문에 코드의 싱크를 신경쓰지 않아도 되게 된다.<br>

---

# 멀티모듈 프로젝트 만들기

본문에선 Spring Boot와 Gradle을 이용하여 프로젝트를 구성한다.<br>

## **1. Parent 프로젝트**

### **1-1) 프로젝트 생성**

Parent 또는 Root 프로젝트는 모듈들을 묶어주는 가장 최상위 프로젝트이다.<br>

![](/assets/img/2025-03-19-MultiModule/멀티모듈-생성(1).png)

프로젝트명과 패키지, 버전 등을 확인한 후 `Gradle`로 설정하여 넘어간다.<br>

![](/assets/img/2025-03-19-MultiModule/멀티모듈-생성(2).png)

`Lombok`만 추가 해 준 후 프로젝트를 생성한다.<br>

### **1-2) 프로젝트 구성 – 디렉토리**

![](/assets/img/2025-03-19-MultiModule/멀티모듈-생성(3).png)      
프로젝트 생성 직후  <br>

![](/assets/img/2025-03-19-MultiModule/멀티모듈-생성(4).png)      
`src` 디렉토리만 삭제<br>

프로젝트 생성 후 **`src` 폴더를 삭제**한다.  <br>
Parent는 모듈 집합 프로젝트로 `java`가 필요없다.  <br>

## **2. 모듈 추가(1) – 공통 코드**

### **2-1) 모듈 프로젝트 추가**

이제 여기에 모듈로 프로젝트를 추가한다.  <br>
공통이 되는 코드 + 공통 모듈을 사용할 어플리케이션<br>

![](/assets/img/2025-03-19-MultiModule/멀티모듈-생성(5).png)

**New** > **Module** 선택<br>

![](/assets/img/2025-03-19-MultiModule/멀티모듈-생성(6).png)

![](/assets/img/2025-03-19-MultiModule/멀티모듈-생성(7).png)

**core-module** 이름으로 공통 코드를 처리할 어플리케이션을 추가한다.  <br>
생성 시 의존성 추가는 필요 없다. 앞으로는 parent Gradle에서 모듈 프로젝트들의 의존성을 한번에 관리할것이기 때문이다.<br>

### **2-2) 추가된 모듈 구성 변경**

![](/assets/img/2025-03-19-MultiModule/멀티모듈-생성(8).png)

▼

![](/assets/img/2025-03-19-MultiModule/멀티모듈-생성(9).png)

모듈 추가 후 모듈 프로젝트 내의 파란색 폴더 및 파일은 전부 삭제하여 오른쪽과 같이 만들어준다.<br>

삭제 목록)

- .gradle
- gradle
- gradlew
- gradlew.bat
- HLEP.md
- settings.gradle

![](/assets/img/2025-03-19-MultiModule/멀티모듈-생성(10).png)


▼

![](/assets/img/2025-03-19-MultiModule/멀티모듈-생성(11).png)


프로젝트 모듈 목록에서 자동으로 추가된 `core-module` 삭제<br>

![](/assets/img/2025-03-19-MultiModule/멀티모듈-생성(12).png)


위에서 모듈 목록 삭제 후 Gradle 목록에 남아있다면 삭제<br>

## **3. Gradle 설정 변경**

### **3-1) Parent – setting.gradle 설정**

Groovy <br>

```groovy
rootProject.name = 'example-root-project'
include 'core-module'
```

모듈 추가(2) 후 모듈이 위와같이 `include` 되어있는지 확인한다.  <br>
되어있지 않으면 위와같이 모듈명으로 해주면 된다.<br>

### **3-2) Parent – build.gradle 설정**

`Parent` 프로젝트의 초기 `build.gradle`은 아래와 같다.<br>

build.gradle (기존) <br>

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.1.0'
    id 'io.spring.dependency-management' version '1.1.0'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

위 설정을 아래와 같이 변경 한다. (모든 프로젝트 의존성과 각 모듈별 의존성을 한곳에서 정의한다)<br>

build.gradle (변경)<br>

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.1.0'
    id 'io.spring.dependency-management' version '1.1.0'
}

// 모든 프로젝트 적용
allprojects {
    group = 'com.example'
    version = '0.0.1-SNAPSHOT'
    sourceCompatibility = '17'

    repositories {
        mavenCentral()
    }
}


// 서브 프로젝트 전체 적용
subprojects {
    apply plugin: 'java'
    // 위(plugins-id)에서 불러온 plugin 을 적용 시켰으므로 버전을 명시하지
    // 않아도 자식 프로젝트에서 3.1.0 버전을 받게된다.
    apply plugin: 'org.springframework.boot'
    apply plugin: 'io.spring.dependency-management'

    dependencies {
        compileOnly 'org.projectlombok:lombok'
        developmentOnly 'org.springframework.boot:spring-boot-devtools'
        annotationProcessor 'org.projectlombok:lombok'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'
    }

}

// 서브 프로젝트(모듈) 개별 설정
project(':core-module') {
    dependencies {
      // 현재 추가된 의존성이 존재하지 않는다.
    }
}

```

### **3-3) 모듈 – build.gradle 설정**

기존 모듈의 `build.gradle`은 아래와 같다.<br>

build.gradle (기존) <br>

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.1.0'
    id 'io.spring.dependency-management' version '1.1.0'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

위 내용 중 의존성에 관한 내용은 Parent의 build.gradle 파일 하나로 관리하기 때문에 대부분의 내용이 삭제되며, 모듈 빌드 설정만 남긴다. (아래 코드 존재)<br>

build.gradle (변경)<br>

```groovy
bootJar {
    // 모듈이기 때문에 bootJar 빌드하지 않는다.
    enabled = false
}

jar {
    enabled = true
}
```

이후 **gradle 빌드** 후 **성공 확인** <br>

## **4. 모듈에 의존성이 추가 되었는지 테스트 하기**

위 과정에서 root 모듈에 Lombok을 서브 프로젝트 전체에 의존성 추가 하였고, core-module 프로젝트에 개별로 추가한 의존성은 없다.<br>

그렇다면 core-module 프로젝트 내에서 Lombok 의존성이 정상적으로 추가되었는지 확인해보자.<br>
<br>

`core-module` 프로젝트 내 DTO 클래스를 하나 생성하여 Lombok 어노테이션을 추가해보니 정상적으로 추가되는 걸 확인 할 수 있다.<br>

![](/assets/img/2025-03-19-MultiModule/멀티모듈-생성(13).png)


![](/assets/img/2025-03-19-MultiModule/멀티모듈-생성(14).png)


## **5. 모듈 추가(2) – 공통코드를 사용하는 client-module**

### **5-1) 모듈 추가** 및 구성 변경

‘**2. 모듈추가(1) – 공통모듈**‘ 과 동일하게 모듈을 추가한다.  <br>
이때 본문에서 모듈명은 `client-module`로 진행한다.<br>

### **5-2) gradle 설정 변경 (parent, module)**

‘**3. Gradle 설정 변경**‘ 과 동일하게 gradle 설정을 변경하는데 의존성 관리시 client-gradle은 아래 의존성을 추가한다.<br>  
`client-module`은 api 서버로 실행시키기 위해 **web 의존성**과 `core-module`의 **Test DTO 사용을 위해** `:core-module` 프로젝트를 추가했다.<br>

<br>

build.gradle<br>

```Groovy
project(':client-module') {
    dependencies {
        implementation project(':core-module')

        implementation 'org.springframework.boot:spring-boot-starter-web'

    }
}
```

gradle 수정 후 build 하여 문제가 없는지 확인한다.<br>

## 6. client-module 작업 및 테스트

### 6-1) client-module에서 Controller 추가

TestController.java<br>

```java
package com.example.clientmodule;

import com.example.coremodule.Test;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class TestController {
    
    @ResponseBody
    @GetMapping("/home")
    public String home(){
        Test test = new Test();
        test.setId(1L);
        test.setName("김서영");
        
        return test.toString();
    }
}
```

이때 import를 보면 `com.example.coremodule.Test` 인것을 확인 할 수 있다.<br>

실행 후 API 호출 결과<br>

![](/assets/img/2025-03-19-MultiModule/멀티모듈-결과.png)

## 7. 최종 Gradle 조회

위까지 프로젝트 구성을 완료한 후 Gradle 구조를 보면 아래와 같다.<br>

![](/assets/img/2025-03-19-MultiModule/멀티모듈-gradle 구조.png)

`example-root-project` 아래에 `client-module`과 `core-module`을 확인 할 수 있으며 각 subproject마다 추가된 의존성을 확인 할 수 있다.<br>

# 결론

>각 API간 통신에 사용되는 파라미터를 클래스(DTO 혹은 VO)로 관리하는 경우 멀티모듈 기능을 이용하면 <br>
>유지보수로 인한 파라미터 불일치 등의 휴먼오류와 복붙을 줄일 수 있다.
