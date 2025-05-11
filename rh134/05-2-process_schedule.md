# 시스템 성능 튜닝

## 프로세스 스케줄링에 미치는 영향

- nice 및 renice 명령을 사용하여 특정 프로세스에 우선순위를 지정하거나 해제

### Nice value

<img width="1179" alt="Image" src="https://github.com/user-attachments/assets/1cdeed32-34ea-4a90-9c17-6e0b6ce3f83b" />

- 프로세스 우선순위를 정하는 설정값
- -20 (우선순위 증가) ~ 19(우선순위 감소) 범위의 값, 기본값 0
- top 명령과 ps 명령에서 NI 값을 조회할 수 있음
  - 일반적으로 PR(priority)열의 값은 `NI + 20`으로 표시됨
  - NI는 사용자단에서 설정하는 프로세스 우선순위 힌트의 개념, PR 값은 커널 내부에서 설정되는 값임
  - ps 명령에서 조회되는 CLS열은 일반적으로 TS(TimeSharing)값을 가지며, 실시간 프로세스의 경우(FF: First in First out, RR: Round Robin)등의 값을 가질 수 있고 이는 스케줄링에 영향을 미침

```
Tasks: 192 total,   1 running, 191 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  1.6 sy,  0.0 ni, 96.9 id,  0.0 wa,  0.0 hi,  1.6 si,  0.0 st
MiB Mem :   5668.6 total,   4655.6 free,    470.1 used,    542.9 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   4942.6 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
      1 root      20   0  172180  16232  10328 S   0.0   0.3   0:01.49 systemd
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.01 kthreadd
      3 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_gp
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_par_gp
```

```
[user@host ~]$ ps axo pid,comm,nice,cls --sort=-nice
  PID COMMAND          NI CLS
   33 khugepaged       19  TS
   32 ksmd              5  TS
  814 rtkit-daemon      1  TS
    1 systemd           0  TS
    2 kthreadd          0  TS
    5 kworker/0:0-cgr   0  TS
    7 kworker/0:1-rcu   0  TS
    8 kworker/u4:0-ev   0  TS
   15 migration/0       -  FF
```



### 사용자가 설정한 NI 값으로 프로세스 시작 및 변경

```shell
# 일반적인 프로세스 백그라운드 실행
[user@host ~]$ sleep 60 &
[1] 2667

# 기본 생성된 프로세스의 NI는 0
[user@host ~]$ ps -o pid,comm,nice 2667
  PID COMMAND          NI
 2667 sleep            0
 
# nice 명령 프로세스 실행
[user@host ~]$ nice sleep 60 &
[1] 2736

# nice로 생성된 프로세스의 NI는 10
[user@host ~]$ ps -o pid,comm,nice 2736
  PID COMMAND          NI
 2736 sleep            10
 
 # NI를 15로 설정한 프로세스 실행
[user@host ~]$ nice -n 15 sleep 60 &
[1] 2673

[user@host ~]$ ps -o pid,comm,nice 2740
  PID COMMAND          NI
 2740 sleep            15

# 이미 생성된 프로세스의 NI값 변경
[user@host ~]$ renice -n 19 2740
2740 (process ID) old priority 15, new priority 19
```