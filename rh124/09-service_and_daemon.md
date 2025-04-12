# 서비스 및 데몬 제어

- 서비스(Service)란?
  - 운영 체제에서 백그라운드에서 실행되는 프로그램을 의미하며, 주로 사용자에게 특정 기능을 제공하는 역할
  - 예를 들어, 웹 서버(Nginx, Apache), 데이터베이스 서버(MySQL, PostgreSQL), SSH 서버(OpenSSH) 등이 서비스에 해당
  - 서비스는 수동으로 실행할 수도 있고, 부팅 시 자동으로 실행되도록 설정할 수도 있음
- 데몬(Daemon)이란?
  - 데몬은 리눅스에서 백그라운드에서 실행되는 프로그램을 의미하며, 특정 작업을 지속적으로 수행하는 역할
  - 서비스와 비슷하지만, 일반적으로 시스템이나 네트워크 관련 작업을 처리하는 것이 많음
  - 대부분 `d`(daemon의 약자)로 끝나는 경우가 많음
    - 예: `sshd` (SSH Daemon), `httpd` (HTTP Daemon), `crond` (Cron Daemon), `systemd` (System Daemon) 등
- 서비스와 데몬의 관계
  - 대부분의 서비스는 특정 데몬을 실행하여 제공
  - 예를 들어, SSH 서비스(`ssh`)는 `sshd`라는 데몬을 실행하여 원격 접속 기능을 제공함
  - 서비스 관리 도구(`systemd`, `service`, `init.d` 등)를 통해 데몬을 관리함
- 리눅스의 init 시스템
  - 리눅스에서는 시스템의 초기화(init)와 서비스 관리를 담당하는 여러 가지 **init 시스템**이 존재함
  - 대표적으로 **`SysVinit`과 `systemd`**가 많이 사용됨
    - 참고, Mac은 `launchd`를 사용하고, systemctl 명령어 사용 불가
  - **SysVinit(System V init)**
    - UNIX의 **System V**에서 유래된 초기화 시스템
    - 오래된 방식으로, 전통적인 리눅스 배포판에서 사용됨 (예: RHEL 6, Debian 6, CentOS 6 등)
    - `/etc/init.d/` 스크립트를 이용하여 서비스(데몬)를 관리
    - 동작 방식
      - 커널이 부팅되면 **`/sbin/init`** 프로세스가 실행됨
      - `/etc/inittab` 파일을 읽고, 시스템의 **런레벨(Runlevel)** 을 결정함
      - 런레벨에 맞는 서비스(데몬)들이 실행됨 (`/etc/rc.d/rcN.d/` 디렉토리의 스크립트 실행)
      - 서비스가 실행되거나 중지될 때 `/etc/init.d/` 스크립트가 사용됨
    - 런레벨(Runlevel)
      - 시스템 상태를 정의하는 개념
      - 0: 시스템 종료
      - 1: 단일 사용자 모드(Single-user mode, root 만 가능)
      - 2: 다중 사용자 모드(네트워크 X)
      - 3: 다중 사용자 모드(CLI, 네트워크 O)
      - 4: 사용자 정의 모드
      - 5: GUI 모드
      - 6: 시스템 재부팅
    - 문제점
      - **직렬(Sequential) 실행** → 부팅 속도가 느림
        - 런레벨 기반으로 하나의 서비스가 실행 완료되면 다음 서비스가 실행됨 → 병렬 실행이 불가능함.
      - 의존성 관리 부족
        - 서비스 간 의존성을 자동으로 관리할 수 없음
      - 실시간 모니터링 부족
        - 서비스가 크래시(crash)해도 자동으로 재시작되지 않음
  - systemd
    - SysVinit의 문제점을 해결하기 위해 개발된 현대적인 **init 시스템**
    - 최신 리눅스 배포판에서 기본 사용됨 (예: RHEL 7+, CentOS 7+, Ubuntu 15.04+, Debian 8+)
    - 서비스 실행을 병렬적으로 수행하여 **부팅 속도 향상**
    - 서비스 의존성 자동 해결 가능
      - 특정 서비스가 실행되기 전에 필요한 서비스가 있으면 자동으로 실행됨
    - **서비스 상태 유지 및 복구**
      - 서비스가 크래시되면 자동으로 다시 실행 가능
    - 단일 바이너리 관리
      - `/lib/systemd/system/` 및 `/etc/systemd/system/` 디렉토리에 **서비스 유닛 파일**이 존재함
        - /lib/systemd/system/
          - 배포판에서 제공하는 기본 서비스 유닛 파일이 위치하는 디렉터리
          - 패키지 설치 시 자동으로 추가되는 서비스 유닛 파일이 저장됨
        - /etc/systemd/system/
          - 사용자가 직접 추가하거나 수정하는 서비스 유닛 파일을 저장하는 디렉터리
          - `/lib/systemd/system/`의 기본 유닛 파일을 오버라이드하거나 새로운 유닛 파일을 생성할 때 사용됨
          - `systemctl enable`을 실행하면 여기에 심볼릭 링크가 생성됨
    - **PID 1**(첫 번째 프로세스)로 실행됨
    - 런레벨 대신 `target` 사용
      - `runlevel` 개념 대신 `target` 개념을 도입했고, runlevel과 약간 다름
      - 유닛 그룹 단위를 관리하는 개념임
    - 주요 target
      - poweroff.target: 시스템 종료
      - rescue.target: 단일 사용자 모드
      - multi-user.target: CLI 기반 다중 사용자 모드
      - graphical.target: GUI 기반 모드
      - reboot.target: 시스템 재부팅

## 자동으로 시작된 시스템 프로세스 식별

- systemd 서비스 및 소켓 유닛에서 시작된 시스템 데몬과 네트워크 서비스를 살펴봄

### systemd 데몬 소개

- `systemd` 데몬은 일반적인 서비스 시작 및 서비스 관리를 포함하여 Linux 시작 프로세스를 관리함
- `systemd` 데몬은 부팅할 때 및 실행 중인 시스템에서 시스템 리소스, 서버 데몬, 기타 프로세스를 활성화함
- *데몬* 은 백그라운드에서 대기하거나 실행되어 다양한 작업을 수행하는 프로세스
- 일반적으로 데몬은 부팅 시 자동으로 시작되며, 종료되거나 수동으로 중지할 때까지 계속 실행됨
- `systemd` 에서 *서비스* 는 하나 이상의 데몬을 가리키는 경우가 많음
- 예외적으로 `oneshot`은 **systemd의 서비스 유형 중 하나**로, 특정 작업을 한 번 실행한 후 종료되는 서비스가 있음
  - `oneshot` 서비스는 **일반적인 데몬 서비스와 달리 실행 후 종료**
  - 따라서 실행이 끝난 후에도 데몬 프로세스는 남지 않음
  - 하지만 `RemainAfterExit=yes` 옵션을 설정하면, systemd는 서비스가 활성 상태(`active (exited)`)라고 간주할 수 있음
  - 예시
    - 시스템 상태를 변경하는 작업
    - 부팅 시 특정 설정을 적용
    - 서비스 간 의존성이 필요할 때
- Linux 제어 그룹을 사용하여 관련 프로세스를 한꺼번에 추적, 관리 가능함
  - 손쉬운 리소스 제한 가능
- 별도의 서비스를 요구하지 않고 필요할 때 데몬 시작
- 긴 타임아웃을 방지할 수 있는 자동 서비스 종속성 관리
  - 예를 들어 네트워크 종속 서비스는 네트워크를 사용할 수 있을 때까지 시작되지 않음

### 서비스 유닛 설명

- `systemd` *유닛* 은 시스템에서 관리 방법을 알고 있는 오브젝트를 정의하는 데 사용되는 추상적인 개념
- 유닛은 시스템 서비스, 수신 소켓, `systemd` init 시스템과 관련된 기타 오브젝트에 대한 정보를 캡슐화하는 ***유닛 파일*이라는 구성 파일로 표시됨**
  - 유닛 파일은 Windows에서 자주 쓰이는  INI 포맷으로 관리됨
  - 변경이 필요할 경우 `/lib/systemd/system/` 경로의 유닛 파일을 직접 수정하지 않고, `/etc/systemd/system/` 경로에 dropin 디렉터리를 만들어서 관리
    - 서비스 이름과 같되, `.d` 확장자를 갖는 디렉터리를 만들어서 conf 파일을 두고 사용함
    - conf 파일은 lexicographical(사전) notation 사용: {숫자}{숫자}-{설명}.conf
      - 낮은 숫자 파일이 먼저 실행되고, 이후 높은 숫자 파일이 설정을 오버라이드함
  - 유닛 파일 변경 후에는  `system daemon reload` 명령으로 재설정 적용 가능
- 유닛에는 **이름(name)**과 유형(type)이 존재
  - name: 유닛의 고유한 ID
  - type: 다른 유사한 유닛 유형과 그룹화할 수 있는 개념
    - 서비스 유닛
      - 확장자는 `.service` 이며 시스템 서비스, 웹 서버와 같이 자주 액세스하는 데몬을 시작할 수 있음
    - 소켓 유닛
      - 확장자는 `.socket` 이며 `systemd` 에서 모니터링해야 하는 프로세스 내 통신(IPC) 소켓을 나타냄
      - 클라이언트가 소켓에 연결하면 `systemd` 가 데몬을 시작하고 데몬에 연결을 전달
        - 서비스가 실행되지 않았더라도 소켓을 먼저 열어두고 대기할 수 있음
      - 소켓 유닛을 사용하여 부팅 시 서비스 시작을 연기하고 자주 사용되지 않는 서비스를 요청할 시 시작할 수 있음
        - 예를 들어, `sshd.socket`이 22번 포트를 열어두고 있다가 SSH 연결 요청이 오면 `sshd.service`를 실행하는 방식
    - 경로 유닛
      - 확장자가 `.path` 이며 특정 파일 시스템 변경이 발생할 때까지 서비스 활성화를 연기함
        - 파일 또는 디렉터리의 변경을 감지하여 특정 서비스(`.service`)를 실행
        - **inotify**(Linux 커널의 파일 시스템 감시 기능)를 사용하여 특정 파일 또는 디렉터리의 변경을 감지함
      - 출력 시스템과 같이 스풀 디렉터리를 사용하는 서비스에 경로 유닛을 사용할 수 있음
        - 스풀(Spool)?: Simultaneous Peripheral Operations On-Line
          - 입출력(I/O) 장치의 작업을 임시로 저장하여 순차적으로 처리하는 방식
          - Spool directory: 작업을 일시적으로 저장하고 나중에 처리하기 위해 사용하는 디렉터리
          - 프린터 시스템, 이메일 시스템, 배치 작업 시스템에서 주로 사용
- 유닛을 관리하려면 `systemctl` 명령을 사용함



### 서비스 유닛 표시

```shell
# 활성화 상태가 active인 서비스 유닛 조회
[root@host ~]# systemctl list-units --type=service
UNIT                  LOAD    ACTIVE  SUB      DESCRIPTION
atd.service           loaded  active  running  Job spooling tools
auditd.service        loaded  active  running  Security Auditing Service
chronyd.service       loaded  active  running  NTP client/server
crond.service         loaded  active  running  Command Scheduler
dbus.service          loaded  active  running  D-Bus System Message Bus
...output omitted...

# 활성화 상태 여부 관계없이 모든 서비스 유닛 조회
[root@host ~]# systemctl list-units --type=service --all
UNIT                          LOAD      ACTIVE   SUB     DESCRIPTION
  atd.service                 loaded    active   running Job spooling tools
  auditd.service              loaded    active   running Security Auditing ...
  auth-rpcgss-module.service  loaded    inactive dead    Kernel Module ...
  chronyd.service             loaded    active   running NTP client/server
  cpupower.service            loaded    inactive dead    Configure CPU power ...
  crond.service               loaded    active   running Command Scheduler
  dbus.service                loaded    active   running D-Bus System Message Bus
● display-manager.service     not-found inactive dead    display-manager.service
...output omitted...

# 로드되고(loaded) 활성화된(active) 유닛 조회
[root@host ~]# systemctl
UNIT                                 LOAD   ACTIVE SUB       DESCRIPTION
proc-sys-fs-binfmt_misc.automount    loaded active waiting   Arbitrary...
sys-devices-....device               loaded active plugged   Virtio network...
sys-subsystem-net-devices-ens3.deviceloaded active plugged   Virtio network...
...output omitted...
-.mount                              loaded active mounted   Root Mount
boot.mount                           loaded active mounted   /boot
...output omitted...
systemd-ask-password-plymouth.path   loaded active waiting   Forward Password...
systemd-ask-password-wall.path       loaded active waiting   Forward Password...
init.scope                           loaded active running   System and Servi...
session-1.scope                      loaded active running   Session 1 of...
atd.service                          loaded active running   Job spooling tools
auditd.service                       loaded active running   Security Auditing...
chronyd.service                      loaded active running   NTP client/server
crond.service                        loaded active running   Command Scheduler
...output omitted...

```

- UNIT: 서비스 유닛 이름
- LOAD: `systemd` 데몬이 유닛 구성을 정확하게 구문 분석하고 유닛을 메모리에 로드했는지 여부
- ACTIVE: 유닛의 개괄적인 활성화 상태, 유닛이 성공적으로 시작되었는지 여부를 나타냄
- SUB: 유닛의 구체적인 활성화 상태, 보다 자세한 유닛 유형, 상태 및 유닛 실행 방법 등 정보 표시
- DESCRIPTION: 유닛에 대한 간단한 설명

```shell

# 설치된 모든 유닛 파일의 상태를 조회
[root@host ~]# systemctl list-unit-files --type=service
UNIT FILE                         STATE       VENDOR PRESET
arp-ethers.service                disabled    disabled
atd.service                       enabled     enabled
auditd.service                    enabled     enabled
auth-rpcgss-module.service        static      -
autovt@.service                   alias       -
blk-availability.service          disabled    disabled
...output omitted...
```

- 주요 STATE 설명
  - enabled: 시스템 부팅 시 자동으로 실행
  - disabled: 서비스가 부팅 시 자동 실행되지 않도록 설정됨, **수동으로는 실행 가능**함
  - static: 단독으로 실행되지 않으며, 다른 서비스에 의해 실행됨, **단독 수동 실행 불가**
  - masked: 서비스가 실행되지 못하도록 완전히 차단됨, 시스템에서 특정 서비스가 실행되지 않도록 강제로 막을 때 사용

### 서비스 상태 보기

```shell
[root@host ~]# systemctl status sshd.service
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-03-14 05:38:12 EDT; 25min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 1114 (sshd)
      Tasks: 1 (limit: 35578)
     Memory: 5.2M
        CPU: 64ms
     CGroup: /system.slice/sshd.service
             └─1114 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Mar 14 05:38:12 workstation systemd[1]: Starting OpenSSH server daemon...
Mar 14 05:38:12 workstation sshd[1114]: Server listening on 0.0.0.0 port 22.
Mar 14 05:38:12 workstation sshd[1114]: Server listening on :: port 22.
Mar 14 05:38:12 workstation systemd[1]: Started OpenSSH server daemon.
...output omitted...
```

조회된 정보 내용

| 필드     | 설명                                                         |
| :------- | :----------------------------------------------------------- |
| Loaded   | 서비스 유닛이 메모리에 로드되었는지 여부입니다               |
| Active   | 서비스 유닛이 실행되고 있는지와 실행 중인 경우 실행된 기간입니다 |
| Docs     | 서비스에 대한 자세한 정보를 찾을 수 있는 위치입니다          |
| Main PID | 명령 이름을 포함하여 서비스의 기본 프로세스 ID입니다         |
| CGroup   | 관련 제어 그룹에 대한 자세한 정보입니다                      |

systemctl 출력의 서비스 status

| 키워드             | 설명                                                         |
| :----------------- | :----------------------------------------------------------- |
| Loaded             | 유닛 구성 파일이 처리되었습니다.                             |
| Active(Running)    | 프로세스가 계속되면서 서비스가 실행 중입니다.                |
| Active(Terminated) | 서비스가 일회성 구성을 성공적으로 완료했습니다.              |
| Active(Waiting)    | 서비스가 실행 중이지만 이벤트를 기다리고 있습니다.           |
| Inactive           | 서비스가 실행되고 있지 않습니다.                             |
| Enabled            | 서비스가 부팅할 때 시작됩니다.                               |
| Disabled           | 서비스가 부팅할 때 시작되도록 설정되어 있지 않습니다.        |
| Static             | 서비스를 수동으로는 활성화할 수 없지만, 활성화된 유닛에 의해 서비스를 자동으로 시작할 수 있습니다. |

### 서비스 상태 확인

```shell
# 서비스 유닛이 활성 상태(실행 중)인지 확인
[root@host ~]# systemctl is-active sshd.service
active
```



## 시스템 서비스 제어

- systemctl을 사용해 시스템 데몬 및 네트워크 서비스를 제어하는 방법 탐색

### 서비스 시작 및 중지

```shell
# 서비스가 실행 중인지 또는 중지되었는지 서비스 상태 조회
[root@host ~]# systemctl status sshd.service
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-03-23 11:58:18 EDT; 2min 56s ago
...output omitted...

# 서비스 시작
# 서비스 유형 없이 서비스 이름만 사용하여 systemctl start 명령을 실행하면 systemd 서비스에서 .service 파일을 찾음
[root@host ~]# systemctl start sshd

# 실행 중인 서비스를 중지
[root@host ~]# systemctl stop sshd.service
```



### 서비스 재시작 및 다시 로드

```shell
# 서비스 재시작: 서비스가 먼저 중지된 후 다시 시작
# 새 프로세스는 시작 중에 새 ID를 받게 되므로 프로세스 ID가 변경됨
[root@host ~]# systemctl restart sshd.service

# 서비스 reload: 일부 서비스는 재시작하지 않고도 구성 파일을 다시 로드(reload) 가능
# 서비스를 다시 로드하는 경우 다양한 서비스 프로세스와 연결된 프로세스 ID가 변경되지 않음
[root@host ~]# systemctl reload sshd.service

# 기능을 사용할 수 있는 경우 구성 변경 사항을 reload, 아니면 restart
[root@host ~]# systemctl reload-or-restart sshd.service
```



### 유닛 종속성 표시

```shell
# 출력 서비스를 중지하려면 종속성이 있는 3개 유닛을 모두 중지해야 함, 서비스를 비활성화하면 종속성도 비활성화
[root@host ~]# systemctl stop cups.service
Warning: Stopping cups, but it can still be activated by:
  cups.path
  cups.socket
  
# 서비스 유닛을 시작하는 종속성의 계층 구조 매핑을 표시
# 역방향 종속성(지정한 유닛에 종속된 유닛)을 표시하려면 명령에 --reverse 옵션을 사용
[root@host ~]# systemctl list-dependencies sshd.service
sshd.service
● ├─system.slice
● ├─sshd-keygen.target
● │ ├─sshd-keygen@ecdsa.service
● │ ├─sshd-keygen@ed25519.service
● │ └─sshd-keygen@rsa.service
● └─sysinit.target
...output omitted...
```



### 서비스 마스킹 및 마스킹 해제

```shell
# 서비스 마스킹 설정, 서비스가 시작되지 않도록 하는 /dev/null 파일에 대한 링크가 생성됨
[root@host ~]# systemctl mask sendmail.service
Created symlink /etc/systemd/system/sendmail.service → /dev/null.

# 서비스 상태 조회
[root@host ~]# systemctl list-unit-files --type=service
UNIT FILE                                   STATE
...output omitted...
sendmail.service                            masked
...output omitted...

# 마스킹된 서비스 유닛 시작 시도 -> 실패
[root@host ~]# systemctl start sendmail.service
Failed to start sendmail.service: Unit sendmail.service is masked.

# 서비스 유닛의 마스킹 해제, 생성됐던 심볼릭 링크 파일 제거
[root@host ~]# systemctl unmask sendmail
Removed /etc/systemd/system/sendmail.service.
```

### 부팅할 때 시작 또는 중지되도록 서비스 Enable

```shell
# 마스킹과 마찬가지로 systemd 구성 디렉터리에 링크를 생성하는 방식으로 enable 설정
# /etc/systemd/system/TARGETNAME.target.wants 디렉터리에 심볼릭 링크를 생성
# 명령을 수행한 쉘 세션에서 서비스 자동 시작되지는 않음
[root@root ~]# systemctl enable sshd.service
Created symlink /etc/systemd/system/multi-user.target.wants/sshd.service → /usr/lib/systemd/system/sshd.service.

# 서비스 enable 설정 및 바로 시작
[root@root ~]# systemctl enable --now sshd.service
Created symlink /etc/systemd/system/multi-user.target.wants/sshd.service → /usr/lib/systemd/system/sshd.service.

# 서비스 enable 여부 조회
[root@host ~]# systemctl is-enabled sshd.service
enabled

# 서비스 disable 및 바로 stop
[root@host ~]# systemctl disable --now sshd.service
Removed /etc/systemd/system/multi-user.target.wants/sshd.service.
```



### systemctl 명령 요약

| 명령                   | 작업                                                         |
| :--------------------- | :----------------------------------------------------------- |
| systemctl status UNIT  | 유닛 상태에 대한 세부 정보를 봄                              |
| systemctl stop UNIT    | 실행 중인 시스템에서 서비스를 중지                           |
| systemctl start UNIT   | 실행 중인 시스템에서 서비스를 시작                           |
| systemctl restart UNIT | 실행 중인 시스템에서 서비스를 다시 시작                      |
| systemctl reload UNIT  | 실행 중인 서비스의 구성 파일을 다시 로드                     |
| systemctl mask UNIT    | 수동으로 및 부팅할 때 시작되지 않도록 서비스를 mask          |
| systemctl unmask UNIT  | 마스킹된 서비스를 unmask                                     |
| systemctl enable UNIT  | 부팅할 때 서비스가 시작되도록 구성함, `--now` 옵션을 사용하여 서비스를 바로 시작할 수도 있음 |
| systemctl disable UNIT | 부팅할 때 서비스가 시작되지 않도록 disable, `--now` 옵션을 사용하여 서비스를 바로 중지할 수도 있음 |

# 기타

```shell
# default target 조회
$ systemctl get-default
```

- systemctl set-default graphical.target
  - 시스템의 기본 타겟 설정
  - 부팅 시 실행될 기본 타겟을 변경할 수 있음