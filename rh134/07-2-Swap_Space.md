# 스왑 공간 관리

## 스왑 공간의 개념

- *스왑 공간*은 Linux 커널 메모리 관리 하위 시스템에서 제어하는 디스크 영역
- 커널은 메모리에 비활성 페이지를 보관하여 시스템 RAM을 보완하기 위해 스왑 공간을 사용
  - 비활성 페이지? -> 최근에 접근되지 않은 메모리 페이지로, 커널이 당장 필요하지 않을 것으로 판단한 페이지
    - 메모리페이지의 상태 용어
      - `Active`: 최근에 접근됨 (읽기/쓰기 등)
      - `Inactive`: 한동안 접근되지 않음
      - `Free`: 아직 아무 프로세스도 사용하지 않음
      - `Swapped out`: 디스크의 스왑 공간으로 옮겨진 상태
- 시스템의 *가상 메모리*에는 결합된 시스템 RAM과 스왑 공간이 포함됨
- 시스템의 메모리 사용량이 정의된 한도를 초과할 경우 커널은 RAM에서 프로세스에 할당된 유휴 메모리 페이지를 검색
- 커널은 유휴 페이지를 스왑 영역에 쓰고 RAM 페이지를 다른 프로세스에 다시 할당함
- 프로그램에서 디스크의 페이지에 액세스해야 하는 경우 커널은 메모리의 다른 유휴 페이지를 찾아 디스크에 쓴 다음 스왑 영역에서 필요한 페이지를 불러옴
- 스왑 영역이 디스크에 있으므로 스왑이 RAM에 비해 속도가 느림



## 스왑 공간 생성

- 파일 시스템 유형이 `linux-swap`인 파티션을 생성
- 장치에서 스왑 시그니처를 저장

```shell
[root@host ~]# parted /dev/vdb
GNU Parted 3.4
Using /dev/vdb
Welcome to GNU Parted! Type 'help' to view a list of commands.

(parted) print
Model: Virtio Block Device (virtblk)
Disk /dev/vdb: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
 1      1049kB  1001MB  1000MB               data

# partition 생성
(parted) mkpart
Partition name?  []? swap1
File system type?  [ext2]? linux-swap
Start? 1001MB
End? 1257MB

(parted) print
Model: Virtio Block Device (virtblk)
Disk /dev/vdb: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system     Name   Flags
 1      1049kB  1001MB  1000MB                  data
 2      1001MB  1257MB  256MB   linux-swap(v1)  swap1

(parted) quit
Information: You may need to update /etc/fstab.
```

```shell
# 시스템이 새 파티션을 감지하고 /dev 디렉터리에 관련 장치 파일을 생성할 때까지 기다림
[root@host ~]# udevadm settle
```



## 스왑 공간 활성화

- `swapon` 명령을 사용하여 포맷된 스왑 공간을 활성화
  - -a 옵션으로 `/etc/fstab` 파일에 나열된 모든 스왑 공간을 활성화
  - --show 및 free 명령으로 사용 가능한 스왑 공간을 검사

```shell
[root@host ~]# free
              total        used        free      shared  buff/cache   available
Mem:        1873036      134688     1536436       16748      201912     1576044
Swap:             0           0           0

# /dev/vdb2 파티션을 스왑 공간으로 활성화
[root@host ~]# swapon /dev/vdb2
[root@host ~]# free
              total        used        free      shared  buff/cache   available
Mem:        1873036      135044     1536040       16748      201952     1575680
Swap:        249852           0      249852
```

- `swapoff` 명령을 사용하여 스왑 공간을 비활성화할 수 있음



### 스왑 공간 영구 활성화

- `/etc/fstab` 파일에 항목을 생성하여 시스템 부팅 시 활성 스왑 공간을 확보

```shell
# /etc/fstab의 위에서 생성한 스왑 공간 정보를 담은 행
UUID=39e2667a-9458-42fe-9665-c5c854605881   swap   swap   defaults   0 0
```

- 마지막 두 필드는 `dump` 플래그와 `fsck` 순서
  - 스왑 공간은 백업 또는 파일 시스템 점검이 필요하지 않으므로 해당 필드를 0으로 설정
- `/etc/fstab` 파일에 항목을 추가하거나 제거할 때 `systemctl daemon-reload` 명령을 실행하거나, `systemd`가 새 구성을 등록하도록 서버를 재부팅



## 스왑 공간 우선순위 설정

- 기본적으로 시스템은 우선 순위에 따라 스왑 공간을 사용
-  커널은 우선순위가 가장 높은 스왑 공간이 가득 찰 때까지 사용한 다음, 우선순위가 낮은 스왑 공간을 사용하기 시작
- 스왑 공간의 기본 우선 순위는 낮으며, 새 스왑 공간은 이전 스왑 공간보다 우선 순위가 낮음
- 스왑 공간의 우선 순위가 같을 경우 커널은 라운드 로빈 방식으로 스왑 공간에 씀
- 우선순위를 설정하려면 `/etc/fstab` 파일에서 `pri` 옵션을 사용하며, 기본 우선 순위는 -2

```shell
# /etc/fstab의 스왑 공간 정보를 담은 행
UUID=af30cbb0-3866-466a-825a-58889a49ef33   swap   swap   defaults  0 0
UUID=39e2667a-9458-42fe-9665-c5c854605881   swap   swap   pri=4     0 0
UUID=fbd7fa60-b781-44a8-961b-37ac3ef572bf   swap   swap   pri=10    0 0
```

