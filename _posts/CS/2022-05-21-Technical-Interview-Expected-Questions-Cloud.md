---
title: '[CS] 기술면접 예상 질문 - 클라우드'
layout: post
categories: cs
tags: cs
comments: true
---

기술면접 대비 클라우드 관련 개념 정리

### 1. CNI
CNI란 **컨테이너 간의 네트워킹을 제어할 수 있는 플러그인을 만들기 위한 표준**이다. 다양한 형태의 컨테이너 런타임과 오케스트레이터 사이의 네트워크 계층을 구현하는 방식이 다양하게 분리되어 각자만의 방식으로 발전하게 되는 것을 방지하고 공통된 인터페이스를 제공하기 만들어 졌다. 쿠버네티스에서는 Pod 간의 통신을 위해서 CNI 를 사용한다.  

쿠버네티스 뿐만 아니라 Amazon ECS, Cloud Foundry 등 컨테이너 런타임을 포함하고 있는 다양한 플랫폼들은 CNI를 사용하고 있다. 쿠버네티스는 기본적으로 'kubenet' 이라는 자체적인 CNI 플러그인을 제공하지만 네트워크 기능이 매우 제한적인 단점이 있다.  

그 단점을 보완하기 위해, 3rd-party 플러그인을 사용하는데 그 종류에는 Flannel, Calico, Weavenet, NSX 등 다양한 종류의 3rd-party CNI 플러그인들이 존재한다.

### 2. Calico
Calico란, **컨테이너, 가상 머신 및 기본 호스트 기반 워크로드를 위한 오픈 소스 네트워킹 및 네트워크 보안 솔루션**이다. Kubernetes, OpenShift, Mirantis Kubernetes Engine(MKE), OpenStack 및 베어메탈 서비스를 포함한 광범위한 플랫폼을 지원한다. Calico의 eBPF 데이터 플레인을 사용하든 Linux의 표준 네트워킹 파이프라인을 사용하든 Calico는 진정한 클라우드 네이티브 확장성과 함께 놀랍도록 빠른 성능을 제공한다.  

Calico는 공용 클라우드나 온프레미스, 단일 노드 또는 수천 개의 노드 클러스터에서 실행되는지 여부에 관계없이 개발자와 클러스터 운영자에게 일관된 경험과 기능 세트를 제공한다.

### 3.Service Mesh
MSA를 적용한 시스템의 내부 통신이 Mesh 네트워크의 형태를 띠는 것에 빗대어 Service Mesh로 명명되었다.
Service Mesh 는 서비스 간 통신을 추상화하여 안전하고, 빠르고, 신뢰할 수 있게 만드는 **전용 InfraStructure Layer**이다.
추상화를 통해 복잡한 **내부 네트워크를 제어**하고, 추적하고, 내부 네트워크 관련 로직을 추가함으로써 안정성, 신뢰성, 탄력성, 표준화, 가시성, 보안성 등을 확보한다.

Service Mesh 는 URL 경로, 호스트 헤더, API 버전 또는 기타 응용 프로그램 수준 규칙을 기반으로하는 **7 계층 네트워크 Layer** 이다.
Service Mesh 의 구현체인 경량화 Proxy를 통해 다양한 Routing Rules, circuit breaker 등 공통기능을 설정할 수 있다. 이는 서비스 간 통신에 연관된 기능 뿐만 아니라, 서비스의 배포 전략에도 도움을 준다.

### 4. istio
- Google, IBM, Lyft가 함께 기여하고 있는 오픈소스 Service Mesh 구현체
- kubernetes를 기본으로 지원하며 Control Plane — Data Plane 구조로 동작한다
- Envoy를 기본 Proxy로 wrapping하여 사용하지만 nginx나 linkerd 등으로 대체할 수 있다.
    - Envoy: connection 풀 동작 방식이나 어디서에서 connection이 끊겼는지도 통계로 확인 가능

service에 sidecar를 붙여서 실행되고 iptable을 통해 모든 트래픽을 제어한다. 제어된 모든 트래픽은 istio-proxy를 통해 나가고 받는다.

- 1% Canary 가능
  - istio route weight로 1% canary 배포 가능
  - 인스턴스 단위로 canary를 하면 세밀한 트래픽 조절이 힘듬. 10개 인스턴스에서 1개의 카나리가 나가면 10% 카나리가 됨. 하지만 istio route weight를 사용하면 인스턴스 개수와 상관없이 1% 카나리 배포가 가능하다.
- failure injection test와 squeeze test
  - failure injection test: 특정조건에 맞는 요청의 경우 실패 또는 응답을 늦게 하도록 만들어 서비스에서 실패에 대한 처리가 제대로 되어 있는지 확인할 수 있는 테승트
  - squeeze test: 하나의 인스턴스에 요청 비율을 높여 부하를 높이는 테스트

### 5. Ceph
![](/assets\img/CS/Ceph_Operation.jpg)
- 분산형 스토리지로 **여러 스토리지들을 클러스터로 묶어 하나로 보이게 하는 스토리지**
- 장점: 분산으로 저장하여 복구에 용이하고 하나의 클러스터로 묶어 클러스터를 바라보게만 하면 됨
- 파일, 블록, 오브젝트 등 다양하게 제공하며 **RADOS**라는 것을 사용해 실질적인 기능을 구현함
- Data가 들어오면 Data의 종류의 따라 Object로 변환되고 변환된 Object는 Libados에 따라 이후과정은 룰에 따라 OSD에 저장 됨

### 6.장애조치/실패허용(fail-over)
특정머신이 다운될 때 중복 백업 머신으로 전환하는 기술로 고가용성을 달성하기 위한 매우 일반적인 구현 방법. 종종 4계층 레이어, 7계층 레이어와 같은 다양한 로드 밸런싱 기술과 혼합되어 사용된다.
