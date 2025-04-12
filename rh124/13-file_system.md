# Linux 파일 시스템 액세스

## 파일 시스템 및 장치 식별

- 블록 디바이스(Block Device)?
  - 데이터를 **고정된 크기의 블록 단위로 읽고 쓰는 저장 장치**를 의미
  - 하드 디스크(HDD), SSD, USB 드라이브, RAID 볼륨 등과 같은 저장 장치가 블록 디바이스에 해당
- 스토리지를 블록 디바이스로 구성하면, 파티션이라는 더 관리하기 쉬운 단위로 나눌 수 있음
- 파티션을 나누면 device file이 /dev 특수 디렉터리에 생성됨(e.g. /dev/sda1...)
- 블록 디바이스에 파일을 저장하려면 파일 시스템이 필요함
- mount?
  - 파일 시스템을 사용 가능하게 만드는 것
- mount point(마운트 지점)?
  - 파일 시스템이 사용 가능하게 만들어진 위치(i.e. 디렉터리)



```
# block device 정보 출력
[root@servera ~]# lsblk --fs

$ blkid /dev/vdb

# 블록 디바이스에 파일 시스템이 없는 케이스
$ echo $?
2

$ blkid /dev/vda4

# mount point, 
$ findmnt -s
```



#### 스토리지 관리 개념
- RHEL(Red Hat Enterprise Linux)은 *XFS(Extents File System)*를 기본 로컬 파일 시스템으로 사용
- RHEL은 로컬 파일 관리를 위해 *확장 파일 시스템(ext4)* 을 지원
- RHEL 9부터 이동식 미디어 사용을 위해 exFAT(Extensible File Allocation Table) 파일 시스템이 지원
- 엔터프라이즈 서버 클러스터에서 공유 디스크는 *GFS2(Global File System 2)* 파일 시스템을 사용하여 동시 다중 노드 액세스를 관리

| 특징                    | XFS                             | EXT4                     | exFAT                 | GFS2               |
| ----------------------- | ------------------------------- | ------------------------ | --------------------- | ------------------ |
| 저널링                  | O(메타데이터)                   | O(메타데이터+데이터옵션) | X                     | O                  |
| 최대 파일 크기          | 8 EiB(Exbibyte, 약 1M TiB 단위) | 16TiB(Tebibyte)          | 16EiB                 | 8EiB               |
| 최대 볼륨 크기          | 8 EiB                           | 1EiB                     | 128PiB                | 8EiB               |
| 온라인 크기 조정        | 확장만 가능                     | X                        | X                     | X                  |
| 데이터 무결성 체크섬    | X                               | X                        | X                     | O                  |
| 다중 사용자(클러스터링) | X                               | X                        | X                     | O                  |
| 운영체제 지원           | Linux                           | Linux                    | Windows, macOS, Linux | Linux(RHEL 기반)   |
| 추천 사용처             | 서버, 대용량 데이터 저장        | 일반 리눅스 환경         | 이동식 저장 장치      | 클러스터 서버 환경 |

- 저널링?
  - 저널링(Journaling)은 **파일 시스템의 무결성을 보장하는 기술**
  - 파일 시스템 변경 사항을 **"저널(Journal, 로그 파일)"**에 먼저 기록한 후 실제 데이터를 변경하는 방식
  - 즉, 시스템이 갑자기 종료되거나 장애가 발생해도 **파일 시스템이 손상되는 것을 방지**
  - EXT4는 저널링 방식을 옵션으로 정할 수 있음
    - data=journal (전체 저널링, Full Journaling)
      - **메타데이터 + 파일 데이터 모두 저널에 기록**한 후, 나중에 디스크에 반영하는 방식
    - data=ordered (기본 설정, Ordered Journaling)
      - **메타데이터는 저널에 기록**하지만, **파일 데이터는 직접 디스크에 씀**
      - 하지만, **파일 데이터가 먼저 디스크에 기록된 후 메타데이터가 저널에 저장됨**
      - 성능과 안정성의 균형을 맞춘 방식
    - data=writeback (비저널링에 가까운 모드)
      - 메타데이터는 저널에 기록하지만, 파일 데이터는 비저널링 방식으로 저장
      - 즉, **파일 데이터가 언제 디스크에 기록될지 보장되지 않음**
      - 가장 빠른 성능을 제공하지만, 충돌 시 데이터 손실 위험이 큼
- 온라인 크기 조정?
  - XFS는 **파일 시스템의 크기를 마운트된 상태에서(운영 중인 상태에서) 확장할 수 있는 기능**을 제공
  - 가능한 이유
    - Allocation Groups(AGs) 구조 사용
      - XFS는 파일 시스템을 **여러 개의 "할당 그룹(Allocation Groups, AGs)"**으로 나눔
      - 각 **AG는 독립적으로 파일을 할당**할 수 있어 병렬 작업이 가능
      - **새로운 AG를 추가하면 기존 AG에 영향을 주지 않고** 파일 시스템을 확장할 수 있음
    - 메타데이터 구조가 미리 예약됨
      - XFS는 파일 시스템을 처음 만들 때 **메타데이터 공간을 미리 할당**하여 저장
      - EXT4 같은 파일 시스템은 **슈퍼블록을 다시 정리해야 할 수도 있어** 온라인 확장이 어렵거나 제한적
  - 온라인 축소(파일 시스템 크기 줄이기)는 불가능함
    - XFS는 AG를 통해 파일을 저장하는데, 특정 AG를 제거하려면 내부 데이터를 이동해야 함

#### 파일 시스템과 마운트 지점

#### 파일 시스템, 스토리지 및 블록 장치

#### 디스크 파티션

| 장치 유형                                 | 장치 이름 패턴                        |
| ----------------------------------------- | ------------------------------------- |
| SATA/SAS/USB 연결 스토리지(SCSI 드라이버) | `/dev/sda`, `/dev/sdb`, `/dev/sdc`, … |
| `virtio-blk` 반가상화 스토리지(VM)        | `/dev/vda`, `/dev/vdb`, `/dev/vdc`,…  |
| `virtio-scsi` 반가상화 스토리지(VM)       | `/dev/sda`, `/dev/sdb`, `/dev/sdc`, … |
| NVMe 연결 스토리지(SSD)                   | `/dev/nvme0`, `/dev/nvme1`, …         |
| SD/MMC/eMMC 스토리지(SD 카드)             | `/dev/mmcblk0`, `/dev/mmcblk1`, …     |

- 일반적으로 전체 스토리지 장치에 하나의 파일 시스템만 생성되지는 않음
- 파티션을 사용하면 디스크를 구분할 수 있고, 파티션을 각기 다른 파일 시스템으로 포맷하거나 다른 용도로 사용할 수 있음
- 반가상화 스토리지란?
  - 가상화 환경에서 성능을 최적화하기 위해 하드웨어를 직접 접근할 수 있도록 설계된 가상화 방식의 스토리지 드라이버

#### 논리 볼륨

| 계층                            | 설명                                                      |
| ------------------------------- | --------------------------------------------------------- |
| 물리 볼륨 (PV, Physical Volume) | 실제 물리 디스크 또는 파티션 (`/dev/sda`, `/dev/sdb1` 등) |
| 볼륨 그룹 (VG, Volume Group)    | 여러 개의 PV를 묶어서 하나의 논리적인 그룹을 형성         |
| 논리 볼륨 (LV, Logical Volume)  | VG에서 특정 크기만큼 할당하여 마운트하는 가상적인 볼륨    |

- 논리 볼륨(Logical Volume, LV)이란?
  - 물리 디스크를 하나의 논리적인 단위로 묶어 유연하게 관리할 수 있도록 해주는 기술
  - 즉, **여러 개의 물리적인 디스크(또는 파티션)를 하나의 가상 디스크처럼 관리**할 수 있도록 해주는 기능
- 논리 볼륨(LV)과 파일 시스템 관계
  - 논리 볼륨(LV)을 생성한 후, 그 위에 **EXT4, XFS 등 파일 시스템을 생성**하고, 해당 파일 시스템을 특정 디렉터리에 **마운트하여 사용**

#### 파일 시스템 검사

- df
  - 로컬 및 원격 파일 시스템 장치에 대한 개요를 표시
  - 총 디스크 공간, 사용된 디스크 공간, 사용 가능한 디스크 공간, 전체 디스크 공간의 백분율이 포함됨

```shell
# host 시스템의 파일 시스템과 마운트 지점을 표시
[user@host ~]$ df
Filesystem     1K-blocks    Used Available Use% Mounted on
devtmpfs          912584       0    912584   0% /dev
tmpfs             936516       0    936516   0% /dev/shm
tmpfs             936516   16812    919704   2% /run
tmpfs             936516       0    936516   0% /sys/fs/cgroup
/dev/vda3        8377344 1411332   6966012  17% /
/dev/vda1        1038336  169896    868440  17% /boot
tmpfs             187300       0    187300   0% /run/user/1000

# 읽기 쉬운 형식으로 조회
[user@host ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        892M     0  892M   0% /dev
tmpfs           915M     0  915M   0% /dev/shm
tmpfs           915M   17M  899M   2% /run
tmpfs           915M     0  915M   0% /sys/fs/cgroup
/dev/vda3       8.0G  1.4G  6.7G  17% /
/dev/vda1      1014M  166M  849M  17% /boot
tmpfs           183M     0  183M   0% /run/user/1000
```

- du
  - 특정 디렉터리 트리 공간에 대한 자세한 정보 조회
  - 현재 디렉터리 트리의 모든 파일 크기를 반복적으로 표시

```shell
# host 시스템의 /usr/share 디렉터리에 대한 디스크 사용량 보고서 조회
[root@host ~]# du /usr/share
...output omitted...
176 /usr/share/smartmontools
184 /usr/share/nano
8 /usr/share/cmake/bash-completion
8 /usr/share/cmake
356676  /usr/share

# 읽기 쉬운 형식으로 조회
[root@host ~]# du -h /usr/share
...output omitted...
176K  /usr/share/smartmontools
184K  /usr/share/nano
8.0K  /usr/share/cmake/bash-completion
8.0K  /usr/share/cmake
369M  /usr/share
```

## 파일 시스템 마운트 및 해제

#### 블록 장치 식별

```shell
# 사용 가능한 모든 장치의 세부 정보를 표시
[root@host ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda    252:0    0   10G  0 disk
├─vda1 252:1    0    1M  0 part
├─vda2 252:2    0  200M  0 part /boot/efi
├─vda3 252:3    0  500M  0 part /boot
└─vda4 252:4    0  9.3G  0 part /
vdb    252:16   0    5G  0 disk
vdc    252:32   0    5G  0 disk
vdd    252:48   0    5G  0 disk
```

```shell
# ext4 파일 시스템을 생성, 생성하지 않고 바로 파일 시스템 마운트는 불가
[root@host ~] mkfs.ext4 /dev/vdc1
```





#### 파티션 이름을 사용해 파일 시스템 마운트

```shell
# /mnt/data 마운트 지점에 /dev/vda4 파티션을 마운트
[root@host ~]# mount /dev/vda4 /mnt/data
```



#### 파티션 UUID를 사용해 파일 시스템 마운트

```shell
[root@host ~]# lsblk --fs
NAME        FSTYPE FSVER LABEL UUID                   FSAVAIL FSUSE% MOUNTPOINTS
/dev/vda
├─/dev/vda1
├─/dev/vda2 vfat   FAT16       7B77-95E7              192.3M     4% /boot/efi
├─/dev/vda3 xfs          boot  2d67e6d0-...-1f091bf1  334.9M    32% /boot
└─/dev/vda4 xfs          root  efd314d0-...-ae98f652    7.7G    18% /
/dev/vdb
/dev/vdc
/dev/vdd

[root@host ~]# mount UUID="efd314d0-b56e-45db-bbb3-3f32ae98f652" /mnt/data
```



#### 파일 시스템 마운트 해제

```shell
# mount point를 인수로 mount 해제
[root@host ~]# umount /mnt/data


# 사용중인 경우 마운트 해제 실패
[root@host ~]# cd /mnt/data
[root@host data]# umount /mnt/data
umount: /mnt/data: target is busy.

# lsof 명령은 열려 있는 모든 파일과 파일 시스템에 액세스하는 프로세스를 표시
# 파일 시스템을 마운트 해제할 수 없게 하는 프로세스를 식별
[root@host data]# lsof /mnt/data
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
bash    1593 root  cwd    DIR 253,17        6  128 /mnt/data
lsof    2532 root  cwd    DIR 253,17       19  128 /mnt/data
lsof    2533 root  cwd    DIR 253,17       19  128 /mnt/data

# 다시 해제
[root@host data]# cd
[root@host ~]# umount /mnt/data
```



## 시스템에서 파일 찾기

#### 파일 검색

- `locate` 명령은 사전 생성된 인덱스에서 파일 이름 또는 파일 경로로 검색하고 결과를 즉시 반환
  - `mlocate` 데이터베이스에서 이 정보를 조회하기 때문에 명령이 빠르게 실행
  - 이 데이터베이스는 실시간으로 업데이트되지 않으므로, 정확한 결과를 얻으려면 자주 업데이트해야 함
  - `locate` 명령이 마지막 데이터베이스 업데이트 이후 생성된 파일을 검색하지 않음
  - `locate` 데이터베이스는 매일 자동으로 업데이트되고,  `root` 사용자는 `updatedb` 명령을 실행하여 강제로 즉시 업데이트 가능
- `find` 명령은 파일 시스템 계층 구조를 구문 분석하여 실시간으로 파일을 검색

#### 이름으로 파일 찾기

```shell
# mlocate DB update
[root@host ~]# updatedb

# 쿼리 passwd 부분적으로 일치하는 파일 이름 또는 경로 출력
[developer@host ~]$ locate passwd
/etc/passwd
/etc/passwd-
/etc/pam.d/passwd
...output omitted...

# -i 옵션은 대소문자 구분않는 검색 수행
[developer@host ~]$ locate -i messages
...output omitted...
/usr/share/locale/zza/LC_MESSAGES
/usr/share/makedumpfile/eppic_scripts/ap_messages_3_10_to_4_8.c
/usr/share/vim/vim82/ftplugin/msmessages.vim
...output omitted...

# 검색 결과 개수 제한
[developer@host ~]$ locate -n 5 passwd
/etc/passwd
/etc/passwd-
/etc/pam.d/passwd
...output omitted...
```



#### 실시간으로 파일 검색

- `find` 명령은 파일 권한, 파일 유형, 크기 또는 수정 시간과 같은 파일 이름 외의 기준으로 파일을 검색 가능

- `find` 명령을 실행하는 사용자는 내용을 검사하려면 디렉터리에 대한 읽기 및 실행 권한이 있어야 함
- `find` 명령에 대한 첫 번째 인수는 검색할 디렉터리
  - 디렉터리 인수를 생략하면 현재 디렉터리에서 검색을 시작하고 모든 하위 디렉터리에서 일치 항목을 찾음
- `find {starting_where} {how} {what}`
- 대부분의 다른 Linux 명령이 이중 대시를 사용하는 것과 달리 `find` 명령의 전체 단어 옵션은 단일 대시를 옵션에 사용

```shell
# / 경로에서 sshd_config 파일 검색
[root@host ~]# find / -name sshd_config
/etc/ssh/sshd_config

# 와일드카드를 이용한 txt 파일 검색
[root@host ~]# find / -name '*.txt'
...output omitted...
/usr/share/libgpg-error/errorref.txt
/usr/share/licenses/audit-libs/lgpl-2.1.txt
/usr/share/licenses/pam/gpl-2.0.txt
...output omitted...

# -iname 옵션은 대소문자 구분 없이 검색
[root@host ~]# find / -iname '*messages*'
/sys/power/pm_debug_messages
/usr/lib/locale/C.utf8/LC_MESSAGES
/usr/lib/locale/C.utf8/LC_MESSAGES/SYS_LC_MESSAGES
...output omitted...
```



#### 소유권 또는 권한을 기준으로 파일 검색

```shell
[developer@host ~]$ find -user developer
.
./.bash_logout
./.bash_profile
...output omitted...

[developer@host ~]$ find -group developer
.
./.bash_logout
./.bash_profile
...output omitted...

[developer@host ~]$ find -uid 1000
.
./.bash_logout
./.bash_profile
...output omitted...

[developer@host ~]$ find -gid 1000
.
./.bash_logout
./.bash_profile
...output omitted...

[root@host ~]# find /home -perm 764
...output omitted...
[root@host ~]# find /home -perm u=rwx,g=rw,o=r
...output omitted...

# ls 옵션은 파일에 대한 정보 제공
[root@host ~]# find /home -perm 764 -ls
 26207447   0 -rwxrw-r--   1 user  user   0 May 10 04:29 /home/user/file1
```



#### 크기를 기준으로 파일찾기

```shell
[developer@host ~]$ find -size 10M
...output omitted...

[developer@host ~]$ find -size +10G
...output omitted...

[developer@host ~]$ find -size -10k
...output omitted...
```



#### 수정 시간을 기준으로 파일 검색

```shell
# 120분 전에 콘텐츠가 변경된 파일 모두 검색
[root@host ~]# find / -mmin 120
...output omitted...

# 200분 이상 전에 콘텐츠가 변경된 파일 모두 검색
[root@host ~]# find / -mmin +200
...output omitted...

# 150분 미만 전에 변경된 파일 검색
[root@host ~]# find / -mmin -150
...output omitted...
```



#### 파일 유형을 기준으로 파일 검색

- 일반 파일의 경우 f 플래그
- 디렉터리의 경우 d 플래그
- 소프트 링크의 경우 l 플래그
- 블록 장치의 경우 b 플래그

```shell
# 디렉터리 검색
[root@host ~]# find /etc -type d
/etc
/etc/tmpfiles.d
/etc/systemd
/etc/systemd/system
/etc/systemd/system/getty.target.wants
...output omitted...

# 소프트링크 검색
[root@host ~]# find / -type l
...output omitted...

# 블록 장치 검색
[root@host ~]# find /dev -type b
/dev/vda1
/dev/vda
```

