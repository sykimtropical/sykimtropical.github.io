---
layout: post
title:  "리눅스 포트 Permission Denied"
date:   2025-03-20 11:00:00 +0900
category: [네트워크]
tags: [포트, 네트워크]
lastmod : 2025-03-20 11:00:00 +0900
sitemap :
changefreq : daily
priority : 1.0
---

linux 서버에서 Tomcat의 포트를 80으로 수정 후 실행 시 80포트 관련하여 permission denied 가 뜨는 경우가 있다.<br/>
root 계정이 아닌 일반 사용자 계정으로 시도 했을 때 만날 수 있는 오류인데, linux에서는 특정 포트 구간은 일반 사용자 권한의 접근이 제한되어있다.<br/>
<br/>

# Linux 0~1023 포트 실행 권한

포트 번호는 크게 3가지로 나뉘어진다.<br/>
0~1023 : well-known port<br/>
1024~49151 : registered port<br/>
49512~65535 : dynamic port<br/>
<br/>
이 중 well-known port는 기본적으로 root 권한으로만 실행이 가능하다.<br/>
(일반 사용자 계정으로 해당 포트 프로그램 실행 시 오류 발생 할 수 있음)<br/>
