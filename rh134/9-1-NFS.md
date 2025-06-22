# NFS를 사용한 네트워크 연결 스토리지 관리
- NFS 정보를 식별하고, 마운트하는 방법에 대해 소개

## NFS 디렉터리 액세스
- Linux, Unix 운영 체제에서 기본 네트워크 파일 시스템으로 사용하는 인터넷 표준 프로토콜
- NFS는 기본 Linux 권한 및 파일 시스템 특성을 지원하는 개방형 표준
- 기본적으로 Red Hat Enterprise Linux 9에서는 NFS 버전 4.2를 사용
- RHEL은 NFSv3 및 NFSv4 프로토콜을 둘 다 지원
  - NFSv3는 TCP 또는 UDP 전송 프로토콜을 사용할 수 있지만 NFSv4는 TCP 연결만 허용

- NFS 서버는 디렉터리를 내보냄 -> NFS 클라이언트는 내보낸 디렉터리를 기존 로컬 마운트 지점 디렉터리에 마운트
- NFS 클라이언트는 내보낸 디렉터리를 다양한 방법으로 마운트할 수 있음
- 마운트 방법
  - mount 명령을 사용하여 수동으로
  - `/etc/fstab` 파일의 항목을 구성하여 부팅 시 영구적으로
  - 자동 마운터 메서드를 구성하여 온디맨드로
    - `autofs` 사용

## NFS 쿼리
- 클라이언트 입장에서 "서버가 어떤 디렉터리를 공유하고 있는지" 알아야 마운트
- 사용 가능한 NFS 서버를 쿼리하는 메서드는 프로토콜 버전마다 다름

### NFSv3 방식
- NFSv3는 오래된 방식으로, 내부적으로 RPC (Remote Procedure Call) 를 사용
- NFSv3 서버는 `rpcbind` 서비스라는 걸 돌리고 있어야 합니다. 이 서비스는 클라이언트에게 NFS 포트를 알려주는 역할을 함
- 클라이언트는 서버의 포트 111에 있는 rpcbind에 연결해서 "NFS 서비스 어디야?"라고 물어보고, 그 응답을 받아 실제 마운트 진행
#### 디렉터리 확인 방법
- `showmount --exports server이름`
```bash
Export list for server
/shares/test1
/shares/test2

```
- 이걸 보면 서버가 `/shares/test1`, `/shares/test2` 디렉터리를 공유 중이라는 걸 알 수 있음

### NFSv4 방식
- NFSv4는 RPC를 사용하지 않음
- 따라서 rpcbind 서비스가 필요하지 않고, `showmount` 명령도 동작하지 않음
→ `showmount`를 써도 응답이 없거나 타임아웃
- NFSv4는 "내보낸 디렉터리들의 트리 구조"를 제공합니다.
  - 클라이언트가 서버의 루트(/)만 마운트하면, 그 아래 경로를 쭉 탐색해서 어떤 디렉터리를 공유했는지 직접 확인할 수 있음
- example
```bash
mkdir /mountpoint
mount server:/ /mountpoint
ls /mountpoint
```
→ /mountpoint 아래에 여러 공유 디렉터리들이 보일 수 있음
  - 여기서 보인다고 해서 자동으로 마운트된 것은 아니고, 탐색만 가능한 상태
  - 실제로 접근하려면 해당 디렉터리를 명시적으로 마운트해야 함

## NFS 마운트
- 마운트할 로컬 마운트 지점이 아직 없는 경우 먼저 생성 필요
- `/mnt` 디렉터리는 임시 마운트 지점으로 사용할 수 있지만 장기 또는 영구 마운트에는 /mnt 를 사용하지 않는 것 권장
- `[root@host ~]# mkdir /mountpoint`

### 임시 mount
- 로컬 볼륨 파일 시스템과 마찬가지로 NFS 공유 디렉토리를 마운트하여 해당 콘텐츠에 액세스
  - NFS 공유는 권한 있는 사용자만 마운트할 수 있음
- `[root@host ~]# mount -t nfs -o rw,sync server:/export /mountpoint`
  - `-t nfs`: NFS 파일 시스템 유형을 지정(mount 명령이 server:/export 구문을 탐지하는 경우 명령의 기본값은 NFS 유형)
  - `-o rw,sync`
    - 쉼표로 구분된 옵션 목록을 mount 명령에 추가할 수 있음
    - `rw` 옵션은 내보낸 파일 시스템이 읽기/쓰기 액세스를 통해 마운트되도록 지정
    - `sync` 옵션은 내보낸 파일 시스템에 대한 동기 트랜잭션을 지정. 이 방법은 트랜잭션을 완료하거나 실패로 반환해야 하는 모든 프로덕션 네트워크 마운트에 사용
  - 영구적이지 않다는 것 주의
  - 테스트 용도 사용 권장

### 영구 mount
- NFS 내보내기를 영구적으로 마운트하려면 `/etc/fstab` 파일을 편집하여 수동 마운트와 구문이 유사한 마운트 항목을 추가
```bash
[root@host ~]# vim /etc/fstab
...
server:/export  /mountpoint  nfs  rw  0 0

```
- 그러면 마운트 지점만 사용하여 NFS 내보내기를 마운트할 수 있음
  -`mount` 명령은 `/etc/fstab` 파일의 일치하는 항목에서 NFS 서버 및 마운트 옵션을 가져옴
  - `[root@host ~]# mount /mountpoint`

## NFS 디렉터리 마운트 해제
- `umount` 명령을 사용하여 NFS 내보내기를 마운트 해제
-  `/etc/fstab` 파일이 존재하는 경우 공유를 마운트 해제해도 파일에서 해당 항목이 제거되지 않음
  -  `/etc/fstab` 파일의 항목은 영구적이며 부팅 중 다시 마운트
- `[root@host ~]# umount /mountpoint`
- 마운트된 디렉터리는 간혹 마운트 해제할 수 없으며 장치가 사용 중이라는 오류를 반환
  - 애플리케이션이 파일 시스템 내에서 파일을 열어 두었거나, 일부 사용자의 쉘에서 작업 중인 디렉터리가 마운트된 파일 시스템의 루트 디렉터리 또는 그 아래에 있어 장치가 사용 중인 경우 발생
- `lsof(list open files)` 명령을 사용하여 마운트 지점을 쿼리하여 해당 파일을 열린 상태로 유지하는 프로세스를 쿼리
```bash
[root@host ~]# lsof  /mountpoint
COMMAND  PID   USER  FD   TYPE  DEVICE  SIZE/OFF  NODE  NAME
program  5534  user  txt  REG   252.4   910704    128   /home/user/program
```