# 프로필을 활용한 성능 튜닝

- `tuned`는 시스템의 성능을 최적화하기 위한 데몬 기반의 유틸리티로, 다양한 워크로드에 맞는 튜닝 프로파일을 제공
- goal
  - `tuned` 데몬을 통해 정적 및 동적 튜닝을 수행
  - 사전 정의된 프로파일을 선택하여 성능/전력 소비 최적화

## 튜닝 구성 방식

### 1. 정적 튜닝 (Static Tuning)
- 시스템 부팅 시 또는 프로파일 변경 시 커널 매개 변수를 설정
- 활동 수준 변화와 무관하게 항상 고정된 값 유지

### 2. 동적 튜닝 (Dynamic Tuning)
- 시스템 활동을 실시간 모니터링하고 설정 조정
- 기본 비활성화 상태  
  → `/etc/tuned/tuned-main.conf` 파일에서 `dynamic_tuning = 1`로 설정
```ini
# /etc/tuned/tuned-main.conf 예시
# update_interval 초단위 작성
dynamic_tuning = 1
update_interval = 10
```

## 동적 튜닝 플러그인 구성
### Monitor 플러그인
| 이름  | 설명                                  |
|-------|---------------------------------------|
| `disk` | 디스크 I/O 수 모니터링                |
| `net`  | NIC 당 전송 패킷 수 모니터링          |
| `load` | CPU 부하 모니터링                     |

### Tuning 플러그인
| 이름   | 설정 대상                           |
|--------|--------------------------------------|
| `disk` | 디스크 스케줄러, 스핀다운, APM 등     |
| `net`  | 인터페이스 속도, Wake-on-LAN 설정    |
| `cpu`  | CPU 관리, 대기 시간 등                |

### 시나리오
- ex: 낮에는 CPU와 네트워크 사용량이 급증하는 웹 서버.
- monitor 플러그인이 CPU 부하(load)와 네트워크 패킷(net)을 감지.
- tuning 플러그인이 부하 증가 시 CPU 주파수 스케일링을 "performance" 모드로 변경.
- 부하가 줄어들면 자동으로 "powersave" 모드로 되돌림.
- 프로파일 자체를 바꾸는 게 아니라, monitor 플러그인이 내부 설정만 바꿈
  - ex. balanced 프로파일을 사용해도, 부하가 높을 때만 일시적으로 cpu_governor 값을 performance로 튜닝함.

## 프로파일 구조
- 기본 위치:
  - `/usr/lib/tuned` (시스템 제공. 수정하지 않는 것을 권장)
  - `/etc/tuned` (사용자 정의. 여기서 drop-in directory를 구성해서 관리하는 것을 권장)
- `/etc/tuned` 디렉터리 내용이 우선됨
- 구성 예시:

```ini
# /etc/tuned/virtual-guest/tuned.conf 예시
# include로 다른 프로파일의 설정값을 기본으로 포함할 수 있음
[main]
summary=Optimize for running inside a virtual guest
include=throughput-performance

[sysctl]
# 시스템 메모리에서 dirty page (디스크에 아직 기록되지 않은 수정된 페이지)가 차지할 수 있는 최대 비율을 설정
# 전체 RAM의 30%까지 dirty page가 누적될 수 있다는 의미입니다. 이를 초과하면 시스템은 디스크에 데이터를 강제로 기록
vm.dirty_ratio = 30
# 커널이 메모리 부족 시 얼마나 빨리 스왑 영역을 사용할지를 결정
# 낮은 값일수록 시스템은 RAM을 우선 사용하고, 스왑은 가능한 한 늦게 사용.
# 파일 캐시와 애플리케이션 메모리 사이의 균형을 유지합
# 스왑 영역: 메모리(RAM)가 부족할 때 일시적으로 디스크 공간을 메모리처럼 사용하는 공간
vm.swappiness = 30
```

## tuned 설치 및 시작
- 기본적으로 설치 및 활성화 되어있음
```bash
dnf install tuned
systemctl enable --now tuned
```

## 주요 튜닝 프로파일 요약 (RHEL 9 기준)

| 프로파일             | 용도 요약                                                 |
|----------------------|------------------------------------------------------------|
| `balanced`           | 절전과 성능 균형 (기본)                                    |
| `powersave`          | 최대 절전                                                  |
| `throughput-performance` | 최대 처리량 최적화, 	대용량 데이터 처리 서버 (예: 파일 서버, 미디어 스트리밍 서버)에서 디스크/네트워크 처리량을 최대화                                    |
| `latency-performance`    | 짧은 대기 시간 중심, 지연 시간이 중요한 금융 트랜잭션 서버, 실시간 게임 서버, 센서 처리 시스템 등에 적합                                     |
| `network-throughput`     | 네트워크 최대 처리량                                    |
| `network-latency`        | 네트워크 최소 대기 시간                                 |
| `desktop`            | 대화형 애플리케이션 응답성 향상, 사용자 인터랙션이 많은 데스크탑 환경 (예: GUI 사용, IDE 등).|
| `hpc-compute`        | 고성능 컴퓨팅 (HPC) 최적화, 과학적 시뮬레이션 등 연산 중심 작업                                |
| `virtual-guest`      | 가상 머신 게스트 최적화                                   |
| `virtual-host`       | 가상 머신 호스트 최적화                                   |


## tuned-adm 명령어 요약

| 명령어                            | 설명                                      |
|----------------------------------|-------------------------------------------|
| `tuned-adm list`                | 사용 가능한 프로파일 목록 확인             |
| `tuned-adm active`              | 현재 활성화된 프로파일 확인                |
| `tuned-adm profile <이름>`       | 프로파일 변경                             |
| `tuned-adm recommend`           | 시스템에 적합한 프로파일 추천             |
| `tuned-adm off`                 | 튜닝 비활성화 (프로파일 해제). 기본값으로 변경              |
| `tuned-adm profile_info [이름]` | 프로파일 요약 정보 확인                   |

- recommed 어떻게?
  - tuned는 /etc/tuned/recommend.d/ 경로에 있는 구성 파일이나 하드웨어 탐지를 통해 다음 정보를 활용합니다:
    - 시스템이 물리 서버인지 VM인지
    - 사용 중인 CPU 종류 (Intel/AMD, 코어 수 등)
    - 네트워크 장치나 디스크 장치의 종류
    - 특정 플랫폼(예: Azure, VMware, KVM, RHEL installer 환경 등)
    - 애플리케이션 자체를 직접 분석해서 판단하지는 않음