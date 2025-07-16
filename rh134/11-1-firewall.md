# 서버 방화벽 관리
- firewalld 규칙을 사용하여 시스템 서비스에 대한 네트워크 연결을 수락하거나 거부

## 방화벽 아키텍처의 개념
### netfilter
- Linux 커널은 패킷 필터링, 네트워크 주소 변환, 포트 변환과 같은 네트워크 트래픽 작업에 필요한 netfilter 프레임워크를 제공
- netfilter는 네트워크 패킷이 지나갈 때 자동으로 실행되는 커널 레벨 기능
- netfilter 프레임워크에는 kernel 모듈이 시스템의 네트워크 스택을 통과하는 네트워크 패킷과 상호 작용하기 위한 후크 가 포함되어 있음
- netfilter 후크는 이벤트(예: 인터페이스에 들어오는 패킷)를 가로채고 다른 관련 루틴(예: 방화벽 규칙)을 실행하는 커널 루틴

| 후크 이름           | 설명                               |
| --------------- | -------------------------------- |
| **PREROUTING**  | 패킷이 들어오는 즉시 가장 먼저 처리             |
| **INPUT**       | 목적지가 현재 호스트인 경우                  |
| **FORWARD**     | 패킷이 다른 인터페이스로 전달되는 경우 (라우터 역할 시) |
| **OUTPUT**      | 현재 호스트에서 발생한 패킷                  |
| **POSTROUTING** | 패킷이 나가기 직전 최종 처리                 |

### nftables 프레임워크
- nftables는 방화벽 규칙을 설정하고 적용하는 시스템
- netfilter 위에서 동작
  - netfilter는 리눅스 커널 내부의 패킷 처리 엔진이고,
  - nftables는 이 netfilter 기능을 사용하는 설정 인터페이스
- Red Hat Enterprise Linux 9에서 nftables 프레임워크는 시스템 방화벽의 코어로, 더 이상 사용되지 않는 iptables 프레임워크를 대체
- nftables 프레임워크는 iptables에 비해 향상된 유용성과 더 효율적인 규칙 집합 등 다양한 이점을 제공
  - 예를 들어 iptables 프레임워크에는 프로토콜별로 규칙이 필요하지만, nftables 규칙은 IPv4 및 IPv6 트래픽에 동시에 적용

### firewalld 서비스
- `firewalld` 서비스는 동적 방화벽 관리자
- Red Hat Enterprise Linux 9 배포에는 firewalld 패키지가 포함되어 있음
- firewalld 서비스는 네트워크 트래픽을 영역으로 분류하여 방화벽 관리를 간소화
  - 할당된 네트워크 패킷 영역은 패킷의 소스 IP 주소 또는 들어오는 네트워크 인터페이스와 같은 기준을 따름
  - 각 영역에는 열려 있거나 닫혀 있는 포트 및 서비스의 자체 목록이 있습니다.
- 동작
  - firewalld 서비스는 시스템에 들어오는 모든 패킷의 소스 주소를 확인
  - 해당 소스 주소가 특정 영역에 할당되면 해당 영역의 규칙이 적용
  - 소스 주소가 영역에 할당되지 않으면 firewalld 서비스가 패킷을 들어오는 네트워크 인터페이스의 영역과 연결하며 해당 영역의 규칙이 적용
  - 네트워크 인터페이스가 영역과 연결되지 않은 경우에는 firewalld 서비스가 패킷을 기본 영역으로 전송
    - 기본 영역은 별도의 영역이 아닌 기존 영역에 할당된 타겟
- example
  | 구성 요소            | 설정 내용                            |
  | ---------------- | -------------------------------- |
  | `zone: public`   | 기본 영역 (default zone)             |
  | `zone: internal` | `interface: eth1` 연결             |
  | `zone: dmz`      | `source IP: 192.168.100.0/24` 연결 | 
  - 192.168.100.23 으로 들어온 경우 -> dmz 영역에 할당
  - eth1 로 들어온 경우 -> internal
  - Q. dmz가 eth2 네트워크 인터페이스에 등록된 zone이지만, ip가 192.168.101.11 이라면? -> dmz에 포함되었다고 보지 않음. 기본 zone 적용 

- 대부분의 영역에서는 트래픽이 특정 포트와 프로토콜 목록(예: 631/udp) 또는 사전 정의된 서비스 구성(예: ssh)과 일치하는 방화벽을 통과
- 일반적으로 허용된 포트 및 프로토콜 또는 서비스와 일치하지 않는 트래픽은 거부
- zone 별 설정
```
[root@servera ~]# firewall-cmd --permanent --zone=public --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: cockpit dhcpv6-client https ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```
| 항목                       | 값                                  | 설명                       |
| ------------------------ | ---------------------------------- | ------------------------ |
| **zone 이름**              | public                             | 해당 설정이 적용되는 zone 이름      |
| **target**               | default                            | 기본 정책 (별도 정책 없으면 기본 처리)  |
| **icmp-block-inversion** | no                                 | no면 icpm-block에 있는 icmp 타입만 차단, 반대면 적힌 값만 허용       |
| **interfaces**           | (없음)                               | 현재 이 zone에 연결된 인터페이스 없음  |
| **sources**              | (없음)                               | 이 zone에 소스 IP로 지정된 주소 없음 |
| **services**             | cockpit, dhcpv6-client, https, ssh | 허용된 사전 정의 서비스들           |
| **ports**                | (없음)                               | 직접 지정한 개별 포트 없음          |
| **protocols**            | (없음)                               | 허용된 별도 프로토콜 없음           |
| **forward**              | yes                                | 패킷 포워딩 허용                |
| **masquerade**           | no                                 | NAT 마스커레이드 비활성화          |
| **forward-ports**        | (없음)                               | 포트 포워딩 설정 없음             |
| **source-ports**         | (없음)                               | 소스 포트 기반 허용 규칙 없음        |
| **icmp-blocks**          | (없음)                               | 차단된 ICMP 타입 없음           |
| **rich rules**           | (없음)                               | 사용자 정의 고급 규칙(ex. 특정 IP만 ssh 접속 허용 따로 문법 존재)          |

- target 가능 값

| 값         | 설명                                                          |
| --------- | ----------------------------------------------------------- |
| `default` | 명시적으로 작성된 것만 허용 |
| `ACCEPT`  | zone에서 **모든 트래픽을 허용**함(trusted에서 사용)                                    |
| `REJECT`  | zone에서 **거부 응답을 반환하며 차단**함                                  |
| `DROP`    | zone에서 **응답 없이 조용히 차단**함 (스텔스)                              |


## 사전 정의된 영역
- firewalld 서비스는 사용자 지정할 수 있는 사전 정의된 영역을 사용

|   영역 이름  	|   기본 구성   |
| ---------------- | -------------------------------- |
| trusted |	들어오는 모든 트래픽을 허용 |
| internal |	나가는 트래픽과 관련이 없거나 ssh, mdns, ipp-client, samba-client 또는 dhcpv6-client 사전 정의 서비스(시작하는 home 영역과 동일)와 일치하지 않을 경우 들어오는 트래픽을 거부합니다. |합니다.
| public | 나가는 트래픽과 관련이 없거나 ssh 또는 dhcpv6-client 사전 정의 서비스와 일치하지 않을 경우 들어오는 트래픽을 거부합니다. 새로 추가된 네트워크 인터페이스의 기본 영역입니다. |
| external | 나가는 트래픽과 일치하지 않거나 ssh 사전 정의 서비스와 일치하지 않을 경우 들어오는 트래픽을 거부합니다. 이 영역을 통해 전달되어 나가는 IPv4 트래픽은 나가는 네트워크 인터페이스의 IPv4 주소에서 시작된 것처럼 마스커레이드 됩니다. |
| dmz |	나가는 트래픽과 일치하지 않거나 ssh 사전 정의 서비스와 일치하지 않을 경우 들어오는 트래픽을 거부합니다. |
| block |	나가는 트래픽과 관련되지 않은 경우 들어오는 모든 트래픽을 거부합니다. |
| drop | 나가는 트래픽과 관련되지 않은 경우(ICMP 오류에도 대응하지 않음) 들어오는 모든 트래픽을 드롭합니다. |


### 사전 정의 서비스
- firewalld 서비스에는 방화벽 규칙 설정을 간소화하기 위해 일반 서비스에 대한 사전 정의된 구성이 포함되어 있음
- 이 서비스 이름들은 firewalld에서 규칙을 설정할 때 포트 번호를 직접 입력하지 않아도 되게 해주는 일종의 "단축어"
- ex. `firewall-cmd --add-service=ssh --permanent`
- 사전 정의 Firewalld 서비스

| 서비스 이름            |설명                                                                                                                                        |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| **ssh**           | 원격 접속용 SSH 서버 서비스입니다. 보통 22번 포트(TCP)를 사용합니다.<br>⇒ 외부에서 이 시스템에 `ssh user@host`로 접속할 수 있도록 허용할 때 필요    |
| **dhcpv6-client** | IPv6 주소를 자동으로 받기 위한 DHCPv6 클라이언트입니다.<br>⇒ 시스템이 네트워크에서 자동으로 IPv6 주소를 받을 수 있게 하려면 이 포트를 열어야 함 |
| **mdns**          | mDNS(멀티캐스트 DNS)를 이용해 `.local` 도메인을 통한 장치 탐색을 가능하게 합니다.<br>예: `printer.local` 같은 이름으로 네트워크 기기를 찾을 수 있어요 |
| **cockpit**       | Red Hat 계열 시스템에서 제공하는 웹 기반 관리 도구              |

- `firewall-cmd --get-services` 명령을 사용하여 서비스를 나열할 수 있음
```
[root@host ~]# firewall-cmd --get-services
RH-Satellite-6 RH-Satellite-6-capsule amanda-client amanda-k5-client amqp amqps
apcupsd audit bacula bacula-client bb bgp bitcoin bitcoin-rpc bitcoin-testnet
bitcoin-testnet-rpc bittorrent-lsd ceph ceph-mon cfengine cockpit collectd
...output omitted...
```

## firewalld 데몬 
- 시스템 관리자가 firewalld 서비스와 상호 작용하기 위해 웹 콘솔 그래픽 인터페이스를 사용하거나 firewall-cmd 명령 사용
### 명령줄에서 방화벽 구성
- firewall-cmd 명령은 firewalld 데몬의 인터페이스
- firewalld 패키지의 일부로 설치되며, 명령줄에서 작업하는 것을 선호하는 관리자를 위해 또는 그래픽 환경이 없는 시스템 작업이나 방화벽 설정 스크립트 작업을 위해 제공
- `--permanent` 옵션을 지정하지 않는 한 대부분의 명령이 런타임 구성에서 실행
  - `--permanent` 옵션이 지정된 경우 firewall-cmd --reload 명령도 실행하여 해당 설정을 활성화해야 함
  - 이 명령은 현재의 영구 구성을 읽고 이 구성을 새 런타임 구성으로 적용
- 기본 명령어
  - `firewall-cmd --get-default-zone`: 현재 기본 영역을 쿼리
  - `firewall-cmd --set-default-zone=ZONE`: 기본 영역을 설정. 이 기본 영역은 런타임 및 영구 구성을 둘 다 변경
  - `firewall-cmd --add-service=SERVICE `: SERVICE에 대한 트래픽을 허용. --zone= 옵션을 입력하지 않은 경우 기본 영역이 사용
  - `firewall-cmd --remove-service=SERVICE`: 영역에 허용된 목록에서 SERVICE를 제거. --zone= 옵션을 입력하지 않은 경우 기본 영역이 사용
  - `firewall-cmd --reload`: 런타임 구성을 삭제하고 영구 구성을 적용
- example
  - 기본 영역을 dmz로 설정하고, 192.168.0.0/24 네트워크에서 발생하는 모든 트래픽을 internal 영역에 할당하고, mysql 서비스의 네트워크 포트를 internal 영역에서 열도록 수정
  - `[root@host ~]# firewall-cmd --set-default-zone=dmz`: dmz 디폴트 설정
  - `[root@host ~]# firewall-cmd --permanent --zone=internal --add-source=192.168.0.0/24 `: source 할당
  - `[root@host ~]# firewall-cmd --permanent --zone=internal --add-service=mysql`: 서비스 할당
  - `[root@host ~]# firewall-cmd --reload`


### Q. 같은 서비스에 대해 충돌되는 규칙이 있다면?
- 동일 zone에 패킷이 충돌되도록 할당 불가
```
firewall-cmd --permanent --zone=internal --add-source=192.168.0.0/24
firewall-cmd --permanent --zone=public --add-source=192.168.0.0/24
```
`Error: ALREADY_ENABLED: '192.168.0.0/24' already bound to zone 'internal'.`