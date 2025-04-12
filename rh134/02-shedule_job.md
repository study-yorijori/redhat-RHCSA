# 사용자 작업 예약
- 한 번 실행될 작업을 예약하거나, 반복적으로 수행될 작업을 예약합니다

## 지연된 사용자 작업
- 특정 시간에 하나 이상의 명령을 실행해야 하는 경우
- 기본으로 설치된 at 명령어 사용하여 atd에 작업 추가 가능
- atd 데몬은 a 부터 z까지 26개 대기열을 제공. 알파벳 순서로 뒤쪽의 대기열에 있는 작업은 시스템 우선순위가 낮음
>      -q queue
             Use the specified queue.  A queue designation consists of a single letter; valid queue
             designations range from a to z and A to Z.  The _DEFAULT_AT_QUEUE queue (a) is the
             default for at and the _DEFAULT_BATCH_QUEUE queue (b) is the default for batch.  Queues
             with higher letters run with increased niceness.  If a job is submitted to a queue
             designated with an uppercase letter, it is treated as if it had been submitted to batch
             at that time.  If atq is given a specific queue, it will only show jobs pending in that
             queue.
- 옵션은 지정된 큐에서 작업을 실행하도록 함. 큐의 지정은 하나의 문자로 이루어짐. 유효한 큐 문자는 a에서 z, A에서 Z까지임
- 높은 문자일수록 "niceness" 값이 증가함
  - Niceness 값?
  - niceness는 작업의 우선순위를 조정하는 값으로, 작업이 시스템 리소스를 얼마나 "친절하게" 사용할지를 결정함. niceness 값은 -20에서 19까지의 범위를 가짐.
  - niceness 값이 낮을수록 (즉, -20에 가까울수록) 우선순위가 높고, CPU 자원을 더 많이 차지할 수 있음.
  - niceness 값이 높을수록 (즉, 19에 가까울수록) 우선순위가 낮고, 다른 중요한 작업들이 CPU 자원을 더 많이 사용하게 허용됨.
  - 따라서, 큐 문자가 뒤로 갈수록(즉, 문자가 'a'에서 'z'로 갈수록) niceness 값이 증가하며, 이는 해당 작업의 우선순위가 낮아지고 다른 작업들이 우선적으로 실행될 수 있도록 만듦. 예를 들어, a 큐의 작업이 가장 높은 우선순위를 가지며, z 큐의 작업은 가장 낮은 우선순위를 가짐.
  - Q. at 명령어로 모두 같은 시간에 수행되는 작업 3개를 등록할 때, 작업 job1, job2, job3가 다음과 같이 큐에 등록됨:
  a 큐에 job1과 job2가 등록됨.
  b 큐에 job3가 등록됨.

  이때, 각 작업이 어떤 순서대로 수행되는지?
  - A. job1,2,3 순서로 실행

### 작업 예약
- `at TIMESPEC` 명령을 통해 예약
- STDIN으로 입력된 내용을 읽음
- `at now +5min < myscript`: 이렇게 입력도 가능
- 시간 예시
```
now +5min: 5분 뒤
teatime tomorrow (teatime은 16:00): 내일 16시
noon +4 days: 4일 후 정오
5pm august 3 2021: 21년 8월 3일 오후 5시
```

### 작업 조회
```bash
[user@host ~]$ atq
28  Mon May 16 05:13:00 2022 a user
29  Tue May 17 16:00:00 2022 h user
30  Wed May 18 12:00:00 2022 a user
```
- 28은 고유한 작업 번호이며, Mon May 16 05:13:00 2022는 예약된 작업의 실행 날짜와 시간을 나타냄. a는 작업이 기본 대기열 a에 예약됨을 의미하며, **user는 작업의 소유자(및 해당 작업을 실행하는 사용자)**임

### 작업 제거
- `atrm JOBNUMBER`

## 반복 실행 사용자 작업 예약
- 사용자의 crontab 파일을 사용하여 반복 스케줄에 따라 실행되도록 명령을 예약
- 기본적으로 활성화되고 시작되는 crond 데몬을 사용
- 각 사용자에게는 crontab -e 명령으로 편집하는 개인 파일이 있음

### 반복 실행 사용자 작업 예약
- `crontab` 명령을 사용하여 예약된 작업을 관리
- `crontab -l`: 현재 사용자의 작업을 나열합니다.
- `crontab -r`: 현재 사용자의 모든 작업을 제거합니다.
- `crontab -e	`: 현재 사용자의 작업을 편집합니다.
- `crontab $filename`: 모든 작업을 제거하고 filename에서 읽은 작업으로 바꿉니다. 이 명령은 파일이 지정되지 않은 경우 stdin 입력을 사용합니다.
- 권한 있는 사용자는 crontab 명령 -u 옵션을 사용하여 다른 사용자의 작업을 관리할 수 있음

### 사용자 작업 형식 설명
crontab 파일의 필드는 다음 순서로 나타납니다.
`분 시간 일 월 요일 명령`
- `15 12 11 * Fri command`:  매월 11일의 12시 15분에 금요일에 해당하는 경우에만 command를 실행하는 크론 작업
- * 문자를 사용하여 필드의 가능한 모든 인스턴스에서 실행합니다.
- 분 또는 시간, 날짜 또는 요일을 지정하는 숫자입니다. 요일의 경우 0은 일요일, 1은 월요일, 2는 화요일에 해당합니다. 7도 일요일에 해당합니다.
- x 및 y 값이 포함되는 범위의 경우 x-y를 사용합니다.
- */x는 x만큼의 간격을 나타냅니다. 예를 들어, Minutes 열에 */7을 입력하면 7분마다 작업이 실행됩니다.

### 예시
- `0 9 3 2 * /usr/local/bin/yearly_backup`: 매년 2월 3일 오전 9시 정각에 /usr/local/bin/yearly_backup 명령을 실행
- `*/5 9-16 * Jul 5 echo "Chime"`:  7월 매주 금요일 오전 9시부터 오후 4시까지 5분마다 이 작업의 소유자에게 Chime 이라는 단어가 포함된 이메일을 전송
  - *해당 명령의 출력 결과(표준 출력 및 표준 에러)**는 자동으로 메일로 전송됨. 이때 메일이 전송되는 주소는 기본적으로 작업을 등록한 사용자 계정의 로컬 메일함
  - 작업을 등록한 사용자가 예를 들어 user1이면, 메일은 다음과 같은 로컬 주소로 전송됨
  `user1@myserver.local`
  - 로컬 메일함은 일반적으로 `mail` 명령어로 확인 가능함:
  - crontab 파일 맨 위에 MAILTO 환경변수를 설정하면 원하는 이메일 주소로 받을 수 있음

## 반복 실행 시스템 작업 예약
### 반복 실행 시스템 작업
- 사용자 계정이 아닌 시스템 계정에서 반복 작업
- crontab 명령 대신 시스템 전체 crontab 파일을 사용하여 이러한 작업을 예약
- crontab 파일의 작업 항목은 사용자의 crontab 항목과 유사. 시스템 전체 crontab 파일에는 명령을 실행하는 사용자를 지정하는 명령 필드 앞에 추가 필드가 있음
- /etc/crontab 
~~~
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
~~~
- `/etc/cron.d/` 디렉터리와 `/etc/crontab` 파일은 주기적으로 실행되는 시스템 작업을 정의하는 곳임
- 시스템에서 반복적으로 실행되는 작업을 예약하려면 사용자 지정 crontab 파일을 `/etc/cron.d/`에 생성해야 함
  - 패키지 업데이트 시 `/etc/crontab` 파일이 덮어써지는 걸 방지하기 위해, 사용자 지정 작업은 `/etc/cron.d/`에 별도로 저장
- 또한, crontab 시스템은 특정 주기(매시간, 매일, 매주, 매월)에 따라 실행되는 스크립트를 저장하는 디렉터리도 제공
- 각각 `/etc/cron.hourly/`, `/etc/cron.daily/`, `/etc/cron.weekly/`, `/etc/cron.monthly/` 디렉터리
- 이 디렉터리들에는 crontab 문법이 아닌, **실행 가능한 쉘 스크립트 파일이 들어 있음**
- 따라서 해당 시간 주기에 따라 자동으로 스크립트가 실행되도록 구성되어 있음.

## Anacron을 사용하여 정기 명령 실행
- 일반적인 cron이 가지는 한계를 보완해주는 도구
  - Anacron은 시스템이 꺼져 있던 동안 실행되지 못한 주기적 작업들을 다시 실행시켜주는 도구
- anacron 명령은 run-parts 스크립트를 사용하여 `/etc/anacrontab` 구성 파일의 작업을 매일, 매주, 매월 실행
- ex. `1   5   cron.daily   run-parts /etc/cron.daily`
  - 매일 부팅후 5분 후에 `/etc/cron.daily` 내부 스크립트 모두 실행
- `/etc/anacrontab` 파일에는 NAME=value 구문을 사용하는 환경 변수 선언도 포함되어 있음
  - ex. `START_HOURS_RANGE` 변수는 작업이 실행되는 시간 간격을 지정

## systemd 타이머
- systemd 타이머 유닛은 특정 시간에 다른 유닛(보통 서비스 유닛)을 자동 실행시키는 역할
- 름이 같은 서비스 유닛을 타이머가 활성화시킴.(예: myjob.timer → myjob.service를 실행함.)
- `systemctl list-timers` 로 타이머 검색 가능 
### sysstat-collect.timer
- sysstat-collect.timer는 10분마다 시스템 통계를 수집하는 타이머 유닛
~~~
[Unit]
Description=Run system activity accounting tool every 10 minutes

[Timer]
OnCalendar=*:00/10

[Install]
WantedBy=sysstat.service
~~~
- OnCalendar=*:00/10 → 매시 0분부터 10분 간격으로 실행됨, 즉 00, 10, 20, 30...에 작동함.
- sysstat 패키지란?
  - sysstat은 CPU, 메모리, 디스크 I/O, 네트워크 등 다양한 시스템 자원의 상태를 수집하고 보고하는 도구 모음

## 임시 파일 관리
- 대부분의 애플리케이션과 서비스는 임시 파일과 디렉터리를 사용
- 디스크에 계속 쌓이지 않도록 임시 파일 제거할 필요가 있음
- Red Hat Enterprise Linux에는 systemd-tmpfiles 툴이 포함되어 임시 디렉터리와 파일을 관리하는 체계적이고 구성 가능한 방법을 제공 
- 작동 방식
  -  시스템이 부팅될 때 자동으로 실행되는 서비스 중 하나가 **systemd-tmpfiles-setup.service**
  - 이 서비스는 내부적으로 다음 명령을 실행
    - `systemd-tmpfiles --create --remove`
    -  특정 설정 파일들에 따라 파일/디렉터리를 만들거나 지움
- 읽는 설정 파일 위치
  - `systemd-tmpfiles`는 아래 3개 경로에서 `.conf` 파일을 읽음:
    - `/usr/lib/tmpfiles.d/` → 시스템 기본 설정 (패키지 제공)
    - `/run/tmpfiles.d/` → 런타임 설정
    - `/etc/tmpfiles.d/` → 사용자가 직접 추가하거나 커스터마이징한 설정

### Systemd 타이머를 사용하여 임시 파일 정리
- 장기 실행 시스템이 부실 데이터로 디스크를 채우지 않도록 하기 위해 systemd-tmpfiles-clean.timer 라는 systemd 타이머 유닛이 일정 간격으로 systemd-tmpfiles-clean.service 유닛을 트리거하며, 이 유닛은 다시 systemd-tmpfiles --clean 명령을 실행
- systemd-tmpfiles-clean.timer 장치 구성 파일의 내용을
~~~bash
[user@host ~]$ systemctl cat systemd-tmpfiles-clean.timer
# /usr/lib/systemd/system/systemd-tmpfiles-clean.timer
#  SPDX-License-Identifier: LGPL-2.1-or-later
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Daily Cleanup of Temporary Directories
Documentation=man:tmpfiles.d(5) man:systemd-tmpfiles(8)
ConditionPathExists=!/etc/initrd-release

[Timer]
OnBootSec=15min
OnUnitActiveSec=1d
~~~
- OnBootSec=15min 매개 변수는 systemd-tmpfiles-clean.service 유닛이 시스템이 부팅되고 15분 후에 트리거됨을 나타냄
- OnUnitActiveSec=1d 매개 변수는 systemd-tmpfiles-clean.service 유닛에 대한 추가 트리거가 서비스 유닛이 마지막으로 활성화되고 24시간 후에 발생

###  설정 파일 
- type: 파일을 생성할지, 삭제할지 등 항목 유형
  - d: 디렉터리 생성 및 오래된 파일 정리	
  - D: d외 동일하지만 디렉터리 안 파일 전부 삭제
  - q: d와 비슷하되, 재부팅 후 삭제 처럼 특별 처리하는 경우
  - r: 파일, 디렉토리 삭제(only 삭제)
  - v: 재귀적으로 모든 파일 디렉토리 삭제
- 디렉토리 정보ㅡ 해당 파일의 권한, 소유자, 경로 등을 어떻게 설정할지 등 내용 포함
- `d /var/log/myapp 0755 root root -`
  - `/var/log/myapp` 디렉터리를 root 소유로, 권한 755로 생성함.
  - 마지막에 나오는 -는 보존 시간을 지정
- `d /run/momentary 0700 root root 30s`
  - `/run/momentary` 디렉터리를 root 소유로, 권한 700로 생성함.
  - 이 디렉터리에서 30초 동안 사용되지 않은 파일을 제거
- `D /home/student 0700 student student 1d`
  - `/home/student` 디렉터리가 없는 경우 해당 디렉터리를 만듦
  - 해당 디렉터리가 있는 경우 모든 콘텐츠를 제거합니다
  - 시스템에서 systemd-tmpfiles --clean 명령을 실행하면 디렉터리에서 1일 초과 액세스, 변경 또는 수정되지 않은 모든 파일이 제거
