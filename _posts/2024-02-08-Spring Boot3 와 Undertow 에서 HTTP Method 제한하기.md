---
layout: post
title:  "Spring Boot3 와 Undertow 에서 HTTP Method 제한하기"
date:   2024-02-08 10:00:00 +0900
category: [HTTP]
tags: [SpringBoot,Undertow,내장WAS]
lastmod : 2024-02-08 10:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---
>모의침투 결과 불필요한 HTTP Method가 허용되어 있어 이에 대한 조치가 필요했다.<br>
>보통 WEB이나 WAS 설정파일을 건드려 제한 할 수 있으나,<br>
>Spring Boot로 내장 WAS 를 사용중이며, Undertow를 사용중에 있어 이 환경의 설정을 처리하고 기록했다.

<br><br>

# 불필요한 메소드 사용 확인

<br>

cmd 창에서 아래 명령어를 통해 허용된 HTTP Method를 확인 할 수 있다.

```cmd
> curl -v -X OPTIONS https://{확인하고자 하는 웹URL}
#### 요청정보 ####
*   Trying [::1]:8070...
*   Trying 127.0.0.1:8070...
* Connected to localhost (127.0.0.1) port 8070
> OPTIONS / HTTP/1.1
> Host: localhost:8070
> User-Agent: curl/8.4.0
> Accept: */*
#### 요청정보 ####

#### 응답정보 ####
< HTTP/1.1 200 OK
< Allow: GET,HEAD,OPTIONS
< Connection: keep-alive
< Cache-Control: no-cache
< Pragma: no-cache
< Strict-Transport-Security: max-age=63072000; includeSubdomains; preload
< X-Frame-Options: SAMEORIGIN
< Content-Length: 0
< Date: Wed, 07 Feb 2024 06:23:37 GMT
<
* Connection #0 to host localhost left intact
#### 응답정보 ###
```

<br>
응답 정보 내용 중 Allow: GET,HEAD,OPTIONS 를 확인할 수 있다.
<br>
모의침투 검사 시 권고사항은 GET, POST외 모든 HTTP 메서드는 취약하다고 나타낸다.
<br>
<br>

# 적용하기

우선 최종 코드는 아래와 같다.
<br>
<br>
프레임워크 : Spring boot 3.1.0<br>
내장WAS : Undertow 3.1.0
<br>
<br>

```java
@ManagementContextConfiguration
public class ManagementContextCustomization {

    @Bean
    public UndertowServletWebServerFactory undertowCustomizer() {
       UndertowServletWebServerFactory factory = new UndertowServletWebServerFactory();
       factory.addDeploymentInfoCustomizers(deploymentInfo -> {

          deploymentInfo.addInitialHandlerChainWrapper(new HandlerWrapper() {
             @Override
             public HttpHandler wrap(HttpHandler handler) {
                // GET, POST, DELETE 메서드만 허용한다.
                HttpString[] allowedHttpMethods = {HttpString.tryFromString("GET"),
                   HttpString.tryFromString("POST"), HttpString.tryFromString("DELETE")};
                return new AllowedMethodsHandler(handler, allowedHttpMethods);
             }
          });
       });
       return factory;


    }

}
```
<br>
<br>
<br>
(1) 내장WAS 설정을 건드려야한다.<br>
<br>
스프링 부트의 내장WAS는 내부적으로 `ServletWebServerApplicationContext` 에서 실행되는데,<br>
이때 WAS설정값은 `UndertowServletWebServerFactory` 를 읽는다.<br>
(정확히는 사용하는 내장 WAS별 `ServletWebServerFactory` 를 읽는다. 나는 Undertow를 사용하기에 위와 같이 기술)<br>
<br>
<br>
(2) `UndertowServletWebServerFactory` 를 구성해야한다.<br>
<br>
어떤 설정을 읽는지 확인했으니 `UndertowServletWebServerFactory` 를 설정해보자.<br>
<br>

[UndertowServletWebServerFactory Spring boot 공식 문서](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/web/embedded/undertow/UndertowServletWebServerFactory.html) <br>

번역)<br>
UndertowServletWebServers를 만드는 데 사용할 수 있는 ServletWebServerFactory입니다.<br>
명시적으로 다르게 구성되지 않는 한, 공장은 포트 8080에서 HTTP 요청을 수신하는 서버를 생성합니다.<br>
<br>
아래 매서드 중 `addDeploymentInfoCustomizers(UndertowDeploymentInfoCustomizer... customizers)`  를 확인<br>
<br>
[UndertowDeploymentInfoCustomizer Spring boot 공식 문서](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/web/embedded/undertow/UndertowDeploymentInfoCustomizer.html) <br>
번역)<br>
Undertow `DeploymentInfo`를 사용자 정의하는 데 사용할 수 있는 콜백 인터페이스입니다.<br>
<br>
`DeploymentInfo` 객체는?<br>
배포에 관련된 정보를 담는 객체<br>
<br>
요청 처리 체인에 추가적인 로직을 적용하기 위한 인터페이스 HandlerWrapper 적용하기 <br>
[Undertow 공식문서](https://undertow.io/undertow-docs/undertow-docs-2.1.0/index.html#handler-chain-wrappers) <br>
<br>
핸들러 구성하기 <br>
[Undertow 공식문서](https://undertow.io/undertow-docs/undertow-docs-2.1.0/index.html#built-in-handlers-2) <br>
<br>
<br>
여기서 `Allowed Methods` 객체를 이용하여 허용할 메소드 항목을 지정하고 `addInitialHandlerChainWrapper` 를 통해 핸들러를 추가한다.<br>
<br>
핸들러를 추가한 `deploymentInfo` 객체를 Undertow로 된 내장WAS에 적용하기 위해<br>
`UndertowServletWebServerFactory` 에 `addDeploymentInfoCustomizers(deploymentInfo)`  해주면 된다.<br>
<br>
<br>

# 적용 결과

적용 후 HTTP Method를 확인한다.
```cmd
>curl -v -X OPTIONS https://{확인하고자 하는 웹URL}
*   Trying [::1]:8070...
*   Trying 127.0.0.1:8070...
* Connected to localhost (127.0.0.1) port 8070
> OPTIONS / HTTP/1.1
> Host: localhost:8070
> User-Agent: curl/8.4.0
> Accept: */*
>
< HTTP/1.1 405 Method Not Allowed
< Connection: keep-alive
< Content-Length: 0
< Date: Thu, 08 Feb 2024 00:47:16 GMT
<
* Connection #0 to host localhost left intact
```

<br>
<br>
OPTIONS 메서드로 요청 할 경우 아까와 달리 405 Method Not Allowed로 OPTIONS 메소드가 허용되지 않았다는 내용을 응답받았다.<br>
<br>
허용한 메소드 목록에 있는 GET,POST,DELETE으로 요청 할 경우에는 성공 200으로 응답 코드를 받을 수 있는 것을 테스트 할 수 있다.<br>
