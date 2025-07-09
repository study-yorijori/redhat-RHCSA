# 부팅 프로세스 제어

## 사전 지식

### BIOS와 UEFI

- BIOS와 UEFI는 컴퓨터의 **펌웨어 인터페이스**
  - 펌웨어?
    - **펌웨어**는 메인보드(마더보드)에 내장된 **비휘발성 메모리(NVRAM, 플래시 등)** 에 저장된 **하드웨어 초기화 및 부팅 담당 프로그램**
    - 운영체제가 실행되기 **전**에 실행되는 프로그램
    - 컴퓨터의 전원이 켜졌을 때 가장 먼저 작동
    - 마더보드에 있는 **ROM/플래시 메모리 칩**에 저장됨
    - 운영체제를 재설치해도 **펌웨어는 유지됨**
- 운영체제가 부팅되기 전에 하드웨어 초기화 및 부트 로더를 실행하는 역할

| 항목             | BIOS                    | UEFI                      |
| ---------------- | ----------------------- | ------------------------- |
| 출시 시기        | 오래됨 (1970~)          | 비교적 최신               |
| 파티션 방식      | MBR(Master Boot Record) | GPT(GUID Partition Table) |
| 부트 모드        | 16비트                  | 32/64비트                 |
| 최대 디스크 용량 | 2TB                     | 9.4ZB 이상                |
| 보안 부팅        | 불가능                  | Secure Boot 지원          |
| 부팅 속도        | 느림                    | 빠름                      |
| GUI 지원         | 불가 (텍스트 모드)      | 가능 (마우스 사용)        |

### GRUB

- **GRand Unified Bootloader**의 줄임말로, 리눅스 시스템에서 널리 사용되는 **멀티 부트 로더**
  - 운영체제를 메모리에 로딩해주는 **중간 관리자**
  - 여러 운영체제 또는 커널을 선택하여 부팅 가능
  - GRUB2는 그 두 번째 버전으로, 대부분의 최신 리눅스 배포판에서 사용됨
- 주요 역할
  - 부트 메뉴 제공
    - 예: Ubuntu, Fedora, Windows를 동시에 설치해두었다면, 어떤 OS를 부팅할지 선택 가능
  - 커널 로딩
    - `/boot/` 아래 있는 리눅스 커널(`vmlinuz`)과 초기 RAM 디스크(`initramfs`)를 메모리로 로딩
  - 부트 설정 관리
    - `grub.cfg` 파일을 읽어 부트 옵션, 타임아웃, 기본 커널 등을 설정
  - 복구 옵션 제공
    - 커널 문제 발생 시 "리커버리 모드"로 부팅할 수 있음

### initramfs(Initial RAM Filesystem)

- 부팅 초기에 **루트 파일 시스템(rootfs)** 을 마운트하기 전, 커널이 **임시로 메모리에 올리는 루트 파일 시스템**
- 역할
  - **드라이버 로딩**: 디스크, 파일시스템, RAID, LVM 등 필요한 커널 모듈을 먼저 로딩
  - **root 파티션 마운트**: 실제 루트 디렉터리(`/`)를 마운트할 수 있도록 준비
  - 그 후, 제어를 커널이 아니라 실제 루트 파일 시스템에 넘김 (`/sbin/init` 또는 `systemd` 실행)
- 구조
  - `cpio` 포맷으로 압축된 파일 시스템 이미지 (`initramfs`는 실제로는 `initrd.img` 등의 이름으로 존재)
  - 위치: `/boot/initramfs-<kernel_version>.img`
- 동작 순서 요약
  - 커널 로드 → initramfs 로드 → init 스크립트 실행 → rootfs 마운트 → systemd 실행

### EFI System Partition(ESP, EFI 파티션)

- **UEFI 기반 시스템**에서 사용되는 특별한 파티션
- 펌웨어가 운영체제를 부팅할 때 사용하는 **EFI 실행 파일 (.efi)** 들이 저장됨
- 역할
  - 부트로더 저장: GRUB, systemd-boot, Windows Boot Manager 등
  - UEFI 펌웨어가 이 파티션에서 `.efi` 파일을 찾아서 실행
  - 복수의 OS (Linux, Windows 등)를 설치해도 각각 자신의 `.efi` 부트로더를 이 파티션에 둠

## 부팅 프로세스

<img width="1161" height="689" alt="Image" src="https://github.com/user-attachments/assets/cae8fe6d-c2d2-40c3-8fdf-4193d6d8a655" />

### BIOS 기반 시스템의 부팅 프로세스

- **Hardware → POST → Hardware Initialized**
  - 전원을 켜면 하드웨어가 작동하고, POST(Power-On Self-Test)로 기본 장치 상태를 점검함
- **BIOS Firmware**
  - BIOS가 실행되어 부팅 가능한 디스크를 찾음
- **MBR (Master Boot Record)**
  - 디스크의 처음 512바이트에서 MBR을 읽어 부트로더를 로드
- **Bootloader (예: GRUB)**
  - MBR에 저장된 부트로더가 `/boot/grub2/` 디렉토리에서 설정 파일을 찾음
- **GRUB2 Menu**
  - 사용자가 커널을 선택하거나 기본 항목으로 자동 진입
- **Kernel / Initramfs**
  - 선택된 리눅스 커널과 초기 RAM 파일 시스템(initramfs) 로딩
- **systemd**
  - 시스템 초기화 프로세스 시작 (initrd.target → 기본 타겟)
- **Login Prompt**
  - 로그인 가능한 상태로 진입

### UEFI 기반 시스템의 부팅 프로세스

- **Hardware → POST → Hardware Initialized**
  - BIOS와 동일하게 하드웨어 초기화 수행
- **UEFI Firmware**
  - BIOS보다 향상된 펌웨어가 NVRAM에 저장된 EFI 부팅 엔트리를 사용하여 EFI 앱을 탐색
- **GPT (GUID Partition Table)**
  - GPT 기반 디스크에서 부팅 정보를 탐색
- **EFI Partition (/boot/efi)**
  - EFI 시스템 파티션을 마운트하여 EFI 애플리케이션 실행 (예: GRUB)
- **GRUB2 Menu**
  - `/boot/grub2/grub.cfg`를 읽어 부트 메뉴 표시
- **Kernel / Initramfs**
  - 선택된 커널 및 initramfs를 메모리에 로딩
- **systemd**
  - 시스템 초기화 시작
- **Login Prompt**
  - 로그인 가능한 상태로 진입



### Systemd 타겟 선택

- systemd 타겟은 원하는 상태에 도달하기 위해 시작해야 하는  systemd 유닛 집합

| 타겟              | 목적                                                         |
| ----------------- | ------------------------------------------------------------ |
| graphical.target  | 멀티 유저를 지원하고, 그래픽 및 텍스트 기반 로그인을 제공    |
| multi-user.target | 멀티 유저를 지원하고, 텍스트 기반 로그인을 제공              |
| rescue.target     | 시스템을 복구할 수 있는 단일 사용자 시스템을 제공            |
| emergency.target  | `rescue.target` 장치가 시작되지 않을 때 시스템을 복구하기 위해 가장 최소한의 시스템을 시작 |

```shell
# graphical target 유닛에는 multi-user.target이 포함되며, basic.target 유닛 및 기타 유닛에 종속됨
[user@host ~]$ systemctl list-dependencies graphical.target | grep target
graphical.target
* └─multi-user.target
*   ├─basic.target
*   │ ├─paths.target
*   │ ├─slices.target
*   │ ├─sockets.target
*   │ ├─sysinit.target
*   │ │ ├─cryptsetup.target
*   | | ├─integritysetup.target
*   │ │ ├─local-fs.target
...output omitted...
```



### 런타임시 타겟 선택

```shell
[root@host ~]# systemctl isolate multi-user.target
```

- 타겟을 격리하면 해당 타겟(및 해당 종속성)에서 요구하지 않는 모든 서비스가 중지되며 아직 시작되지 않은 모든 필수 서비스가 시작됨



### 기본 타겟 설정

- 일반적으로 `/etc/systemd/system/` 디렉터리의 기본 타겟은 `graphical.target` 또는 `multi-user.target` 타겟에 대한 심볼릭 링크
- 볼릭 링크를 직접 편집하는 대신 `systemctl` 명령에는 이 링크를 관리하는 두 가지 하위 명령, `get-default` 및 `set-default`가 제공됨



### 부팅 시 다른 타겟 선택

1. 시스템을 부팅 또는 재부팅합니다.
2. 정상적으로 부팅을 시작하는 **Enter** 키를 제외한 임의의 키를 눌러 부트 로더 메뉴 카운트다운을 중단합니다.
3. 커서를 시작할 커널 항목으로 이동합니다.
4. **e** 를 눌러 현재 항목을 편집합니다.
5. 커서를 `linux` 로 시작하는 행으로 이동합니다. 이는 커널 명령줄입니다.
6. `systemd.unit={target}.target` 를 추가합니다(예: `systemd.unit=emergency.target`).
7. **Ctrl**+**x** 를 눌러 이러한 변경 사항을 적용하여 부팅합니다.

```shell
load_video
set gfxpayload=keep
insmod gzio
linux ($root)/vmlinuz-5.14.0-70.13.1.e19_0.x86_64 root=UUID=fb535add-9799-4a27-b8bc-e8259f39a767 console=tty0 console=ttyS0,115200n8 no_timer_check net.ifnames=0 crashkernel=1G-4G:192M,4G64G:256M,64G-:512M systemd.unit=emergency.target
initrd ($root)/initramfs-5.14.0.70.13.1.e19_0.x86_64.img $tuned_initrd
```

