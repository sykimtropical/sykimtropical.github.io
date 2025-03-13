---
layout: post
title:  "Node 삭제도 pod 우아하게 종료하기"
date:   2024-03-20 10:00:00 +0900
tags: [kubernetes,graceful shutdown]
lastmod : 2024-03-20 10:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

네이버 클라우드 쿠버네티스 서비스를 이용중에<br>
쿠버네티스 버전 업데이트를 진행하던 중 작업중인 POD가 삭제되는 문제가 발생했다.<br>
<br>
콘솔에서 제공되는 업데이트 방식은 2가지 존재한다.<br>
ex) 기존 사용 노드 2대 일 때<br>

1. **노드풀 업데이트**<br>추가 증가 노드 개수를 2, 서비스 불가 가능 노드 개수 0 으로 진행 할 경우 기존 노드풀에서 새로운 버전의 워커노드를 2대 추가 실행<br>새로운 워커노드 2대가 실행 완료되면 기존 워커노드 2대 제거<br>이 과정에서 pod들은 새로운 노드로 새로 생성됨<br><br>
2. **노드풀 추가 및 삭제**<br>새로운 노드 풀 생성 (워커노드 동일하게 2대 설정)<br>새 워커노드 2대가  실행 완료되면 기존 노드를 1대씩 drain<br>중지하는 node의 pod가 안전하게 종료되었는지, 새로운 노드에 정상적으로 배포되었는지 확인 하며 기존 워커노드 2대 순차적으로 중지(drain) 후 <br>기존 노드 풀 콘솔에서 삭제<br>

1번 방법으로 처음 진행하였더니 daemon이 현재 작업중에 있고 graceful shotdown 설정을 해 두었으나,<br>
daemon pod가 실행중인 worker node가 삭제되어버려 작업에 유실이 생겼다.<br>
<br>
찾아보니 k8s 에서 제공하는 drain 기능을 이용하면 node를 종료하기 전 pod에 안전하게 shutdown 명령을 날려준다하여 2번 방식으로 테스트 한 결과<br>
원하던 대로 daemon이 작업을 마치고 안전하게 종료되었으며, Node 삭제는 불편하지만 관리자가 확인 후 직접 삭제를 진행할 수 있었다.<br>
<br>
drain 명령어
```shell
kubectl drain  {중지할 node 명} --grace-period={graceful shutdown 초} --delete-emptydir-data --ignore-daemonsets
```
=> 선택한 노드의 pod를 다른곳에 이동 시키고, 스케줄링을 차단한다.<br>
이때 pod의 graceful shutdown을 제공한다.<br>

drain 이후 node 상태를 조회하면 아래와 같이 노드의 status 에서 `SchedulingDisabled` 를 확인 할 수 있다.
```shell
kubectl get nodes
NAME             STATUS                   ROLES    AGE     VERSION
worker-01   Ready,SchedulingDisabled     <none>   4d18h   v1.27.9
worker-02   Ready                        <none>   4d18h   v1.27.9

```

