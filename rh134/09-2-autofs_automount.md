# 네트워크 연결 스토리지 자동 마운트

## 자동 마운터를 사용해 NFS 내보내기 마운트

- *자동 마운터*는 파일 시스템 및 NFS 내보내기를 온디맨드로 자동 마운트하고, 마운트된 리소스가 현재 더 이상 사용되지 않는 경우 파일 시스템 및 NFS 내보내기를 자동으로 마운트 해제하는 서비스(`autofs`)
- 자동 마운터 기능은 권한이 없는 사용자에게 mount 명령을 사용할 수 있는 충분한 권한이 없는 문제를 해결하기 위해 고안됨
  -  일반 사용자가 CD, DVD와 같은 이동식 미디어 및 이동식 디스크 드라이브에 액세스하려면 mount 명령을 사용해야 함
  - 부팅 시 `/etc/fstab` 구성을 사용하여 로컬 또는 원격 파일 시스템을 마운트해야 일반 사용자가 해당 시스템에 액세스할 수 있음
- 자동 마운터 제어 파일 시스템은 사용자 또는 애플리케이션이 파일에 액세스하기 위해 파일 시스템 마운트 지점을 입력할 때 온디맨드로 마운트됨

### 자동 마운터 이점

- 파일 시스템은 프로그램에서 열려 있는 파일을 읽고 쓸 때만 리소스를 사용하기 때문에 자동 마운터 파일 시스템의 리소스 사용량은 부팅 시 마운트된 파일 시스템과 동일함
- 자동 마운터의 이점은 파일 시스템을 더 이상 사용하지 않을 때마다 마운트 해제하여 파일 시스템이 열려 있는 동안 발생하는 예기치 않은 손상으로부터 보호함

### 자동 마운터 autofs 서비스 메서드

- `autofs` 서비스는 NFS 및 SMB 파일 공유 프로토콜을 포함하여 `/etc/fstab` 파일과 같은 로컬 및 원격 파일 시스템을 지원

### 직접 및 간접 맵 사용 사례

- 자동 마운터는 직접 및 간접 마운트 지점 매핑을 모두 지원하여 두 가지 유형의 마운트 요청을 처리함

- *직접* 마운트는 파일 시스템이 변경되지 않는 알려진 마운트 지점 위치에 마운트되는 경우

  - *직접* 마운트 지점은 다른 일반 디렉터리와 마찬가지로 영구 디렉터리로 존재

- *간접* 마운트는 마운트 요구가 발생할 때까지 마운트 지점 위치를 알 수 없는 경우

  - 간접 마운트의 한 예는 원격 마운트 홈 디렉터리 구성으로, 사용자 홈 디렉터리의 디렉터리 경로에 사용자 이름이 포함됨

- 추가 예시

  - 간접 맵

    ```shell
    # /etc/auto.master.d/indirect.autofs
    /shares  /etc/auto.indirect
    
    # /etc/auto.indirect
    work   -rw,sync  hosta:/nfs/work
    dev    -rw,sync  hosta:/nfs/dev
    logs   -rw,sync  hosta:/nfs/logs
    ```

    - 사용자가 `/shares/work`에 접근하면 → `hosta:/nfs/work`가 마운트됨
    - `work`, `dev`, `logs`는 **키(key)** 역할을 함
    - autofs가 `/shares/work` 디렉터리를 **자동 생성**해줌

  - 직접맵

    ```shell
    # /etc/auto.master.d/direct.autofs
    /-  /etc/auto.direct
    
    # /etc/auto.direct
    /mnt/docs   -rw,sync  hosta:/nfs/docs
    /opt/tools  -rw,sync  hosta:/nfs/tools
    ```

    - 사용자가 `/mnt/docs`에 접근하면 → `hosta:/nfs/docs`가 마운트됨
    - **절대 경로가 key** 역할을 함

## 자동 마운터 서비스 구성

- `autofs` 서비스를 사용하려면 `autofs` 및 `nfs-utils` 패키지를 설치해야 함

```shell
[root@host ~]# dnf install autofs nfs-utils
```

### 마스터 맵 파일

- 마스터 맵 파일(`/etc/auto.master`)은 `autofs` 서비스의 기본 구성 파일
-  `/etc/autofs.conf` 파일을 사용하여 `autofs` 서비스의 마스터 맵 파일을 변경할 수 있음
- `/etc/auto.master.d` 디렉터리를 사용하여 마스터 맵 파일을 구성함
  - 마운트 지점의 기본 디렉터리를 확인하고 자동 마운트를 생성하는 매핑 파일을 확인하는 역할

```shell
[root@host ~]# cat /etc/auto.master.d/demo.autofs
mount-point     map-file
```

- mount-point
  - 자동 마운트의 기본 마운트 지점으로 사용할 디렉터리
- map-file
  - 맵 파일로 사용할 파일
  - `autofs` 서비스를 시작하기 전에 맵 파일을 생성해야 함

### 맵 파일

- 개별 온디맨드 마운트 지점의 속성을 구성함

```shell
[root@host ~]# cat /etc/auto.demo
mount-point     mount-options     source-location
```

- mount-point
  - autofs 마운트 지점
- mount-options
  - autofs 마운트 지점을 마운트하는데 사용할 옵션
- source-location
  - 마운트의 소스 위치

#### 간접 맵 파일 생성

```shell
# 간접 자동 마운트의 경우, 마스터 맵 파일에 다음 콘텐츠를 포함
[root@host ~]# cat /etc/auto.master.d/indirect.autofs
/shares  /etc/auto.indirect
```

-  간접 자동 마운트의 기본 마운트 지점으로 `/shares` 디렉터리를 사용
- `/etc/auto.indirect` 맵 파일에는 마운트 세부 사항이 포함

```shell
[root@host ~]# cat /etc/auto.indirect
work  -rw,sync  hosta:/shares/work
```

- 정규화된 마운트 지점은 `/shares/work`(마스터 맵 파일의 기본 마운트 지점)
- 마운트 옵션은 대시 문자(`-`)로 시작하며 공백 없이 쉼표로 구분됨

#### 직접 맵 파일 생성

- 직접 맵은 NFS 내보내기를 절대 경로 마운트 지점에 매핑
-  직접 맵 파일은 하나만 필요하며 여러 개의 직접 맵을 포함할 수 있음

```shell
# 
[root@host ~]# cat /etc/direct.autofs
/-  /etc/auto.direct
```

- 매핑 파일 이름 지정 규칙은 `/etc/auto.name`이며, 여기서 *name* 에는 맵 콘텐츠가 반영
- 모든 직접 맵 항목은 기본 디렉터리로 `/-` 경로를 사용

```shell
[root@host ~]# cat /etc/auto.direct
/mnt/docs  -rw,sync  hosta:/shares/docs
```

- 마운트 지점(또는 키)은 항상 절대 경로

#### 맵 파일에서 와일드카드 사용

- NFS 서버가 디렉터리 내에서 여러 하위 디렉터리를 내보내는 경우 자동 마운터가 단일 매핑 항목을 사용하여 그중 임의의 하위 디렉터리에 액세스하도록 구성할 수 있음
- 앞의 예제에서 `hosta:/shares` 내보내기가 두 개 이상의 하위 디렉터리를 포함하고 동일한 마운트 옵션을 사용하여 액세스 가능한 경우 `/etc/auto.indirect` 파일의 항목은 다음과 같음

```shell
[root@host ~]# cat /etc/auto.indirect
*  -rw,sync  hosta:/shares/&
```

- 사용자가 `/shares/work`에 액세스를 시도하면 소스 위치의 앰퍼샌드가 `*` 키(이 예제에서 `work` )로 바뀌고, `hosta:/shares/work` 가 마운트됨
- 앰퍼샌드(&)의 의미
  - **해당 키 값을 그대로 대입**하는 변수

## 자동 마운터 서비스 시작

- 마스터 맵 및 맵 파일을 구성한 후 `systemctl` 명령을 사용하여 `autofs` 서비스를 시작하고 활성화

```shell
[root@host ~]# systemctl enable --now autofs
Created symlink /etc/systemd/system/multi-user.target.wants/autofs.service → /usr/lib/systemd/system/autofs.service.
```

