---
layout: post
title:  "ArgoCD 서버 0,1 Running 오류"
date:   2025-04-03 11:00:00 +0900
category: [오픈소스]
tags: [오픈소스, ArgoCD, 클라우드, kubernets, k8s]
lastmod : 2025-04-03 11:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---

NKS 버전 업데이트로 인해 WorkerNode를 최신 버전으로 신규 생성 후 drain 작업<br>
신규 WorkerNode에서 Argocd 실행 시 오류 발생<br>
※ argocd-dex-server-{uuid} 포드 생성 오류 <br>
<br>

# argocd-controller-0, argocd-server 0/1 Running
분명 오류 없이 Running 상태로 진입 했으나 deployment가 0/1 (전체 1개 중 0개 실행중) 으로 조회되는 현상 발생<br>
아래는 해당 포드 로그 ( $ kubectl logs -f argocd-controller-0 )<br>

```shell
time="2025-01-06T02:50:23Z" level=warning msg="The cluster https://kubernetes.default.svc has no assigned shard."
W0106 02:50:29.368437       7 reflector.go:561] pkg/mod/k8s.io/client-go@v0.31.0/tools/cache/reflector.go:243: failed to list *v1alpha1.Application: applications.argoproj.io is forbidden: User "system:serviceaccount:default:argocd-application-controller" cannot list resource "applications" in API group "argoproj.io" at the cluster scope
E0106 02:50:29.368592       7 reflector.go:158] "Unhandled Error" err="pkg/mod/k8s.io/client-go@v0.31.0/tools/cache/reflector.go:243: Failed to watch *v1alpha1.Application: failed to list *v1alpha1.Application: applications.argoproj.io is forbidden: User \"system:serviceaccount:default:argocd-application-controller\" cannot list resource \"applications\" in API group \"argoproj.io\" at the cluster scope" logger="UnhandledError"
```

<br>
두개의 디플로이먼트 모두 동일한 오류 메시지 출력을 확인<br>
<br>

## 확인 : 
[https://github.com/argoproj/argo-cd/issues/18829](https://github.com/argoproj/argo-cd/issues/18829){:target="_blank"} <br>
위 글에서 apiGroups와 권한을 확인해 추가하라는 내용을 보고 수정하였으나, 권한이 존재함에도 불구하고 같은 오류 발생이 지속됨<br>
<br>

깃 내용과 다른 점은 <br>
<br>
깃 issue : 
```shell
Error: W0626 15:42:39.415329 7 reflector.go:539] pkg/mod/k8s.io/client-go@v0.29.6/tools/cache/reflector.go:229: failed to list *v1alpha1.AppProject: appprojects.argoproj.io is forbidden: User "system:serviceaccount:argocd:argocd-applicationset-controller" cannot list resource "appprojects" in API group "argoproj.io" in the namespace "argocd"
```

<br>
나의 오류 :

```shell
E0106 02:50:25.431516       7 reflector.go:158] "Unhandled Error" err="pkg/mod/k8s.io/client-go@v0.31.0/tools/cache/reflector.go:243: Failed to watch *v1alpha1.ApplicationSet: failed to list *v1alpha1.ApplicationSet: applicationsets.argoproj.io is forbidden: User \"system:serviceaccount:default:argocd-server\" cannot list resource \"applicationsets\" in API group \"argoproj.io\" at the cluster scope" 
```

<br>
두개의 마지막 부분이 다름을 확인 <br>

깃 : `in the namespace "argocd"` / 내 오류 : `at the cluster scope" `
<br>

## 결과 : 
현재 발생중인 오류는 위 이슈와 달리 configmap이 아닌 clusterbinding을 확인 하라는 오류였음

```shell
kubectl get clusterrolebinding 
```
 `argocd-application-controller`, `argocd-server` 없는 것 확인


  
[https://github.com/argoproj/argo-cd/tree/master/manifests/cluster-rbac](https://github.com/argoproj/argo-cd/tree/master/manifests/cluster-rbac){:target="_blank"} <br>
위 argocd 깃 경로에서 clusterbinding yaml 파일을 모두 가져와 합친 후 namespace를 default로 모두 변경&지정 해줌<br>
clusterbinding apply 후 argocd 재배포 하여 오류 해결<br>

<br>

---
보안 취약점 조치를 위해 ArgoCD 를 git에서 가져와 이미지를 직접 빌드 하고 있는데, 워커 노드를 아예 구성하게 되어 ArgoCD가 새로 배포되는 바람에 이젠에 apply 해두었던 설정들이 날아가면서 생긴 문제였다.<br>
clusterbinding 이 누락되어 배포되지 않는 바람에 생긴 문제로 해당 yaml 파일 재 생성 후 폴더와 install yaml 또한 전부 추가하여 수정해 두었다.<br>
