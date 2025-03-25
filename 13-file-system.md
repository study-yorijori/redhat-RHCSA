# Linux File System
## 사전 지식
-  Block Storage(블록 스토리지), Object Storage(오브젝트 스토리지) 비교
### Block Storage
- 데이터를 고정 크기의 블록(섹터 단위)으로 나눠 저장
- 서버 또는 OS에서 직접 마운트하여 디스크처럼 사용 가능
- 빠른 데이터 액세스 (고성능, 저지연)
- OS에서 디스크처럼 다룰 수 있어 데이터베이스, 가상머신(VM) 등에 적합

### Object Storage
- 데이터를 객체(Object) 단위로 저장 (파일 + 메타데이터 + 고유한 ID)
- 파일 시스템 없이 API 기반 접근 (HTTP, REST API)
- 무제한 확장 가능 (페타바이트급 데이터 저장 가능)
- 블록 스토리지보다 속도가 느림 (랜덤 액세스 어려움)

## 스토리지 관리
- xfs를 기본 로컬 파일 시스템으로 사용
- ext4, exFAT, GFS2 등 파일 시스템도 지원(리눅스 버전마다 다를 수 있음)

| 파일 시스템 | 특징 | 장점 | 단점 | 사용 사례 |
|-------------|------|------|------|-----------|
| **XFS** | 저널링 기반 고성능 파일 시스템 | 대용량 파일 강점 | 작은 파일 성능 낮음, ext4보다 복구 속도 느림 | 데이터베이스, 고성능 서버, 엔터프라이즈 스토리지 |
| **ext4** | 리눅스 기본 파일 시스템 | 안정적이고 범용적 1EB까지 저장 가능 (이론상) | 파일 크기 및 inode 한계 있음, 매우 큰 파일 성능은 XFS보다 떨어짐 | 리눅스 서버, 일반 데스크탑, 웹서버 |
| **exFAT** | 마이크로소프트가 개발한 가벼운 파일 시스템 | 파일 크기 제한 거의 없음 |  저널링 없음 (데이터 손상 위험), 보안 기능 부족 | USB, SD카드, 외장 하드 (다양한 OS에서 사용) |
| **GFS2** | 클러스터 파일 시스템 (Red Hat) | 여러 서버에서 동시에 접근 가능 , 데이터 무결성 보장 | 단일 서버 환경에서는 불필요한 오버헤드, 설정 복잡 |  고가용성 클러스터, 공유 스토리지 |

- 저널링 파일 시스템이란?
  - 파일 변경 사항을 기록(log)한 후 실제 데이터에 반영하는 방식
  - 데이터 손실과 파일 시스템 손상을 방지
  - 동작 방식
    1. 파일을 수정하면 메타데이터와 변경 사항을 저널(journal)에 먼저 기록
    2. 변경 사항이 안전하게 저장된 후, 실제 파일 시스템에 적용
    3. 갑작스러운 정전이나 시스템 장애 발생 시 저널을 읽어 파일 시스템을 빠르게 복구 가능
  - 추가적인 디스크 쓰기 작업 발생
  - ext3, ext4, XFS, NTFS, JFS, Btrfs, GFS2 지원

- 왜 XFS가 대용량에 강할까?
  - B-Tree 기반 인덱싱 활용, 대용량 파일을 더 빠르게 탐색
  - ext4는 다중 블록 할당을 사용하지만 B-Tree만큼 효율적이지 않음
  - XFS는 다중 CPU 코어에서 동시에 파일 시스템 작업을 수행할 수 있도록 병렬 I/O 처리 지원

- 왜 ext4는 작은 파일에 강할까?
  - 디스크 공간 효율적 사용
  - 저널링 외에도 작은 파일을 효율적으로 저장하는 다양한 최적화 기능

### 파일 시스템, 스토리지 및 블록 장치 
- `/dev` 경로는 저널링 외에도 작은 파일을 효율적으로 저장하는 다양한 최적화 기능
- 탐지된 순서대로 드라이브 네이밍 변경 `/dev/sda`, `/dev/sdb` .. 
- 장치 타입별로 네이밍이 다름 ex. 가상화 디바이스`dev/vda`
- host 시스템에 있는 `/dev/sda` 장치 파일의 확장 목록에는 블록 장치를 나타내는 b 파일 유형이 표시
```bash
[user@host ~]$ ls -l /dev/sda1
brw-rw----. 1 root disk 8, 1 Feb 22 08:00 /dev/sda1
```

### 파티션
- 파티션 이유
  - OS 와 앱 디스크를 분리해 사용자 데이터 안정성 확보
  - 사용성에 따라 파티션을 세팅해 성능 최적화
  - 데이터 보호 및 백업
  - 파티션 별로 보안 및 접근 제어 설정

- 파티션된 스토리지에 각각 원하는 파일 시스템을 만들 수 있음
- 파티션을 하지 않고도 파일 시스템은 만들 수 있음
- 논리 볼륨(LVM)
  - 하드디스크의 공간을 유연하게 관리할 수 있도록 해주는 기술
  - 고정된 파티션 방식(MBR, GPT)과 달리, 논리적으로 디스크 공간을 묶고 확장/축소가 가능
  - notation
    - PV (Physical Volume, 물리 볼륨): 실제 디스크 또는 파티션
    - VG (Volume Group, 볼륨 그룹): 여러 개의 물리 볼륨(PV)을 묶어서 하나의 큰 저장소처럼 사용
    - LV (Logical Volume, 논리 볼륨): VG에서 원하는 크기만큼 할당하여 마치 파티션처럼 사용

- `fdisk /dev/sdX` 명령어로 파티션 가능

### 파일 시스템 검사
- `bf`
  - host 시스템의 파일 시스템과 마운트 지점 표시
  - `tmpfs` 및 `devtmpfs` 장치는 시스템 메모리에 있는 파일 시스템
  - df 명령의 -h 또는 -H 옵션은 사용자가 읽을 수 있는 옵션으로, 출력 크기의 가독성을 개선
  - **"1K-blocks"**: 각 파일 시스템의 크기와 사용량을 1KB(=1024바이트) 단위
    - /dev/vda3의 총 용량 = 8,377,344 KB (약 8GB)
  ```bash
  [user@host ~]$ df
  Filesystem     1K-blocks    Used Available Use% Mounted on
  devtmpfs          912584       0    912584   0% /dev
  tmpfs             936516       0    936516   0% /dev/shm
  tmpfs             936516   16812    919704   2% /run
  tmpfs             936516       0    936516   0% /sys/fs/cgroup
  /dev/vda3        8377344 1411332   6966012  17% /
  /dev/vda1        1038336  169896    868440  17% /boot
  tmpfs             187300       0    187300   0% /run/user/1000
  ```

- `du`
  - 특정 디렉터리 트리 공간에 대한 정보
  - du 명령의 -h 및 -H 옵션은 출력을 사용자가 읽을 수 있는 형식으로 변환
  ```bash
  [root@host ~] du -h /usr/share
  ...output omitted...
  176K  /usr/share/smartmontools
  184K  /usr/share/nano
  8.0K  /usr/share/cmake/bash-completion
  8.0K  /usr/share/cmake
  369M  /usr/share
  ```

## 파일 시스템 마운트 및 마운트 해제
- 블록 장치에 파일 시스템을 설정하고 마운트
### 블록 장치 식별
- `lsblk` 명령을 사용하여 지정된 블록 장치 또는 사용 가능한 모든 장치의 세부 정보를 표시
  - `MAJ:MIN`: Major:Minor 번호, 장치 식별 번호
  - `RM`: 제거 가능 여부(0: 고정 디스크, 1: 이동 디스크)
  - `RO`: 읽기 전용 여부(0: 읽기/쓰기 가능, 1: 읽기 전용)
  - `TYPE`: 장치 유형 (disk: 물리적 디스크, part: 파티션)
  ```bash
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
- `lsblk --fs`
  - 파일 시스템에서는 UUID로 장치를 식별
  - 파티션이 안 보이더라도 파일 시스템이 있을 수 있음
  ```bash
  NAME   FSTYPE  LABEL UUID                                 MOUNTPOINTS
  vda                                                     
  ├─vda1 vfat    EFI   1234-ABCD                           /boot/efi
  ├─vda2 ext4          e99f8c6d-8a1f-4b9b-9d4d-6a2b63e41e38 /boot
  └─vda3 xfs           5a31c2e4-87d6-4f9c-b0b2-2e239aa4c945 /
  vdb                                                     
  vdc                                                     
  vdd                                                     
  ```

### 파일 시스템 포멧
- 파티션을 ext4 파일 시스템으로 포맷
  - **파일 시스템의 UUID** 생성
  ```bash
  $ sudo mkfs.ext4 /dev/vdc1
  mke2fs 1.46.5 (30-Dec-2021)
  Creating filesystem with 1310720 4k blocks and 327680 inodes
  Filesystem UUID: 3f9e5f2a-3c3e-4d5a-889b-2d8ff4a12345
  Superblock backups stored on blocks: 
          32768, 98304, 163840, 229376, 294912, 819200, 884736

  Allocating group tables: done                           
  Writing inode tables: done                            
  Creating journal (16384 blocks): done
  Writing superblocks and filesystem accounting information: done
  ```
- `blkid /dev/vdb`: 블록 디바이스(디스크 및 파티션)의 파일 시스템 유형과 UUID, LABEL 등을 확인
```bash
/dev/vdb: UUID="f2d1a6c3-0b7a-4f2a-bdd6-abc123456789" TYPE="ext4"
```

### 파일 시스템 마운트
- 파티션 이름 사용
  - `[root@host ~]# mount /dev/vda4 /mnt/data`
- UUID 사용
  - `[root@host ~]# mount /dev/vda4 /mnt/data`
- `umount`로 마운트 헤제 가능


## 시스템에서 파일 찾기
- `find` 명령을 사용해서 탐색 가능
### 파일 검색
- `find / -name sshd_config`: 이름 기반 검색
- 패턴 검색
```bash
[root@host ~]# find / -name '*.txt'
...output omitted...
/usr/share/libgpg-error/errorref.txt
/usr/share/licenses/audit-libs/lgpl-2.1.txt
/usr/share/licenses/pam/gpl-2.0.txt
...output omitted...
```
- 대소문자 커버
```bash
[root@host ~]# find / -iname '*messages*'
/sys/power/pm_debug_messages
/usr/lib/locale/C.utf8/LC_MESSAGES
/usr/lib/locale/C.utf8/LC_MESSAGES/SYS_LC_MESSAGES
...output omitted...
```
### 권한 기준 검색
- 사용자 기준 검색
```bash
[developer@host ~]$ find -user developer
.
./.bash_logout
./.bash_profile
...output omitted...
[developer@host ~]$ find -uid 1000
.
./.bash_logout
./.bash_profile
...output omitted...
```
- `-perm` 사용
  - man page
  > -perm [-|+]mode
  >           The mode may be either symbolic (see chmod(1)) or an octal number.  If the mode is symbolic, a starting value of zero is assumed
  >           and the mode sets or clears permissions without regard to the process' file mode creation mask.  If the mode is octal, only bits
  >           07777 (S_ISUID | S_ISGID | S_ISTXT | S_IRWXU | S_IRWXG | S_IRWXO) of the file's mode bits participate in the comparison.  If the
  >           mode is preceded by a dash (“-”), this primary evaluates to true if at least all of the bits in the mode are set in the file's mode
  >           bits.  If the mode is preceded by a plus (“+”), this primary evaluates to true if any of the bits in the mode are set in the file's
  >           mode bits.  Otherwise, this primary evaluates to true if the bits in the mode exactly match the file's mode bits.  Note, the first
  >           character of a symbolic mode may not be a dash (“-”).
  - `-perm mode`: 정확히 일치하는 파일
  - `-perm -mode`: 최소한 이 권한을 포함하는 파일
    - `find . -type f -perm -644`
    - `-rw-rw-r—(644)`: O
    - `-rwxr—r—(744)`: O
    - `-rw-r——(640)`: X
  - `-perm +mode`: 적어도 하나라도 포함된
    - `find . -type f -perm +644`
    - `-rw-r-----`(640): O
    - `-rwx------`(700): O
  - `find /etc -type f -perm /o=w` : ohters 가 최소 쓰기권한인 파일을 찾는 것

### 크기를 기준으로 탐색
- `find -size 10M`: 크기가 정확히 10M
- `find -size +10G`: 크기가 10M 보다 큰
- `find -size -10G`: 크기가 10M 보다 작은

### 파일 유형을 기준으로 파일 검색
- `-type` 옵션은 검색 범위를 지정된 파일 유형으로 제한합
  - f: 일반 파일의 경우
  - d: 디렉터리
  - l: 소프트 링크
  - b: 블록 장치
  - `find /dev -type b`

### 수정 시간 기준 검색
- `find / -mmin 120`: 120분 전에 콘텐츠가 변경된 파일

### 조회 결과를 활용해 명령
- `find /usr -size +100M -exec ls -lh {} \;` : find 결과에 대해 설명
  - {}: result find
  - `;`: exec 명령 끝 의미
    - `;`이 명령어 구분자로 사용되지 않도록 백슬래쉬를 줌

### locate 명령어 (비추)
- 파일 이름이나 경로를 기준으로 파일을 검색
- mlocate 데이터베이스에서 이 정보를 조회하기 때문에 `find`보다 빠르게 실행됨
- 이 데이터베이스는 실시간으로 업데이트되지 않으므로, 정확한 결과를 얻으려면 자주 업데이트해야 함
  - `[root@host ~]# updatedb`
- Usage
  - 키워드 검색
  ```bash
  [developer@host ~]$ locate passwd
  /etc/passwd
  /etc/passwd-
  /etc/pam.d/passwd
  ...output omitted...
  ```
  - 대소문자를 구분하지 않는 검색
  ```bash
  [developer@host ~]$ locate -i messages
  ...output omitted...
  /usr/share/locale/zza/LC_MESSAGES
  /usr/share/makedumpfile/eppic_scripts/ap_messages_3_10_to_4_8.c
  /usr/share/vim/vim82/ftplugin/msmessages.vim
  ...output omitted...
  ```




 