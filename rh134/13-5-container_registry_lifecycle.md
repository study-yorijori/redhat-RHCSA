## 컨테이너 이미지 레지스트리

### 컨테이너 레지스트리

- 컨테이너 이미지는 애플리케이션을 실행하는 데 필요한 모든 종속성이 포함된 패키지 버전의 애플리케이션
- 컨테이너 이미지 레지스트리에 컨테이너 이미지를 저장하고 관리함

```shell
# registry.redhat.io에서 관리하느 ubi9/ubi:9.1 이미지 다운로드
[user@host ~]$ podman pull registry.redhat.io/ubi9/ubi:9.1
Trying to pull registry.redhat.io/ubi9/ubi:9.1...
Getting image source signatures
...output omitted...
Writing manifest to image destination
Storing signatures
3434...8f6b
```

### Redhat 레지스트리

- Redhat은 두 개의 레지스트리를 사용해 컨테이너 이미지를 배포함
  - `registry.access.redhat.com`: 인증이 필요하지 않음
  - `registry.redhat.io`: 인증이 필요함
- Red Hat Ecosystem Catalog를 사용하여 이미지를 검색하고 이미지에 대한 기술 세부 정보를 얻을 수 있음
  - https://catalog.redhat.com/en/search?searchType=containers
- 유용한 컨테이너 이미지
  - registry.access.redhat.com/ubi9
    - RHEL 9를 기반으로 하는 기타 이미지를 생성하기 위한 기본 이미지
  - registry.access.redhat.com/ubi9/python-312
    - Python 3.12 런타임이 포함된 UBI 기반 이미지
  - registry.access.redhat.com/ubi9/nodejs-18
    -  Node.js 18 런타임이 포함된 UBI 기반 이미지
  - registry.access.redhat.com/ubi9/go-toolset
    -  Go 런타임이 포함된 UBI 기반 이미지
- UBI 기반 이미지를 사용하거나 배포하기 위해서는 Red Hat 서브스크립션이 필요하지 않음
  - 컨테이너가 Red Hat OpenShift Container Platform(RHOCP) 또는 RHEL과 같은 Red Hat 플랫폼에 배포된 경우에만 UBI 기반 컨테이너를 완벽하게 지원함
- Red Hat Registry는 Red Hat 및 인증된 프로바이더의 이미지만 저장하지만 Quay.io 레지스트리를 사용하여 사용자 지정 이미지를 저장할 수 있음
  - Quay.io에 공용 이미지를 저장하는 것은 무료
  - 유료 고객은 프라이빗 리포지토리 혜택

### Skopeo를 사용해 레지스트리 관리

- *Skopeo* 는 컨테이너 이미지 작업을 위한 커맨드라인 툴
- 제공 기능
  - 원격 컨테이너 이미지 검사
  - 레지스트리 간 컨테이너 이미지 복사
  - OpenPGP 키를 통한 이미지 서명
  - 이미지 형식 변환(예: `Docker` 형식에서 `OCI` 형식으로)

```shell
# 이미지 메타데이터 조회
[user@host ~]$ skopeo inspect \
 docker://registry.access.redhat.com/ubi9/nodejs-18
{
    "Name": "registry.access.redhat.com/ubi9/nodejs-18",
    "Digest": "sha256:741b...22e0",
    "RepoTags": [
...output omitted...


# 이미지 복사
[user@host ~]$ skopeo copy \
 docker://registry.access.redhat.com/ubi9/nodejs-18 \
 docker://quay.io/myuser/nodejs-18
Getting image source signatures
...output omitted...
```

### Podman으로 레지스트리 자격 증명 관리

```shell
# 레지스트리에서 이미지 다운로드 실패 - 자격 정보 없음
[user@host ~]$ podman pull registry.redhat.io/rhel9/nginx-120
Trying to pull registry.redhat.io/rhel9/nginx-120:latest...
Error: initializing source docker://registry.redhat.io/ubi9/httpd-24:latest: unable to retrieve auth token: invalid username/password: unauthorized: Please login to the Red Hat Registry using your Customer Portal credentials. Further instructions can be found here: https://access.redhat.com/RegistryAuthentication

# 레지스트리 로그인
[user@host ~]$ podman login registry.redhat.io
Username: YOUR_USER
Password: YOUR_PASSWORD
Login Succeeded!

# 레지스트리에서 이미지 다운로드 성공
[user@host ~]$ podman pull registry.redhat.io/rhel9/nginx-120
Trying to pull registry.redhat.io/rhel9/nginx-120:latest...
Getting image source signatures
...output omitted...
```

- Podman은 자격 증명을 `${XDG_RUNTIME_DIR}/containers/auth.json` 파일에 저장됨
  - `${XDG_RUNTIME_DIR}` 은 현재 사용자와 관련된 디렉터리를 나타냄



## 컨테이너 라이프 사이클 관리

<img width="1185" height="776" alt="Image" src="https://github.com/user-attachments/assets/04430b69-df6f-4e57-aabe-bd61da16b5aa" />

- 컨테이너 status 중 pause?
  - `pause`와 `stop`은 모두 컨테이너의 실행을 멈추는 방식이지만 멈추는 수준과 방식이 다름
  - pause
    - 컨테이너 안의 **모든 프로세스를 `SIGSTOP` 시그널로 일시 정지**시킴. 마치 `kill -STOP <pid>` 한 것과 같음.
    - 메모리에 그대로 유지됨.
    - CPU 스케줄링은 되지 않음 (실행은 멈췄지만, 리소스는 점유).
    - `podman unpause` 명령어로 빠르게 복구 가능.
  - stop
    - 컨테이너 안의 메인 프로세스에 **`SIGTERM` (혹은 `SIGKILL`)을 보내 종료**시킴. 일반적으로 **완전히 중지**하는 의미.
    - 프로세스가 종료됨 → 상태는 `exited`.
    - 메모리 및 리소스 해제.
    - 다시 실행하려면 `podman start` 필요 → 컨테이너 프로세스 재시작.
- podman 명령어 구성은 docker와 거의 유사

```shell
[user@host ~]$ podman ps
CONTAINER ID  IMAGE         COMMAND              CREATED STATUS PORTS       NAMES
0ae7be593698  ...server     /bin/sh -c python... ...ago  Up...  ...8000/tcp httpd
c42e7dca12d9  ...helloworld /bin/sh -c nginx...  ...ago  Up...  ...8080/tcp nginx

[user@host ~]$ podman inspect 7763097d11ab
[
    {
        "Id": "7763...cbc0",
        "Created": "2022-05-04T10:00:32.988377257-03:00",
        "Path": "container-entrypoint",
        "Args": [
            "/usr/bin/run-httpd"
        ],
        "State": {
            "OciVersion": "1.0.2-dev",
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 9746,
...output omitted...
        "Image": "d2b9...fa0a",
        "ImageName": "registry.access.redhat.com/ubi8/httpd-24:latest",
        "Rootfs": "",
...output omitted...
            "Env": [
                "PATH=/opt/app-root/src/bin:/opt/app-root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "TERM=xterm",
                "container=oci",
                "HTTPD_VERSION=2.4",
...output omitted...
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "CgroupConf": null
        }
    }
    

# 컨테이너 중지
[user@host ~]$ podman stop 1b982aeb75dd
1b982aeb75dd

# 컨테이너 강제 중지
[user@host ~]$ podman kill httpd
https

# 컨테이너 다시 시작
[user@host ~]$ podman restart nginx
1b98...75dd

# 컨테이너 일시 중지
[user@host ~]$ podman pause 4f2038c05b8c
4f2038c05b8c

# 컨테이너 제거
[user@host ~]$ podman rm c58cfd4b90df
c58c...90df
```

### 부팅시 컨테이너화된 서비스 시작

- 웹 서버 또는 데이터베이스와 같은 애플리케이션이 부팅 시 시작되고 systemd 서비스로 무기한 실행되도록 구성

#### systemd 유닛 파일 생성

```shell
[user@host ~]$ podman generate systemd --name web --files
/home/user/container-web.service
```

- `--files` 옵션은 표준 출력(STDOUT)으로 출력하는 대신 파일을 생성함
- `web`이라는 하나의 컨테이너에 대한 `container-web.service` 유닛 파일을 생성
- 성된 서비스 구성을 검토하고 요구 사항에 맞게 조정한 후 유닛 파일을 사용자의 systemd 구성 디렉터리(`~/.config/systemd/user/`)에 저장함

```shell
[user@host ~]$ systemctl --user daemon-reload

[user@host ~]$ systemctl --user [start, stop, status, enable, disable] container-web.service
```



#### 컨테이너화된 서비스 관리