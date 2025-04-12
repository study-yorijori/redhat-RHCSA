# Redhat System Administration 1, Linux란?

Linux는 어디에나 있다.

Linux는 오픈소스 소프트웨어이기 때문이다.

일반적인 기업용 소프트웨어는 버그가 발생하면 공급사의 개발자가 있어서, 공급사의 개발자에 의해 버그를 해결하게 된다.

오픈소스의 경우 관심 있는 누구나 소스코드를 볼 수 있기 때문에, 문제가 더 빨리 식별되고 안전하다.

## 오픈소스 소프트웨어란?

오픈소스 소프트웨어는 누구든지 사용, 연구, 수정 및 공유할 수 있는 소스 코드가 있는 소프트웨어

일부 소프트웨어는 생성한 사람, 팀 또는 조직만 보거나 변경 또는 배포할 수 있는 `전용`, `closed` 소스 코드를 사용함. 전용 라이센스는 일반적으로 사용자가 프로그램 실행만 가능하도록 제한하고 소스에 대한 액세스를 제한하거나 제공하지 않음

> 일례로 대부분의 게임 클라이언트 설치해서 사용할 때, 소스코드 수정해서 사용하도록 허용하지 않는 것 처럼...

오픈소스 소프트웨어는 사용자에게 프로그램을 실행할 수 있는 권한은 물론 소스를 보고, 수정하고, 컴파일하고 로열티 없이 다른 사람에게 재배포할 수 있는 권한을 부여함. 심지어 오픈소스 라이센싱은 더 많은 사람이 소프트웨어를 수정 및 개선하고 개선 사항을 더 광범위하게 공유하도록 권장함

상업적 용도로 오픈소스 소프트웨어를 제공할 수도 있음. 일부 오픈소스 라이센스는 전용 제품에서 코드를 재사용하도록 허용함. 누구든지 오픈소스 코드를 판매할 수 있지만, 일반적으로 오픈소스 라이센싱은 고객이 소스 코드를 재배포하도록 허용함. Red Hat과 같은 오픈소스 벤더는 오픈소스 제품에 기반한 솔루션의 배포, 관리, 빌드를 위한 상업적 지원 서비스 제공

### 오픈소스 라이센스 유형

- **카피레프트(copyleft) 라이센스**
  - 코드를 오픈소스로 유지하도록 권장
  - **변경 사항의 유무와 관계없이 다른 사람에게 코드를 복사, 변경 및 배포할 수 있는 자유를 제공**해야 함
  - 기존 코드와 해당 코드의 개선 사항을 오픈 상태로 유지하고 사용 가능한 오픈소스의 양을 늘리는데 도움이 됨
  - GNU GPL(일반 공중 라이센스), LGPL(완화된 GNU 공중 라이센스)가 포함됨
- **퍼미시브(permissive)**
  - 코드 재사용성을 최대화
  - **보다 제한적인 라이센스 또는 전용 라이센스로 코드를 재사용하는 것을 포함해, 저작권 및 라이센스 조항이 유지되는 한 어떠한 목적으로든 소스 사용 가능**
  - **코드를 쉽게 재사용할 수 있지만 전용 개선 사항이 증가할 위험이 있음**
  - MIT/X11 라이센스, Simplified BSD 라이센스, Apache Software License 2.0 등이 있음

## Linux 배포판이란?

Linux 커널과 Linux 배포판을 구별할 필요가 있다.

**Linux 배포판은 Linux 커널을 기반으로 구성되고 사용자 프로그램 및 라이브러리를 지원하는 설치 가능한 운영체제이다. 배포판을 사용해서 현장에서 Linux 시스템을 쉽게 설치하고 관리할 수 있다.**

커널은 운영 체제의 핵심이며 하드웨어, 메모리 및 실행 중인 프로그램의 스케줄링을 관리한다. Linux 커널은 MIT X Window System의 그래픽 인터페이스인 GNU 프로젝트의 유틸리티 및 프로그램을 비롯한 기타 오픈소스 소프트웨어로 보완된다. Linux 커널에는 Sendmail 메일 서버 및 Apache HTTP 웹 서버와 같은 기타 오픈소스 구성 요소도 포함되어 있어 완전한 오픈소스 UNIX와 유사한 운영 체제가 된다.

Linux 사용자는 다양한 소스에서 이런 모든 소프트웨어를 조합하며, 초기 Linux 개발자는 사용자가 Linux 시스템을 신속하게 구현하기 위해 다운로드하여 설치할 수 있는 미리 빌드되고 테스트된 툴을 배포했다.

각기 다른 목표와 지원 기준을 가진 많은 Linux 배포판이 있는데, 일반적으로 배포판에는 다음과 같은 공통점이 있다.

- 배포판은 Linux 커널과 지원하는 사용자 환경 프로그램으로 구성된다.
- 배포판은 단일 용도의 소규모이거나, 수천 개의 오픈소스 프로그램을 포함할 수 있다.
- 배포판은 소프트웨어와 해당 구성 요소를 설치하고 업데이트하는 수단을 제공한다.
- 배포판 프로바이더는 소프트웨어를 지원하며, 개발 커뮤니티에 참여한다.

### Red Hat이란?

Red Hat은 오픈소스 소프트웨어 솔루션 분야의 프로바이더

레드햇은 fedora, centos, RHEL 리눅스 배포판을 보유함

![ecosystem](https://static.ole.redhat.com/rhls/courses/rh124-9.3/images/getstarted/whatislinux/images/getstarted/ecosystem.svg)

|                       | Fedora    | CentOS Stream     | RHEL    |
| --------------------- | --------- | ----------------- | ------- |
| 예상 라이프사이클     | 12~18개월 | 5년               | 10년    |
| 소프트웨어 벤더 인증  | 아니요    | 일반적으로 인증 X | 예      |
| 설명서 제공           | 커뮤니티  | 커뮤니티          | Red Hat |
| 전문가 지원 사용 가능 | 아니요    | 아니요            | 예      |
| 제품 보안 팀          | 아니요    | 아니요            | 예      |
| 보안 인증             | 아니요    | 아니요            | 예      |
| 무료 옵션             | 예        | 예                | 예      |
| 관리 툴               | 아니요    | 아니요            | 예      |

- **Fedora**
  - community support
  - **모든 새롭고 시험적인 것들이 Fedora에 들어감**
  - 주요 업데이트가 6개월마다 출시되며 변경 사항을 제공하며, 프로덕션 용도로 적합하지 않음
- **Extra Packages for Enterprise Linux(EPEL)**
  - Fedora 프로젝트 SIG(Special Interest Group)는 EPEL(Extra Packages for Enterprise Linux)이라는 커뮤니티 지원 패키지 리포지토리를 빌드하고 유지 관리함. EPEL 버전은 주요 RHEL 릴리스에 맞게 조정되며 **RHEL 고객이 RHEL에서 지원되지 않는 소프트웨어 종속성으로 워크로드를 실행할 수 있도록 함**. EPEL 패키지는 Red Hat 지원에 포함되지 않지만 Fedora의 품질 수준과 동일함
  - 일반적으로 EPEL 패키지는 RHEL 릴리스를 기준으로 빌드됨. EPEL Next는 패키지 유지 관리자가 CentOS Stream을 기준으로 빌드하는 추가 리포지토리. 이 리포지토리는 CentOS Stream에 예정된 RHEL 라이브러리 리베이스가 포함되어 있거나 CentOS Stream에는 이미 있지만 RHEL에 아직 없는 최소 버전 빌드 요구사항이 EPEL 패키지에 있는 경우에 유용함.
  - **EPEL 패키지는 기본 Enterprise Linux 배포판에 포함되지 않은 소프트웨어를 설치할 수 있도록 해주는 확장 저장소**이며, 일반적으로 Fedora에서 제공되는 패키지를 기반으로 함. **기본 Enterprise Linux 배포판의 패키지와 충돌하거나 이를 대체하지 않아야** 함. 
- **CentOS Stream**
  - community support
  - RHEL용 업스트림 프로젝트
  - **Fedora에서 안정성, 보안, 성능 및 고객 요구 사항이 완성된 것으로 간주되는 경우 CentOS Stream에 포함**
  - 2019년 이전의 CentOS Linux는 각 주요 RHEL 릴리스 이후 Red Hat 소스코드로 커뮤니티에서 빌드된, 지원되지 않는 무료 배포판이었음. 하지만 RHEL 릴리스와 CentOS 배포판 빌드 사이에 상당한 지연이 발생하는 단점이 있어 Red Hat이 CentOS Stream 모델로 전환함
- **RHEL(Redhat enterprise Linux)**
  - **Centos에서 안정적이고, 인증된 것들이 RHEL**
  - 레드햇은 구독기반 지원 모델을 제공함

### RHEL for Edge

RHEL의 이미지 기반 변형임. 

RHEL은 Image Builder라는 툴을 통해 특수 용도의 운영 체제 이미지를 생성하는 기능을 제공하며, RHEL 이미지를 빌드, 배포, 유지 관리할 수 있음. 이미지 기반 배포는 다양한 엣지 아키텍처에 최적화되어 있지만 특정 엣지 배포에 맞게 사용자 지정할 수 있다.

RHEL의 엣지 기능에는 단일 인터페이스 내에서의 제로 터치 프로비저닝, 시스템 상태 가시성 및 빠른 보안 문제 해결을 비롯한 보안 관리 및 확장 기능이 포함됨

> **엣지 아키텍처란?**
>
> 엣지 아키텍처는 데이터와 서비스의 처리를 사용자가 위치한 물리적 또는 논리적 경계(엣지) 가까이에서 수행하는 구조.
>
> **엣지 아키텍처의 주요 구성 요소**
>
> - 엣지 디바이스: IoT 기기, 센서, 게이트웨이, 로봇 등 분산 환경에서 데이터 처리 및 애플리케이션 실행을 담당합니다.
> - 엣지 게이트웨이: 디바이스와 클라우드 간의 중계 역할을 하며, 데이터를 집계 및 필터링합니다.
> - 엣지 데이터 센터: 엣지 장치가 생성한 데이터를 지역적으로 저장, 분석 및 처리하는 소규모 데이터 센터
>
> 엣지 아키텍처는 데이터가 발생하는 현장(엣지) 가까운 곳에서 처리하여 지연(latency)을 줄이고 실시간 처리를 가능하게 하며, 스마트 팩토리, 자율주행차, 스마트 시티 등의 사례에서 활용

### Red Hat CoreOS

RHCOS(RHEL CoreOS)는 RHEL 구성 요소에서 빌드된 다음, 클라우드 네이티브 애플리케이션용 RHOCP(Red Hat OpenShift Container Platform)의 일부로 릴리스, 업그레이드, 관리된다. RHCOS는 기본적으로 RHOCP에 통합된 CRI-O(Container Runtime Interface) 규격 컨테이너 엔진을 사용하는 이미지 기반 RHEL 컨테이너 호스트이다.

### Red Hat Universal Base Image

Red Hat UBI(Universal Base Image)는 기본적으로 RHEL에서 자유롭게 재배포할 수 있다. UBI는 컨테이너에서 개발된 클라우드 네이티브 및 웹 애플리케이션 사용 사례의 기반이 되도록 설계됨. 

UBI를 사용하면 개발자가 컨테이너 이미지의 애플리케이션에 집중할 수 있으며, UBI는 기본 이미지 세트와 애플리케이션 이미지 세트(예: python, ruby, node.js, httpd, nginx)이다. 



### RHEL CD(Continous Development)

![redhat-fedora-centos-releases](https://static.ole.redhat.com/rhls/courses/rh124-9.3/images/getstarted/whatislinux/images/getstarted/redhat-fedora-centos-releases.svg)

Fedora 업스트림 커뮤니티에서 Fedora Rawhide는 정기적인 공개 Fedora 릴리스를 위한 지속적인 개발 환경이다.

## Red Hat Enterprise Linux 얻기

- Fedora CoreOS의 새로운 버전을 포함하여 *Fedora Linux* 및 파생 제품은 [https://﻿getfedora.org/](https://getfedora.org/)의 Fedora 프로젝트에서 무료로 사용 가능
- *EPEL* 및 *EPEL Next* 패키지는 EPEL 프로젝트 리포지토리에서 무료로 사용 가능, https://docs.fedoraproject.org/en-US/epel/
- *CentOS Stream*은 https://www.centos.org/centos-stream/에서 무료로 사용 가능