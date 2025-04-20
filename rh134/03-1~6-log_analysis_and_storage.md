# 로그 분석 및 저장
## 시스템 로그 아키텍처 설명
### 시스템 로깅
- 운영 체제 커널 및 기타 프로세스는 시스템이 실행 중일 때 발생하는 이벤트의 로그를 기록
- 로그는 시스템 검사 및 문제 해결에 사용할 수 있음
- less 및 tail 명령과 같은 텍스트 유틸리티를 사용하여 이러한 로그를 검사 가능
- Red Hat Enterprise Linux는 syslog 프로토콜을 기반으로 하는 표준 로깅 시스템을 사용하여 시스템 메시지를 기록
- systemd-journald 및 rsyslog 서비스는 Red Hat Enterprise Linux 9에서 syslog 메시지를 처리

###	systemd-journald
- systemd의 일부로, 시스템 로그를 수집하는 기본 서비스
- 바이너리 (journal format) 형태로 구조화하여 저장
  - 빠른 검색 지원
  - 사용자별 접근 제어 가능
  - 로그 조작 방지
- 기본적으로 메모리(휘발성)에 저장되지만, 설정에 따라 디스크에 영구 저장도 가능
- journalctl 활용하여 로그 조회
- 수집 데이터
  - 커널 메시지 (/dev/kmsg)
  - 서비스 stdout/stderr
  - syslog API (/dev/log)

### rsyslog
- 전통적인 syslog 데몬으로, 텍스트 기반 로그를 파일로 저장합니다.
- 텍스트 기반 저장
  - 느린 조회
  - 파일 권한 기반 제어
  - 조작이 쉬움
- cat, tail, vi 등 텍스트 도구를 사용해 조회
- rsyslog 서비스는 재부팅 후에도 지속되는 `/var/log` 디렉터리의 로그 파일에 `syslog` 메시지를 정렬하고 씀 또한 이 서비스는 각 메시지를 전송한 프로그램 유형 및 각 syslog 메시지의 우선 순위에 따라 로그 메시지를 특정 로그 파일에 정렬합니다
- 수집 데이터
  - syslog API (/dev/log)
  - network syslog (UDP/TCP)
  - 파일 입력

### syslog API?
- "로그를 남기는 표준적인 방식"을 OS에서 제공하는 API
- POSIX 표준에서는 다음 함수들로 구성됨
```
openlog()      // 로그 태그 설정
syslog()       // 로그 남기기
closelog()     // (필요하면) 닫기
```
- Python, Go, Node 등 많은 언어에서 래퍼를 제공
- 내부적으로는 /dev/log라는 Unix Domain Socket으로 메시지를 보냄

### 사용
- 최신 리눅스 배포판에서는 journald와 rsyslog가 함께 동작하는 경우가 많음
- journald가 수집한 로그를 rsyslog가 읽어가서 외부로 전송하는 형태로 둘을 연동해서 사용하기도 함
- FastAPI
~~~
FastAPI 애플리케이션
    ↓
SysLogHandler → /dev/log (Unix socket)
    ↓
systemd-journald (보통은 이게 listen) or rsyslog
    ↓
저장:
- journald: /run/log/journal/...
- rsyslog: /var/log/syslog, /var/log/messages 등
~~~
- or 단순 stdout/stderr → systemd-journald가 자동으로 처리
- rsyslog와 journald가 같이 있는 경우 journald에서 rsyslog로 ForwardToSyslog=yes 설정하면 로그 중복 없이 연동 가능

## rsyslog 시스템 로그 파일
| 로그 파일              | 저장되는 메시지 유형                                                                 |
|------------------------|--------------------------------------------------------------------------------------|
| /var/log/messages      | 대부분의 syslog 메시지가 여기에 기록됩니다. 단, 인증 및 이메일 처리와 예약된 작업 실행에 대한 메시지, 순수하게 디버깅에만 관련된 메시지는 예외입니다. |
| /var/log/secure        | 보안 및 인증 이벤트에 대한 syslog 메시지                                            |
| /var/log/maillog       | 메일 서버에 대한 syslog 메시지                                                     |
| /var/log/cron          | 예약된 작업 실행에 대한 syslog 메시지                                              |
| /var/log/boot.log      | 시스템 시작에 대한 syslog 콘솔 메시지                                              |

## Syslog 파일 검토
- syslog 프로토콜을 사용하여 이벤트를 시스템 로그에 기록
- 각 로그 메시지는 facility(메시지를 생성하는 하위 시스템) 및 우선순위(메시지 심각도)에 따라 분류
- facility
| 코드  | 기능         | 기능 설명                        |
|-------|--------------|----------------------------------|
| 0     | kern         | 커널 메시지                      |
| 1     | user         | 사용자 수준 메시지               |
| 2     | mail         | 메일 시스템 메시지               |
| 3     | daemon       | 시스템 데몬 메시지               |
| 4     | auth         | 인증 및 보안 메시지              |
| 5     | syslog       | 내부 syslog 메시지               |
| 6     | lpr          | 프린터 메시지                    |
| 7     | news         | 네트워크 뉴스 메시지             |
| 8     | uucp         | UUCP 프로토콜 메시지             |
| 9     | cron         | 시계 데몬 메시지                 |
| 10    | authpriv     | 비 시스템 권한 부여 메시지       |
| 11    | ftp          | FTP 프로토콜 메시지              |
| 16~23 | local0~local7| 사용자 지정 로컬 메시지          |

- priority
| 코드 | 우선 순위     | 우선 순위 설명             |
|------|--------------|----------------------------|
| 0    | emerg        | 시스템을 사용할 수 없음     |
| 1    | alert        | 즉각적인 조치 필요          |
| 2    | crit         | 심각한 상태                |
| 3    | err          | 심각하지 않은 오류 상태     |
| 4    | warning        | 경고 상태                  |
| 5    | notice       | 정상이지만 중요한 이벤트   |
| 6    | info         | 정보 이벤트                |
| 7    | debug        | 디버깅 수준의 메시지        |

- /etc/rsyslog.conf 파일과 /etc/rsyslog.d 디렉터리의 파일( .conf 확장자)에서 이 기능과 우선 순위를 구성
`authpriv.*                  /var/log/secure`

### syslog 항목 분석
- 로그 메시지는 로그 파일 시작 부분에 가장 오래된 메시지가 있고 끝에 최신 메시지가 있는 상태로
- rsyslog 서비스는 표준 형식을 사용하여 로그 파일에 항목을 기록
~~~
Mar 20 20:11:48 localhost sshd[1433]: Failed password for student from 172.25.0.10 port 59344 ssh2
~~~
- Mar 20 20:11:48 : 로그 항목의 타임스탬프를 기록합니다.
- localhost : 로그 메시지를 전송하는 호스트입니다.
- sshd[1433] : 로그 메시지를 전송한 프로그램 또는 프로세스 이름 및 PID 번호입니다.
- Failed password for …​: 전송된 메시지입니다.

### 수동으로 메세지 전송
`logger -p local7.notice "Log entry created on host"`

## 시스템 저널 항목 검토
### 시스템 저널에서 이벤트 찾기
- systemd-journald 서비스는 저널이라는 인덱싱된 구조적 바이너리 파일에 로깅 데이터를 저장
- 저널에서 로그 메시지를 검색하려면 journalctl 명령을 사용
- journalctl 명령을 사용하여 저널의 모든 메시지를 보거나 옵션과 기준에 따라 특정 이벤트를 검색할 수 있음
- `-n`: 갯수 제한
```bash
[root@host ~]# journalctl -n 5
Mar 15 04:42:17 host.lab.example.com systemd[1]: Started Hostname Service.
Mar 15 04:42:47 host.lab.example.com systemd[1]: systemd-hostnamed.service: Deactivated successfully.
Mar 15 04:47:33 host.lab.example.com systemd[2127]: Created slice User Background Tasks Slice.
Mar 15 04:47:33 host.lab.example.com systemd[2127]: Starting Cleanup of User's Temporary Files and Directories...
Mar 15 04:47:33 host.lab.example.com systemd[2127]: Finished Cleanup of User's Temporary Files and Directories.
```
- `-f`: 실시간 조회
```bash
[root@host ~]# journalctl -f
Mar 15 04:47:33 host.lab.example.com systemd[2127]: Finished Cleanup of User's Temporary Files and Directories.
Mar 15 05:01:01 host.lab.example.com CROND[2197]: (root) CMD (run-parts /etc/cron.hourly)
Mar 15 05:01:01 host.lab.example.com run-parts[2200]: (/etc/cron.hourly) starting 0anacron
Mar 15 05:01:01 host.lab.example.com anacron[2208]: Anacron started on 2022-03-15
Mar 15 05:01:01 host.lab.example.com anacron[2208]: Will run job `cron.daily' in 29 min.
Mar 15 05:01:01 host.lab.example.com anacron[2208]: Will run job `cron.weekly' in 49 min.
Mar 15 05:01:01 host.lab.example.com anacron[2208]: Will run job `cron.monthly' in 69 min.
Mar 15 05:01:01 host.lab.example.com anacron[2208]: Jobs will be executed sequentially
Mar 15 05:01:01 host.lab.example.com run-parts[2210]: (/etc/cron.hourly) finished 0anacron
Mar 15 05:01:01 host.lab.example.com CROND[2196]: (root) CMDEND (run-parts /etc/cron.hourly)
​^C
[root@host ~]#
```
- 우선순위 필터링
  - debug, info, notice, warning, err, crit, alert, emerg 우선 순위 수준을 오름차순
```bash
[root@host ~]# journalctl -p err
Mar 15 04:22:00 host.lab.example.com pipewire-pulse[1640]: pw.conf: execvp error 'pactl': No such file or direct
Mar 15 04:22:17 host.lab.example.com kernel: Detected CPU family 6 model 13 stepping 3
Mar 15 04:22:17 host.lab.example.com kernel: Warning: Intel Processor - this hardware has not undergone testing by Red Hat and might not be certif>
Mar 15 04:22:20 host.lab.example.com smartd[669]: DEVICESCAN failed: glob(3) aborted matching pattern /dev/discs/disc*
Mar 15 04:22:20 host.lab.example.com smartd[669]: In the system's table of devices NO devices found to scan
```
- 유닛 필터링
```bash
[root@host ~]# journalctl -u sshd.service
May 15 04:30:18 host.lab.example.com systemd[1]: Starting OpenSSH server daemon...
May 15 04:30:18 host.lab.example.com sshd[1142]: Server listening on 0.0.0.0 port 22.
May 15 04:30:18 host.lab.example.com sshd[1142]: Server listening on :: port 22.
```
- 복합 옵션
~~~bash
[root@host ~]# journalctl _SYSTEMD_UNIT=sshd.service _PID=2110
Mar 15 04:42:16 host.lab.example.com sshd[2110]: Accepted publickey for root from 172.25.250.254 port 46224 ssh2: RSA SHA256:1UGybTe52L2jzEJa1HLVKn9QUCKrTv3ZzxnMJol1Fro
Mar 15 04:42:16 host.lab.example.com sshd[2110]: pam_unix(sshd:session): session opened for user root(uid=0) by (uid=0)
~~~