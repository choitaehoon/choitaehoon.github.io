---
title: 인스타그램과 같은 SNS 플랫폼은 어떻게 많은 트래픽을 처리할까? - 2편(Sticky Session vs Session Clustering vs Session Storage)
date: 2020-10-08 14:15:00 
---

1편에서 데이터 정합성 이슈가 발생했습니다. 2편에서는 데이터 정합성 이슈를 해결하는 방안들을 제시하며 어떤 것이 프로젝트에 잘 어울릴지
살펴보겠습니다.

### Sticky Session

![session 구성](https://user-images.githubusercontent.com/33123391/95419269-42698500-0974-11eb-9230-dd134ee49a01.jpg)

Sticky Session은 **고정 세션**을 의미합니다. 위의 그림을 보며 설명하겠습니다.
User1 요청이 Was 1번으로 보내지고 User2 요청이 Was 2번으로 보내졌다면 그 이후의 모든 요청은 User1은 Was 1번, User2는 Was 2번으로 처리합니다.
사용자에 대한 요청은 항상 해당 세션에 대한 요청을 처음 처리한 동일한 시스템으로 라우팅 되기 때문에 데이터 정합성 이슈를 해결할 수 있습니다.

그렇다면 Sticky Session을 사용하면 되지 않을까? 라는 생각이 들지만, 요청을 처음 처리한 동일한 시스템으로 라우팅 되어 트래픽이 몰릴 수밖에 없습니다.
요청을 처음 처리한 서버에 트래픽이 몰리고 장애가 난다면 다른 서버로 라우팅 되어 세션을 다시 생성해야 합니다. 이러한 문제로 Sticky Session은 합리적인 방안이라고 보기 힘듭니다.

### Session Clustering

Session Clustering은 여러 대의 Was가 하나의 세션을 공유할 수 있도록 도와줍니다.
Session Clustering에는 몇 가지 방법이 있는데 하나하나씩 살펴보겠습니다.

DeltaManager

![session 구성](https://user-images.githubusercontent.com/33123391/95436201-89647400-098e-11eb-9af6-7f6082de2441.png)

DeltaManager 방식을 이용하면 모든 세션 저장소에 세션을 복제하여 데이터 정합성 이슈를 해결할 수 있습니다. 하지만 DeltaManager도 문제점이 있습니다.
각각의 서버 메모리는 모든 세션을 저장하여 많은 메모리를 사용하고 있으며 세션을 복제하기 위해 모든 서버에 요청을 보내 network 트래픽이 증가할 수 있습니다.
이 방식은 소규모 클러스터에 적합하지만 4대 이상의 서버를 운영한다면 권장하지 않는 방법입니다.

BackupManager

![session 구성 (1)](https://user-images.githubusercontent.com/33123391/95443339-6db19b80-0997-11eb-9ab4-0395e3985f78.png)

Primary Node와 Backup Node로 분리되어 모든 노드에 복제하지 않고 Backup Node에만 복제합니다. 하나의 노드에만 복제하기 때문에 DeltaManager의 단점을
커버할 수 있습니다. 하지만 만일 Proxy Node로 접속할 경우 세션에 대한 정보가 없기 때문에 세션을 요청해야 하는 문제점이 발생합니다.

**그렇다면 데이터 정합성 이슈를 해결하고 성능의 한계를 극복할 수 있을까?** 라는 고민이 생깁니다. 이 문제점을 해결하기 위해 Session Storage를 생각했습니다.

### Session Storage

![session storage](https://user-images.githubusercontent.com/33123391/95448663-a7d26b80-099e-11eb-94b3-47ecf8d5f9f6.png)

Session Storage를 공유하고 있어 하나의 서비스가 장애가 나더라도 다른 서버에서 연속적인 서비스가 가능하고 해결하고자 했던
데이터 정합성 이슈를 해결할 수 있어 저희 프로젝트에는 Session Storage를 선택했습니다. 모든 문제를 해결한 것 같지만, 서비스를 확장함에 따라 관리 비용이 증가하고 있으며 Session Storage도 장애가
발생할 가능성이 있습니다.

Session Storage로 인해 연속적인 서비스가 가능하고 데이터 정합성 이슈를 해결해주는 이유로 관리 비용이 증가하는 문제는 충분히 납득할 수 있습니다. 하지만 Session Storage에 대한 장애는
어떻게 처리해야 할지 고민해야 합니다. 이 고민을 하기 전 Session Storage의 저장소를 먼저 선택해야 합니다. **다음 편에서
어떠한 저장소를 선택해야 효율적인지 알아보겠습니다.**

참조 문서
* https://access.redhat.com/solutions/900933
* https://tomcat.apache.org/tomcat-8.5-doc/cluster-howto.html
* https://sarc.io/index.php/tomcat/111-tomcat-session-cluster-1