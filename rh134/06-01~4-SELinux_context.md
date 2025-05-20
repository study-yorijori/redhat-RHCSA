# SELinux 보안 관리

## SELinux 아키텍처

- *SELinux(Security Enhanced Linux)* 는 Linux의 중요한 보안 기능
- 파일, 포트 및 기타 리소스에 대한 액세스가 매우 세밀한 수준으로 제어됨
- 프로세스는 해당 SELinux 정책 또는 SELinux 부울 설정에서 지정하는 리소스에만 액세스할 수 있음
- 기존에 일반적으로 사용되던 파일 권한은 특정 사용자 또는 그룹의 파일 액세스 권한을 제어하지만, 파일 액세스 권한이 있는 사용자가 파일을 의도하지 않은 용도로 사용하는 것을 방지하지 못함
  - 예를 들어 파일에 대한 쓰기 권한이 있는 경우, 다른 편집기나 프로그램에서 특정 프로그램만 작성할 수 있도록 설계된 구조화된 데이터 파일을 여전히 열고 수정할 수 있으면 손상 또는 데이터 보안 문제가 발생할 수 있음
- SELinux는 애플리케이션 개발자가 애플리케이션에서 사용하는 각 바이너리 실행 파일, 구성 파일, 데이터 파일에 대해 허용되는 조치와 액세스 권한을 선언하기 위해 정의한 애플리케이션별 정책으로 구성됨
-  하나의 정책이 애플리케이션 하나의 활동을 정의하기 때문에 이 정책을 *타겟 정책*이라고 함
  - 정책은 개별 프로그램, 파일, 네트워크 포트에 구성된 사전 정의 레이블을 선언함

## SELinux 사용

- SELinux는 프로세스와 리소스 간에 허용된 작업을 명시적으로 정의하는 액세스 규칙 집합을 적용함
- 액세스 규칙에 정의되어 있지 않은 작업은 허용되지 않음
- 타겟 정책이 있는 애플리케이션 또는 서비스는 *제한된* 도메인에서 실행되는 반면, 정책이 없는 애플리케이션은 *제한 없이* 실행되며 SELinux로 보호되지 않음
- SELinux 작동 모드
  - **Enforcing(적용)** : SELinux가 로드된 정책을 적용함. 이 모드는 Red Hat Enterprise Linux의 기본값.
  - **Permissive(허용)** : SELinux가 정책을 로드하고 활성 상태지만 액세스 제어 규칙을 적용하지 않고 액세스 위반을 기록함. 이 모드는 애플리케이션 및 규칙을 테스트하고 문제를 해결하는 데 유용함.
  - **Disabled(비활성화됨)**: SELinux가 꺼져 있습니다. SELinux 위반이 거부되거나 기록되지 않음. SELinux는 비활성화하지 않는 것이 좋음.
    - RHEL 9부터는 부팅 시 `selinux=0` 커널 매개 변수를 사용해야만 SELinux를 완전히 비활성화할 수 있음
    - `/etc/selinux/config` 파일에서 `SELINUX=disabled` 옵션 설정을 지원하지 않으며, 만약 설정할 경우 SELinux에서 활성 상태의 적용을 시작하고 수행하지만 정책은 로드하지 않음.
      - 정책 규칙은 허용된 작업을 정의하므로 정책이 로드되지 않으면 모든 작업이 거부됨

## 기본 SELinux 개념

- SELinux의 기본 목적은 손상된 애플리케이션 또는 시스템 서비스로 인한 부적절한 사용으로부터 사용자 데이터를 보호하는 것
- SELinux는 *MAC(Mandatory Access Control)*이라는 세분화된 규칙에 정의된 오브젝트 기반 보안 기능을 추가로 제공함
  - MAC 정책은 모든 사용자에게 적용되므로 임의 구성 설정을 통해 특정 사용자에게 적용되지 않게 할 수 없음
- 파일, 프로세스, 디렉터리 또는 포트와 같은 모든 리소스 엔터티에는 *SELinux 컨텍스트*라는 레이블이 있음
  - 컨텍스트 레이블은 프로세스에서 레이블이 지정된 리소스에 액세스할 수 있도록 정의된 SELinux 정책 규칙을 찾음
- SELinux 레이블에는 `user`, `role`, `type`, `security level` 필드가 있음. 
- 타겟 정책은 RHEL에서 기본적으로 활성화되어 있으며, `type` 컨텍스트를 사용하여 규칙을 정의함



<img width="1164" alt="Image" src="https://github.com/user-attachments/assets/74603475-362e-470f-9e45-039c765e1f45" />

- /var/www/html/file2 파일에 대한 SELinux 보안 컨텍스트

<img width="1174" alt="Image" src="https://github.com/user-attachments/assets/33ed7ba6-8ef4-44d3-a2bf-e65baf94fecb" />

- 웹 서버 프로세스
  - /usr/sbin/httpd
  - PID: 958
  - SELinux context type: `httpd_t`
- 웹 서버 리소스
  - /var/www/html/index.html
  - SELinux context type: `httpd_sys_content_t`
- SELinux Policy
  - allow httpd_t httpd_sys_content_t:file(read write)

```shell
# 리소스를 나열하는 대부분의 명령은 -Z 옵션으로 SELinux 컨텍스트를 관리함(ps, ls, cp, mkdir...)
# 프로세스의 SELinux 컨텍스트 조회
[root@host ~]# ps axZ
LABEL                               PID TTY      STAT   TIME COMMAND
system_u:system_r:kernel_t:s0         2 ?        S      0:00 [kthreadd]
system_u:system_r:kernel_t:s0         3 ?        I<     0:00 [rcu_gp]
system_u:system_r:kernel_t:s0         4 ?        I<     0:00 [rcu_par_gp]
...output omitted...

# 웹서버 실행
[root@host ~]# systemctl start httpd

# 웹서버 프로세스의 SELinux 컨텍스트 조회
[root@host ~]# ps -ZC httpd
LABEL                               PID TTY          TIME CMD
system_u:system_r:httpd_t:s0       1550 ?        00:00:00 httpd
system_u:system_r:httpd_t:s0       1551 ?        00:00:00 httpd
system_u:system_r:httpd_t:s0       1552 ?        00:00:00 httpd
system_u:system_r:httpd_t:s0       1553 ?        00:00:00 httpd
system_u:system_r:httpd_t:s0       1554 ?        00:00:00 httpd

# /var/www 경로의 파일에 대한 SELinux 컨텍스트 조회
[root@host ~]# ls -Z /var/www
system_u:object_r:httpd_sys_script_exec_t:s0 cgi-bin
system_u:object_r:httpd_sys_content_t:s0 html
```

```shell
# 현재 SELinux 모드 조회
[root@host ~]# getenforce
Enforcing

[root@host ~]# setenforce
usage:  setenforce [ Enforcing | Permissive | 1 | 0 ]

# 0: permissive 설정
[root@host ~]# setenforce 0
[root@host ~]# getenforce
Permissive

# 1: enforcing 설정
[root@host ~]# setenforce Enforcing
[root@host ~]# getenforce
Enforcing
```



## SELinux 파일 컨텍스트 제어

### 초기 SELinux 컨텍스트

- 프로세스, 파일, 포트와 같은 모든 리소스에는 SELinux *컨텍스트*로 레이블이 지정됨
- SELinux는 `/etc/selinux/targeted/contexts/files/` 디렉터리에서 파일 레이블 지정 정책에 대한 파일 기반 데이터베이스를 유지 관리함
- 새 파일은 해당 파일 이름이 기존 레이블 지정 정책과 일치하는 경우 기본 레이블을 가져옴
- 파일을 새 위치에 복사하면 해당 파일의 SELinux 컨텍스트가 새 위치의 레이블 지정 정책 또는 상위 디렉터리 상속(정책이 없는 경우)으로 결정된 새 컨텍스트로 변경될 수 있음
- 복사하는 동안 파일의 SELinux 컨텍스트를 보존하여 파일의 원래 위치에 대해 결정된 컨텍스트 레이블을 유지할 수 있음
  - 예를 들어 `cp -p` 명령은 가능한 경우 모든 파일 특성을 유지하고 `cp --preserve=context` 명령은 복사하는 동안 SELinux 컨텍스트만 유지함
  - 이전에 파일 압축 단원에서 관련 개념이 등장했음!
  - 파일을 동일한 파일 시스템 내에서 이동하는 경우에는 이동해도 inode가 생성되지 않고, 대신 기존 inode의 파일 이름이 새 위치로 이동함
  - 파일에 새 컨텍스트를 설정하지 않는 한 `mv` 를 사용하여 이동한 파일은 SELinux 컨텍스트가 유지됨

```shell
# /tmp 하위에 2개의 파일 생성
[root@host ~]# touch /tmp/file1 /tmp/file2
[root@host ~]# ls -Z /tmp/file*
unconfined_u:object_r:user_tmp_t:s0 /tmp/file1
unconfined_u:object_r:user_tmp_t:s0 /tmp/file2

# /var/www/html 경로의 SELinux context 조회
[root@host ~]# ls -Zd /var/www/html/
system_u:object_r:httpd_sys_content_t:s0 /var/www/html/
[root@host ~]# ls -Z /var/www/html/index.html
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/index.html

# 같은 SELinux context를 갖던 2개의 파일에 대해 복사, 이동의 경우 SELinux context 차이 존재
[root@host ~]# mv /tmp/file1 /var/www/html/
[root@host ~]# cp /tmp/file2 /var/www/html/
[root@host ~]# ls -Z /var/www/html/file*
unconfined_u:object_r:user_tmp_t:s0 /var/www/html/file1
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/file2
```



### SELinux 컨텍스트 변경

- 파일의 SELinux 컨텍스트는 `semanage fcontext`, `restorecon`, `chcon` 명령으로 관리할 수 있음
-  권장되는 방법은 `semanage fcontext` 명령을 사용하여 파일 컨텍스트 정책을 생성한 다음 `restorecon` 명령을 사용하여 정책에 지정된 컨텍스트를 파일에 적용하는 것
- `chcon` 명령은 SELinux 컨텍스트를 파일에서 직접 변경하지만 시스템의 SELinux 정책은 참조하지 않음
  -  `chcon` 은 테스트 및 디버깅에 유용하지만 이 방법을 사용하여 컨텍스트를 수동으로 변경하는 것은 일시적임
  - SELinux database에 내용 변경을 트랜잭션하지 않아서, restorecon을 이후 사용하면 다시 변경됨

```shell
# / 디렉터리에서 상속된 default_t SELinux 컨텍스트를 가진 /virtual 생성
[root@host ~]# mkdir /virtual
[root@host ~]# ls -Zd /virtual
unconfined_u:object_r:default_t:s0 /virtual

# /virtual을 httpd_sys_content_t로 변경
[root@host ~]# chcon -t httpd_sys_content_t /virtual
[root@host ~]# ls -Zd /virtual
unconfined_u:object_r:httpd_sys_content_t:s0 /virtual

# 컨텍스트 기본 값인 /default_t로 재설정
[root@host ~]# restorecon -v /virtual
Relabeled /virtual from unconfined_u:object_r:httpd_sys_content_t:s0 to unconfined_u:object_r:default_t:s0
[root@host ~]# ls -Zd /virtual
unconfined_u:object_r:default_t:s0 /virtual
```



### SELinux 기본 파일 컨텍스트 정책 정의

- `semanage fcontext` 명령은 기본 파일 컨텍스트를 결정하는 정책을 표시하고 수정함

- `semanage fcontext -l` 명령을 실행하여 모든 파일 컨텍스트 정책 규칙을 나열할 수 있음

- 정책을 볼 때 가장 일반적인 확장 정규 표현식은 `(/.*)?`이며, 일반적으로 디렉터리 이름에 추가됨

  - 이 표기법은 유머러스하게 *해적*(pirate)이라고 함. 얼굴에 눈 가리개가 있고 그 옆에 갈고리 손이 있는 것처럼 보이기 때문

  -  '슬래시로 시작하고 그 뒤에 임의의 수의 문자가 있는 문자 집합'

  - ```shell
    # /var/www/cgi-bin 아래 있는 모든 파일 및 하위디렉터리의 파일에 context 적용
    /var/www/cgi-bin(/.*)?  all files  system_u:object_r:httpd_sys_script_exec_t:s0
    ```

```shell
# 파일들의 SELinux context 조회
[root@host ~]# ls -Z /var/www/html/file*
unconfined_u:object_r:user_tmp_t:s0 /var/www/html/file1
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/file2

# 전체 SELinux context 정책 조회
[root@host ~]# semanage fcontext -l
...output omitted...
/var/www(/.*)?       all files    system_u:object_r:httpd_sys_content_t:s0
...output omitted...

# /var/www/ 하위 파일들의 SELinux context 재설정
[root@host ~]# restorecon -Rv /var/www/
Relabeled /var/www/html/file1 from unconfined_u:object_r:user_tmp_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
[root@host ~]# ls -Z /var/www/html/file*
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/file1
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/file2
```

```shell
# /virtual 디렉터리 및 하위 파일 생성
[root@host ~]# mkdir /virtual
[root@host ~]# touch /virtual/index.html
[root@host ~]# ls -Zd /virtual/
unconfined_u:object_r:default_t:s0 /virtual
[root@host ~]# ls -Z /virtual/
unconfined_u:object_r:default_t:s0 index.html

# 디렉터리에 대한 SELinux 파일 컨텍스트 정책 추가
[root@host ~]# semanage fcontext -a -t httpd_sys_content_t '/virtual(/.*)?'

# 디렉터리 및 하위 파일의 기본 SELinux 컨텍스트 재적용
# F는 force, vv는 very verbose을 나타냄
[root@host ~]# restorecon -RFvv /virtual
Relabeled /virtual from unconfined_u:object_r:default_t:s0 to system_u:object_r:httpd_sys_content_t:s0
Relabeled /virtual/index.html from unconfined_u:object_r:default_t:s0 to system_u:object_r:httpd_sys_content_t:s0
[root@host ~]# ls -Zd /virtual/
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /virtual/
[root@host ~]# ls -Z /virtual/
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 index.html


# path 기본 정책에 대한 조회
[root@host ~]# semanage fcontext -l -C
SELinux fcontext     type         Context

/virtual(/.*)?       all files    system_u:object_r:httpd_sys_content_t:s0
```