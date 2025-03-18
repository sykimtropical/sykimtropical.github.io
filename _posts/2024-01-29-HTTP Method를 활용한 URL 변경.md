---
layout: post
title:  "HTTP Method를 활용한 URL 변경"
date:   2024-01-29 11:00:00 +0900
category: [HTTP]
tags: [HTTPMethod,RESTful,HTTP]
lastmod : 2024-01-29 11:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---
<br>

>기존 URL은 행위가 포함되어 있었다. (Restful URL 규칙 위반)<br>
>ex) /000ResultList.do , /000ResultDetail.do /000Send.do...<br>
>URL 길이를 줄이고, HTTP 메서드를 적극 활용하여 URL에서 행위에 대한 내용을 삭제하고자<br>
>공부하여 새로 만드는 솔루션에 적용해보았다.<br>


# Restful 이란?
**Rest** : Representational State Tranfer의 약자로 웹을 이용 할 때 제약 조건들을 정의하는 소프트웨어 아키텍처 스타일<br>
HTTP URL 을 통해서 자원(Resource)을 명시하고 HTTP Method(`GET`, `POST`, `PUT`/`PATCH`, `DELETE`)를 통해 해당 자원(URL)에 대한 CRUD를 적용하는 것을 의미한다.<br>
<br>
<br>
<br>

# HTTP Method 종류
1. `GET`: 리소스  **조회**
2. `POST`: 요청 데이터 처리, 주로 **등록**에 사용
3. `PUT`: **리소스를 대체**(전체 변경), 해당 리소스가 없으면 생성
4. `PATCH`: 리소스 **부분 변경**
5. `DELETE`: 리소스 **삭제**

<br>

ex)
1. GET /members : 멤버 목록 **조회** (READ)
2. GET /members/{id} : {id}에 해당하는 멤버 상세 조회 (READ ... WHERE id)
3. POST /members : 멤버 **추가** (CREATE)
4. PUT /members/{id} : 멤버 **수정** (UPDATE all colums WHERE id)
5. PATCH/members/{id} body:name=홍길동 : 멤버 **부분 수정** (UPDATE name WHERE id)
6. DELETE /members/{id} : 멤버 **삭제** (DELETE .. WHERE id)

<br>
<br>

# 멱등성
멱등성이란 100번을 호출해도 결과가 같은 걸 뜻한다.<br>
이때, 외부 요인으로 인해 결과가 달라진건 고려하지 않는다. <br>(사용자를 조회할 때 조회 자체는 멱등하나 외부에서 POST, PUT 등으로 결과가 달라진건 고려하지 않는다.<br>

<br>

ex)
1. GET : 100번을 호출해도 결과가 동일하다. (**멱등 메서드**)
2. POST : 100번을 호출하면 100개의 데이터가 CREATE 되므로 결과가 동일하다고 볼 수 없다. (멱등 메서드가 **아님**)
3. PUT : 100번을 호출해도 결과를 대체할 뿐(UPDATE) 이므로 최종 결과는 동일하다고 볼 수 있다. (**멱등 메서드**)
4. DELETE : 100번을 호출해도 데이터는 1번만 삭제되므로 최종 결과는 동일하다고 볼 수 있다. (**멱등 메서드**)

<br>
<br>

# 적용 규칙

<br>
우리는 결론적으로 아래와 같이 사용하기로 했다.<br>

1. URL에 행위를 담지 않을 것<br>/memberList (X) -> GET /members
2. 수정(UPDATE)는 POST와 `@PathVariable`를 이용할 것<br>POST /members/{id} => 수정 AND POST /members => 등록
3. 기존 URL에 카멜 케이스(단어 간 구분은 대문자로)를 `-` 이용으로 바꿀 것<br>/allowIp -> /allow-ip
4. DELETE는 DB 데이터 삭제 규칙에 맞게 내부적으로 처리할 것<br>DELETE HTTP 메서드를 사용하긴 하지만 테이블의 데이터가 Soft Delete 일수도, Hard Delete 일수도 있다.<br>즉, DELETE /members/{id} 를 호출한다고 모두 delete 쿼리가 아닌 update 쿼리일 수 있다.

<br>

수정 행위에 대해 POST를 사용한 것은 이미 등록에서의 POST와 수정에서의 POST가 구분되기 때문이었는데,<br>
기왕 Restful 하게 바꾸기로 했으면 PUT 또는 PATCH를 이용할 걸 그랬다는 생각이 든다.<br>

<br>
<br>

# 결과
## AS-IS

```java
@Controller
@RequiredArgsConstructor
public class AllowIpController{
	// ...

	@RequestMapping(value="/addrList.do")
	public String addr(Model model, AddressDTO dto, HttpServletRequest request) {
	    model.addAttribute("dto", addressListService.service(dto));
	    return "system/address";
	}

	@RequestMapping(value="/addrInsert.do")
	public @ResponseBody String insert(AddressVO vo, HttpServletRequest request) {
	    return addressInsertService.service(vo);
	}

	@RequestMapping(value="/addrDelete.do")
	public @ResponseBody String delete(AddressDTO dto) {
	    return addressDeleteService.service(dto);
	}

	@RequestMapping(value="/addrUpdate.do")
	public @ResponseBody String update(AddressVO vo) {
	    return addressUpdateService.service(vo);
	}

}
```

<br>
ip 조회/추가/수정/삭제 가 제각각의 URL을 가지고 있다.<br>
<br>

## TO-BE

```java
@RequiredArgsConstructor
@Controller
public class AllowIpController {

	// ...

    @GetMapping("/allow-access-ip")
    public String findAllAllowIp(Model model) throws Exception {
       model.addAttribute("allowIpResponse", allowIpClient.findAllCategory().getBody());
       return "sys-mng/ipList";
    }

    @PostMapping("/allow-access-ip")
    public ResponseEntity<String> createAllowIp(@RequestBody @Valid CreateAllowIpRequest allowIpRequest) throws Exception{
       allowIpClient.createAllowIp(allowIpRequest);
       return ResponseEntity.status(HttpStatus.OK).body("IP가 추가되었습니다.");
    }

    @PostMapping("/allow-access-ip/{id}")
    public ResponseEntity<String> modifyAllowIp(@PathVariable Long id,
       @RequestBody @Valid ModifyAllowIpRequest allowIpRequest) throws Exception {
       allowIpClient.ModifyAllowIp(id, allowIpRequest);
       return ResponseEntity.status(HttpStatus.OK).body("IP가 수정되었습니다.");
    }

    @DeleteMapping("/allow-access-ip/{ids}")
    public ResponseEntity<String> deleteAllowIp(@PathVariable List<Long> ids) throws Exception {
       allowIpClient.deleteAllowIp(ids);
       return ResponseEntity.status(HttpStatus.OK).body("IP가 삭제되었습니다.");
    }

}
```

<br>
모든 URL은 /allow-access-ip 에서 `GET`,`POST`,`DELETE` 메서드와 `PathVariable을` 이용해 각자의 역할이 부여된다.<br>
<br>
<br>

처음 공부 할 때는 HTTP Method는 `GET`,  `POST` 정도만 배웠던 것 같다.<br>
민감한 정보는 `POST`로 외에는 `GET`으로 처리하면 된다고..<br>
<br>
또한 PathVariable 도 사용하지 않다 보니 CURD를 URL에 행위를 넣어 구분하였고,<br>
그러하다보니 여러명이 작업하면서 규칙이란 찾아볼 수 없게 되었다.<br>
<br>
<br>
url 규칙도 정할 겸 솔루션도 새로 만들 겸 겸사겸사 HTTP Method를 적극 활용해보고자 했다.<br>
공공 사업이 주 타겟이다 보니 DELETE 메서드가 보안취약점에 걸릴 수 있다는 점이 마음에 걸렸으나<br>
그러면 /d/* 형태로 url을 변경 하여 POST 메서드로 변경할 계획이었고,<br>
PUT/PATCH 메서드가 모호하여 수정은 PathVariable 값이 무조건 붙으므로 아예 사용하지 않은 점이 아쉽지만<br>
어플리케이션이 여러 모듈로 변경되고, FeignClient를 잦게 사용하게 되면서<br>
URL이 규칙성이 생겨 좀 더 편하게 개발 할 수 있게 된 것 같아 마음에 들었다.<br>
<br>


