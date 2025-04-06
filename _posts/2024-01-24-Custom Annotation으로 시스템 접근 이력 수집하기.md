---
layout: post
title:  "Custom Annotaion으로 시스템 접근이력 수집하기"
date:   2024-01-24 11:00:00 +0900
category: [Spring]
tags: [customannotaion, AOP]
lastmod : 2024-01-24 11:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---
코드 : [https://github.com/sykimtropical/annotation-loggin-aop.git](https://github.com/sykimtropical/annotation-loggin-aop.git) {:target="_blank"}

>사용자가 웹사이트에서 어떤 동작을 요청하였는지 로그를 수집하고 싶었다.<br>
>AOP 또는 Filter에서 어떤 클래스, 메서드를 실행하였는지 일일이 확인하여 텍스트로 변환 하는 건 너무 번잡해 보여<br>
>CustomAnnotation을 이용하기로 했다.

<br><br>

# Test Code 작성
우선 간단하게 샘플을 작성하였다. (Controller, Service, Html 까지)
<br><br>

**TestController**
```java

@Controller
@RequiredArgsConstructor
public class TestController {

    private final TestService testService;

    @GetMapping("/home")
    public String home(){
       // home에 진입하였습니다.
       System.out.println("/home call");
       testService.home();
       return "home";
    }

    @PostMapping("/home")
    public ResponseEntity<String> createHome(@RequestParam Map<String, String> params){
       // home 을 생성하였습니다.
       System.out.println("POST /home call");
       System.out.println("params.toString() = " + params.toString());
       testService.createHome(params);
       return ResponseEntity.status(HttpStatus.CREATED).body("생성이 완료되었습니다.");
    }


    @GetMapping("/list")
    public String list(){
       // list에 진입하였습니다.
       System.out.println("/list call");
       testService.list();
       return "list";
    }

}

```
<br><br>
**TestService**
```java

@Service
public class TestService {

    public String home(){
       // /home 호출 시 서비스 로직이 동작되었다고 가정합니다.
       System.out.println("TestService.home()");
       return "service return";
    }

    public void createHome(Map<String, String> params){
       // POST /home 호출 시 서비스 로직이 동작되었다고 가정합니다.
       System.out.println("TestService.createHome()");
    }

    public String list(){
       // /list 호출 시 서비스 로직이 동작되었다고 가정합니다.
       System.out.println("TestService.list()");
       return "list return";
    }

}
```

<br><br>
**home.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>HOME</title>
</head>
<body>
<h1>HOME 화면입니다.</h1>
<button onclick="callAjax();">post 요청</button>
</body>

<script src="http://code.jquery.com/jquery-latest.min.js"></script>
<script type="application/javascript">
    function callAjax() {
        let params = {
            data: "데이터입니다."
        };
        $.ajax({
            type: 'POST',
            url: '/home',
            data: params,
            success: function (data, status, xhr) {
                alert(xhr.responseText);
            },
            error: function (xhr, status, error) {
                // xhr : 응답 메시지 , status : error 고정 / error : 오류
                alert('code : ' + xhr.code, ', msg : ' + xhr.responseText);
            }
        });
    }

</script>

</html>
```

<br><br>
**list.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>List</title>
</head>
<body>
<h1>LIST 페이지 입니다.</h1>
</body>
</html>
```
<br><br>

# 시나리오
1. GET /home 을 조회하면 home 조회 라는 access log를 남긴다.
2. POST /home 을 요청하면 home 추가 라는 accesslog를 남긴다.
3. GET /list 를 조회하면 list 조회 라는 accesslog를 남긴다.

<br><br>

# AOP 추가

<br>

의존성 주입
**build.gradle**
```groovy
implementation 'org.springframework.boot:spring-boot-starter-aop'
```
<br><br>

## AOP 생성
**AOPConfig.java**
```java

@Configuration
@Aspect
@Slf4j
public class AOPConfig {

    @Pointcut("execution(* com.example.{패키지}.controller.*..*(..))")
    public void accessLogging() {}

    @Around("accessLogging()")
    public Object accessLog(ProceedingJoinPoint pjp) throws Throwable {
       // @AccessAnnotation 에 저장된 log메시지를 가져와 로깅
       MethodSignature signature = (MethodSignature) pjp.getSignature();
       if(signature.getMethod().getAnnotation(AccessAnnotation.class) == null){
          return pjp.proceed();
       }

       Action action = signature.getMethod().getAnnotation(AccessAnnotation.class).action();
       String accessLog  = signature.getMethod().getAnnotation(AccessAnnotation.class).log();

       log.info("action : {}, accessLog : {}", actiontoString(), accessLog);
       return pjp.proceed();
    }

}
```
<br><br>


## Enum 생성
```java
public enum Action {  
    CREATE, READ, UPDATE, DELETE  
}
```
<br><br>


## 커스텀 어노테이션 인터페이스 생성
**AccessAnnotation.java**
```java

/**
 * 컨트롤러에서 @AccessAnnotation(action="사용자 목록 조회") 으로 사용   <br>
 *
 * @param: log - ex.home 조회    <br><br>
 * @param: action - [CREATE, READ, UPDATE, DELETE] 중 택1
 * */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface AccessAnnotation {
    Action action();
    String log();
}
```
<br><br>

## Controller에서 매서드마다 커스텀 어노테이션 적용하기
**TestController.java**
```java

@Controller
@RequiredArgsConstructor
@Slf4j
public class TestController {

    private final TestService testService;

    @GetMapping("/home")
    @AccessAnnotation(log = "home 조회", action = Action.READ)
    public String home(){
       // home에 진입하였습니다.
       log.info("/home call");
       testService.home();
       return "home";
    }

    @PostMapping("/home")
    @AccessAnnotation(log = "home 생성", action = Action.CREATE)
    public ResponseEntity<String> createHome(@RequestParam Map<String, String> params){
       // home 을 생성하였습니다.
       log.info("POST /home call");
       log.info("params.toString() = " + params.toString());
       testService.createHome(params);
       return ResponseEntity.status(HttpStatus.CREATED).body("생성이 완료되었습니다.");
    }


    @GetMapping("/list")
    @AccessAnnotation(log = "list 조회", action = Action.READ)
    public String list(){
       // list에 진입하였습니다.
       log.info("/list call");
       testService.list();
       return "list";
    }

}
```

<br><br>

## 로그

적용 이후 localhost:8080/home 요청 시 로그
```bash
2024-01-22T15:55:31.103+09:00  INFO 11828 --- [nio-8080-exec-1] c.e.e.AOPConfig                          : action : READ, accessLog : home 조회
2024-01-22T15:55:31.105+09:00  INFO 11828 --- [nio-8080-exec-1] c.e.e.controller.TestController          : /home call
2024-01-22T15:55:31.106+09:00  INFO 11828 --- [nio-8080-exec-1] c.e.e.service.TestService                : TestService.home()
```

POST /home 요청 시 로그
```bash
2024-01-22T15:55:33.302+09:00  INFO 11828 --- [nio-8080-exec-2] c.e.e.AOPConfig                          : action : CREATE, accessLog : home 생성
2024-01-22T15:55:33.302+09:00  INFO 11828 --- [nio-8080-exec-2] c.e.e.controller.TestController          : POST /home call
2024-01-22T15:55:33.303+09:00  INFO 11828 --- [nio-8080-exec-2] c.e.e.controller.TestController          : params.toString() = {data=데이터입니다.}
2024-01-22T15:55:33.303+09:00  INFO 11828 --- [nio-8080-exec-2] c.e.e.service.TestService                : TestService.createHome()
```

GET /list 요청 시 로그
```bash
2024-01-22T15:55:43.043+09:00  INFO 11828 --- [nio-8080-exec-3] c.e.e.AOPConfig                          : action : READ, accessLog : list 조회
2024-01-22T15:55:43.044+09:00  INFO 11828 --- [nio-8080-exec-3] c.e.e.controller.TestController          : /list call
2024-01-22T15:55:43.044+09:00  INFO 11828 --- [nio-8080-exec-3] c.e.e.service.TestService                : TestService.list()
```

<br><br>

# 결론
1. AOPConfig.java 에서 `@Pointcut` 을 이용하여 controller 패키지만 AOP를 거치도록 처리
2. `(MethodSignature) pjp.getSignature()` 을 이용하여 처리할 매소드 정보를 가져오고 해당 메서드가 커스텀어노테이션(AccessAnnotation) 을 가지고 있는지 확인
3. 가지고 있지 않다면 추가 작업 없이 프로세스를 실행하도록 하고,
4. 가지고 있다면 해당 어노테이션에 등록된 action과 log의 데이터를 가져와 로그 출력 후 프로세스를 실행한다.

