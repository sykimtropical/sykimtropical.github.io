---
layout: post
title:  "mybatis connection pool 개수 감소"
date:   2025-03-27 11:00:00 +0900
category: [Java]
tags: [mybatis, Java, DB]
lastmod : 2025-03-27 11:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---
솔루션을 ibatis에서 mybatis로 변경 후 connection pool 이 특정 시점에 줄어드는 이슈를 확인 한 적이 있다.<br>
확인해보니 SqlSession객체를 이용해 작성되었는데, mybatis는 ibatis와 달리 자원해제 `close()` 메서드를 직접 호출해주는데 일부 finally 구문에 누락된 부분이 존재했기 때문이다.<br>
<br>
mybatis가 ibatis 를 보완해서 나중에 나온 버전인데 이상하다 싶어 찾아보니<br>
ibatis 때 처럼 SqlSession을 이용한 것이 mybatis를 제대로 활용하지 못한 문제였다.<br>
<br>
mybatis에서는 SqlSessionTemaplate 객체를 이용하며 이 객체에서 내부적으로 인터셉터를 통해 자동으로 `close()`를 호출해 자원 해제 문제를 해결한다.<br>
<br>
*Mapper 인터페이스를 이용한다면 내부적으로 SqlSessionTemplate를 이용하기 때문에 따로 신경 쓸 필요 없다.*

