# 시스템 저널 보존

- 서버가 재부팅될 때 이벤트 레코드를 보존하도록 시스템 저널을 구성함



## 시스템 저널 스토리지

- 기본적으로 RHEL9에서는 /run/log 디렉터리에 시스템 저널을 저장하며, 재부팅 후 시스템에서 시스템 저널을 제거함
- 재부팅 후에도 저널이 유지되도록 `/etc/systemd/journald.conf` 파일에서 `systemd-journald` 서비스의 구성 설정을 변경할 수 있음
- `/etc/systemd/jounrnald.conf` 파일의 Storage 매개 변수가 시스템 저널을 일시적으로 저장할지 또는 재부팅 후에도 저장할지 정의함
  - 매개 변수를 아래 값 중 하나로 설정 가능
    - `persistent`: 재부팅 후에도 지속되는 `/var/log/journal` 디렉터리에 저널을 저장함, 디렉터리가 없으면 systemd-journald 서비스에서 생성함
    - `volatile`: 일시적인 `/run/log/journal` 디렉터리에 저널을 저장함 /run 파일 시스템은 일시적이고 런타임 메모리에만 존재해 시스템 저널을 포함해 이 파일 시스템 데이터는 재부팅 후 유지되지 않음
    - `auto`: `/var/log/journal` 디렉터리가 있으면 systemd-journald 서비스는 persistent storage를 사용하고, 없으면 일시적 스토리지 사용함, Storage 매개 변수를 정하지 않으면 이 동작이 기본값임
    - `none`: 스토리지를 사용하지 않음. 시스템에서 모든 로그를 삭제하지만 계속해서 로그를 전달할 수 있음



## 영구 시스템 저널 구성

재부팅 후에도 시스템 저널이 영구적으로 보존되도록 `systemd-journald` 서비스를 구성하기 위해 다음을 수행함

```shell
[root@host ~]# mkdir /var/log/journal

# Storage 매개 변수 값을 persistent로 수정
[root@host ~]# vi /etc/systemd/jounald.conf

[Journal]
Storage=persistent
...output omitted...

# 설정 변경을 반영하기 위한 서비스 재시작
[root@host ~]# systemctl restart systemd-journald

# 서비스가 생성한 디렉터리 및 파일
[root@host ~]# ls /var/log/journal
4ec03abd2f7b40118b1b357f479b3112
[root@host ~]# ls /var/log/journal/4ec03abd2f7b40118b1b357f479b3112
system.journal  user-1000.journal

# 시스템 부팅 이벤트 표시
[root@host ~]# journalctl --list-boots
  -6 27de... Wed 2022-04-13 20:04:32 EDT—Wed 2022-04-13 21:09:36 EDT
  -5 6a18... Tue 2022-04-26 08:32:22 EDT—Thu 2022-04-28 16:02:33 EDT
  -4 e2d7... Thu 2022-04-28 16:02:46 EDT—Fri 2022-05-06 20:59:29 EDT
  -3 45c3... Sat 2022-05-07 11:19:47 EDT—Sat 2022-05-07 11:53:32 EDT
  -2 dfae... Sat 2022-05-07 13:11:13 EDT—Sat 2022-05-07 13:27:26 EDT
  -1 e754... Sat 2022-05-07 13:58:08 EDT—Sat 2022-05-07 14:10:53 EDT
   0 ee2c... Mon 2022-05-09 09:56:45 EDT—Mon 2022-05-09 12:57:21 EDT

# 출력을 특정 시스템 부팅으로 제한하려면 -b 옵션 사용
[root@host ~]# journalctl -b 1
...output omitted...

# 현재 시스템 부팅의 항목만 검색
[root@host ~]# journalctl -b
...output omitted...

```



<img width="608" alt="Image" src="https://github.com/user-attachments/assets/7033f73a-7a19-4f18-94f1-065866e207d0" />



# 정확한 시간 유지 관리

- NTP(Network Time Protocol)을 사용해 정확한 시간 동기화를 유지하고 시스템 저널 및 로그에서 기록하는 이벤트의 타임스탬프가 정확하도록 시간대를 구성함

## 로컬 시계 및 시간대 관리

- 여러 시스템의 로그 파일을 분석하기 위해 시스템 시간 동기화가 중요함
- 시스템은 네트워크 타임 프로토콜을 사용해 인터넷을 통해 올바른 시간 정보를 제공하고 얻음
- 시스템은 NTP 풀 프로젝트와 같은 공용 NTP 서비스로부터 정확한 시간 정보를 얻을 수 있음
- `timedatectl` 명령은 시스템의 현재 시간, 시간대 및 NTP 동기화 설정을 포함해 현재 시간과 관련된 시스템 설정의 개요를 표시함

```shell
# 시스템의 현재 시간, 시간대 및 NTP 동기화 설정 표시
[user@host ~]$ timedatectl
               Local time: Wed 2022-03-16 05:53:05 EDT
           Universal time: Wed 2022-03-16 09:53:05 UTC
                 RTC time: Wed 2022-03-16 09:53:05
                Time zone: America/New_York (EDT, -0400)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
          
# 시간대 데이터베이스 표시
[user@host ~]$ timedatectl list-timezones
Africa/Abidjan
Africa/Accra
Africa/Addis_Ababa
Africa/Algiers
Africa/Asmara
Africa/Bamako
...output omitted... 
```

- IANA(Internet Assigned Numbers Authority)는 공용 시간대 데이터베이스를 제공하고, `timedatectl` 명령은 해당 데이터베이스를 기반으로 하여 시간대 이름을 지정함
- `tzselect` 명령을 사용하여 올바른 시간대 이름을 식별하고, 시스템의 시간대 설정은 변경하지 않음

```shell
[root@host ~]# timedatectl set-timezone America/Phoenix
[root@host ~]# timedatectl
               Local time: Wed 2022-03-16 03:05:55 MST
           Universal time: Wed 2022-03-16 10:05:55 UTC
                 RTC time: Wed 2022-03-16 10:05:55
                Time zone: America/Phoenix (MST, -0700)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
          
# 자동 시간 조정을 위한 NTP 동기화를 비활성화
[root@host ~]# timedatectl set-ntp false
```

<img width="604" alt="Image" src="https://github.com/user-attachments/assets/8d631d98-4941-4f4d-8918-63f504738efe" />

## chronyd 서비스 구성 및 모니터링

- `chronyd` 서비스는 구성된 NTP 서버와 동기화하여 일반적으로 부정확한 로컬 *RTC(실시간 시계)* 를 추적함
  - RHEL 7 이후 버전부터 적용
- 네트워크 연결을 사용할 수 없는 경우 `chronyd` 서비스는 RTC 시계 드리프트를 계산하고, `/etc/chrony.conf` 구성 파일에서 `driftfile` 값으로 지정된 파일에 기록함

```shell
# /etc/chrony.conf 파일을 수정해 classroom.example.com 서버를 NTP 시간 소스로 사용하도록 변경
[root@host ~]# vi /etc/chrony.conf 
# Use public servers from the pool.ntp.org project.
...output omitted...
server classroom.example.com iburst
...output omitted...

# 설정 변경 반영을 위한 chronyd 서비스 재시작
[root@host ~]# systemctl restart chronyd

# 
[root@host ~]# chronyc sources -v

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current best, '+' = combined, '-' = not combined,
| /             'x' = may be in error, '~' = too variable, '?' = unusable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* 172.25.254.254                3   6    17    26  +2957ns[+2244ns] +/-   25ms

```

- Source mode: server
- Source state: current best
- Startum: 시간 계층 3(낮을수록 원본에 가까우며, 0이 가장 정확)
- Poll: 2^6 == 64초 간격으로 동기화 시도
- Reach: 17(octal) == 00001111(2) == 최근 8번 시도 중 가장 최근 4번 성공
  - 8비트 시프트 레지스터는 8진수로 기록하면 3자리수로 표현 가능(10진수 256 미만의 값)하고, 10진수에 비해 8진수가 비트 표현에 직관적임(?)
  - 255(10) == 377(8) == 011 111 111(2)
- LastRx: 26초전 마지막 응답
- Last sample
  - 2957ns == 조정된 오프셋 (우리 시스템이 이만큼 느리거나 빠름)
  - 2244ns ==  실제 측정된 오프셋
  - +/- 25ms == 시간 오차 범위 (이만큼 오차가 있을 수 있음)