# Software Package 설치 및 업데이트

## 사전 지식

### RPM Package

- RPM: Red Hat Package Manager의 약자
- Red Hat 계열 리눅스 배포판에서 사용하는 소프트웨어 패키지 형식
- `.rpm` 확장자를 가진 파일로 배포됨
- RPM 패키지는 Software(바이너리, 라이브러리, 설정 파일 등)와 설치 스크립트, 메타데이터(버전, 의존성 등)를 포함한 단일 파일
- e.g. `httpd-2.4.46-1.fc33.x86_64.rpm (Apache 웹 서버 패키지)`

### RPM의 구성

1. 메타데이터: 패키지 이름, 버전, 아키텍처, dependencies 정보
2. 파일 목록: 설치 시 어디에 어떤 파일이 배치될지 정의
3. 스크립트: 설치 전/후 실행될 명령

```shell
$ rpm -ivh package.rpm
```

- -i: 설치, -v: verbose(상세 출력), -h: 해시 표시
- 의존성을 자동으로 해결하지 않으므로, 보통 DNF를 사용함

### DNF(Dandified Yum)

- Fedora, CentOS, RHEL 같은 리눅스 배포판에서 사용되는 패키지 관리 도구
- Yum의 후속 버전
- DNF는 소프트웨어 패키지를 설치, 업데이트, 제거하는데 사용됨
- Package Repository라는 원격 저장소를 참조함

#### **DNF와 Repository의 동작**

1. **리포지토리 설정 확인**: /etc/yum.repos.d/ 디렉토리에 .repo 파일로 정의됩니다. 예: fedora.repo
2. **패키지 검색 및 설치**: dnf install <패키지명> 명령어로 리포지토리에서 패키지를 찾아 설치.
3. **업데이트**: dnf update로 리포지토리의 최신 패키지를 시스템에 반영.
4. **의존성 해결**: DNF는 패키지의 의존성을 자동으로 계산하고 필요한 다른 패키지도 함께 설치합니다.

## Software 패키지 설치 및 업데이트

### Red Hat 구독(subscription) 관리

- Red Hat Subscription Management는 시스템에 제품 구독 자격을 부여하기 위한 툴을 제공함
- 웹 콘솔도 있고, CLI를 통해 액세스할 수 있으며 Ansible을 이용해 자동화할 수도 있음
- Red Hat Content Delivery Network(CDN)에서 콘텐츠 받으려면 구독(subscription)이 필요

## RPM Software Package 설명 및 조사

- RPM 패키지 파일 이름은 네가지 요소 {name}-{version}-{release}.{architecture}.rpm으로 구성됨
  - version: 원본 소프트웨어의 버전 번호
  - release: 패키지의 릴리스 번호, 패키지 제작자가 설정, 패키지 제작자는 원본 소프트웨어 개발자가 아닐 수 있음
  - architecture: 패키지가 실행되도록 컴파일된 프로세서 아키텍처
- 일반적으로 소프트웨어 프로바이더는 GPG(GNU Privacy Guard) 키를 사용하여 RPM 패키지에 디지털 서명함
  - RPM 시스템은 패키지가 적절한 GPG 키로 서명되었는지 확인하여 패키지 무결성을 확인
  - GPG 서명이 일치하지 않으면 RPM 시스템에서 패키지를 설치하지 않음
- RPM 커맨드를 이용해 RPM 파일과 인터페이스를 할 수 있음
- RPM DB는 설치된 소프트웨어 패키지 목록
  - /var/lib/rpm 경로에 저장

### RPM 패키지 검사

- `rpm` 유틸리티는 패키지 파일 내용 및 설치된 패키지에 대한 정보를 검색할 수 있는 하위 수준 툴
- 기본적으로 이 툴은 설치된 패키지의 로컬 데이터베이스에서 정보를 가져옴
- `rpm` 명령의 `-p` 옵션을 사용하여 다운로드했지만 설치되지 않은 패키지 파일에 대한 정보를 가져옴

```shell
# rpm -qf {PATH}: PATH에 해당하는 디렉터리(/etc/yum.repos.d) 혹은 파일을 제공하는 패키지를 확인
# /etc/yum.repos.d는 package repository를 관리하는 디렉터리이고, 이는 redhat-release 패키지에 의해 생성됨 
[user@host ~]$ rpm -qf /etc/yum.repos.d
redhat-release-9.1-1.0.el9.x86_64

# rpm -q: 현재 설치된 패키지 버전을 출력
[user@host ~]$ rpm -q dnf
dnf-4.10.0-4.el9.noarch

# rpm -ql: 패키지가 설치한 모든 파일을 출력
[user@host ~]$ rpm -ql dnf
/usr/bin/dnf
/usr/lib/systemd/system/dnf-makecache.service
/usr/lib/systemd/system/dnf-makecache.timer
/usr/share/bash-completion
/usr/share/bash-completion/completions
/usr/share/bash-completion/completions/dnf
...output omitted...

# rpm -qc: 패키지가 설치한 config 파일만 출력
[user@host ~]$ rpm -qc openssh-clients
/etc/ssh/ssh_config
/etc/ssh/ssh_config.d/50-redhat.conf

# rpm -qd: 패키지가 설치한 설명서 파일만 출력
[user@host ~]$ rpm -qd openssh-clients
/usr/share/man/man1/scp.1.gz
/usr/share/man/man1/sftp.1.gz
/usr/share/man/man1/ssh-add.1.gz
/usr/share/man/man1/ssh-agent.1.gz
...output omitted...

# rpm -q --scripts: 패키지 설치 또는 제거 전후에 실행되는 쉘 스크립트를 출력
[user@host ~]$ rpm -q --scripts openssh-server
preinstall scriptlet (using /bin/sh):
getent group sshd >/dev/null || groupadd -g 74 -r sshd || :
getent passwd sshd >/dev/null || \
  useradd -c "Privilege-separated SSH" -u 74 -g sshd \
  -s /sbin/nologin -r -d /usr/share/empty.sshd sshd 2> /dev/null || :
postinstall scriptlet (using /bin/sh):

if [ $1 -eq 1 ] && [ -x "/usr/lib/systemd/systemd-update-helper" ]; then
    # Initial installation
    /usr/lib/systemd/systemd-update-helper install-system-units sshd.service sshd.socket || :
fi
...output omitted...

# rpm -q --changelog: 패키지에 대한 변경 로그 정보를 출력
[user@host ~]$ rpm -q --changelog audit
* Tue Feb 22 2022 Sergio Correia <scorreia@redhat.com> - 3.0.7-101
- Adjust sample-rules dir permissions
  Resolves: rhbz#2054432 - /usr/share/audit/sample-rules is no longer readable by non-root users

* Tue Jan 25 2022 Sergio Correia <scorreia@redhat.com> - 3.0.7-100
- New upstream release, 3.0.7
  Resolves: rhbz#2019929 - capability=unknown-capability(39) in audit messages
...output omitted...

# rpm -qlp: 로컬 패키지 파일이 설치하는 파일을 출력
[user@host ~]$ rpm -qlp podman-4.0.0-6.el9.x86_64.rpm
/etc/cni/net.d
/etc/cni/net.d/87-podman-bridge.conflist
/usr/bin/podman
...output omitted...
```



### RPM 패키지 설치

```shell
# 로컬 디렉터리에 다운로드한 RPM 패키지를 설치
[root@host ~]# rpm -ivh podman-4.0.0-6.el9.x86_64.rpm
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
        podman-2:4.0.0-6              ################################# [100%]
```



### RPM 패키지 추출

- `rpm2cpio` 명령을 사용하여 패키지를 설치하지 않고 RPM 패키지 파일에서 파일을 추출
- `rpm2cpio` 명령은 RPM 패키지를 `cpio` 아카이브로 변환함
  - cpio 아카이브?
    - 리눅스나 유닉스 시스템에서 파일을 아카이브(묶고 압축)하고 추출하는 데 사용되는 도구이자 파일 형식
    - `copy in, copy out`의 약자로, 파일과 디렉토리를 하나의 아카이브 파일로 묶거나, 반대로 아카이브에서 파일을 추출하는 유틸리티
    - 유닉스 초기 시절부터 사용된 오래된 도구로, `tar`와 비슷한 역할을 함
    - cpio 명령어로 생성된 아카이브는 파일과 디렉토리의 메타데이터(경로, 권한, 소유자 등)와 내용을 하나의 연속된 바이너리 파일로 저장함
    - RPM 패키지 파일의 내부 엔진같은 존재로, RPM을 설치하면 cpio 아카이브를 풀어서 시스템에 파일을 배치하는 구조
- RPM 패키지를 `cpio` 아카이브로 변환하면 `cpio` 명령을 사용하여 파일 목록을 추출할 수 있음
- 표준 입력에서 파일을 추출하려면 `cpio` 명령을 `-i` 옵션과 함께 사용
  - 현재 작업 디렉터리에서 시작하여 필요에 따라 하위 디렉터리를 생성하려면 `-d` 명령을 사용
  - 상세 출력을 원하는 경우 `-v` 옵션을 사용
- 출력 값에 있는 blocks는 cpio의 데이터 블록임
  - cpio는 512 바이트를 1블록으로 계산함(옵션으로 변경 가능)

```shell
# rpm -> cpio archive로 변환 및 현재 위치 기준으로 file 추출
[user@host tmp-extract]$ rpm2cpio httpd-2.4.51-7.el9_0.x86_64.rpm | cpio -idv
./etc/httpd/conf
./etc/httpd/conf.d/autoindex.conf
./etc/httpd/conf.d/userdir.conf
./etc/httpd/conf.d/welcome.conf
./etc/httpd/conf.modules.d
./etc/httpd/conf.modules.d/00-base.conf
./etc/httpd/conf.modules.d/00-dav.conf
./etc/httpd/conf.modules.d/00-mpm.conf
./etc/httpd/conf.modules.d/00-optional.conf
./etc/httpd/conf.modules.d/00-proxy.conf
./etc/httpd/conf.modules.d/00-systemd.conf
./etc/httpd/conf.modules.d/01-cgi.conf
./etc/httpd/conf.modules.d/README
./etc/httpd/conf/httpd.conf
...output omitted...
9774 blocks
[user@host tmp-extract]$ ls -l
total 1552
drwxr-xr-x. 5 user user       55 Feb  3 15:06 etc
-rw-r--r--. 1 user user  1588633 Feb  3 15:06 httpd-2.4.51-7.el9_0.x86_64.rpm
drwxr-xr-x. 3 user user       19 Feb  3 15:06 run
drwxr-xr-x. 7 user user       70 Feb  3 15:06 usr
drwxr-xr-x. 5 user user       41 Feb  3 15:06 var

# 파일 경로를 지정해서, 개별 파일만 추출
[user@host ~]$ rpm2cpio httpd-2.4.51-7.el9_0.x86_64.rpm | cpio -id "*/etc/httpd/conf/httpd.conf"
9774 blocks
[user@host ~]$ ls etc/httpd/conf/
httpd.conf

# rpm2cpio 및 cpio -t 명령을 사용하여 RPM 패키지의 파일을 표시
# -t: 추출 없이 목록만 확인, -v: verbose 상세 출력
[student@servera ~]$ rpm2cpio httpd-2.4.51-7.el9_0.x86_64.rpm | cpio -tv
drwxr-xr-x   1 root     root            0 Mar 21  2022 ./etc/httpd/conf
-rw-r--r--   1 root     root         2893 Mar 21  2022 ./etc/httpd/conf.d/autoindex.conf
-rw-r--r--   1 root     root         1252 Mar 21  2022 ./etc/httpd/conf.d/userdir.conf
-rw-r--r--   1 root     root          653 Mar 21  2022 ./etc/httpd/conf.d/welcome.conf
drwxr-xr-x   1 root     root            0 Mar 21  2022 ./etc/httpd/conf.modules.d
-rw-r--r--   1 root     root         3372 Mar 21  2022 ./etc/httpd/conf.modules.d/00-base.conf
-rw-r--r--   1 root     root          139 Mar 21  2022 ./etc/httpd/conf.modules.d/00-dav.conf
-rw-r--r--   1 root     root          948 Mar 21  2022 ./etc/httpd/conf.modules.d/00-mpm.conf
-rw-r--r--   1 root     root          787 Mar 21  2022 ./etc/httpd/conf.modules.d/00-optional.conf
-rw-r--r--   1 root     root         1073 Mar 21  2022 ./etc/httpd/conf.modules.d/00-proxy.conf
-rw-r--r--   1 root     root           88 Mar 21  2022 ./etc/httpd/conf.modules.d/00-systemd.conf
-rw-r--r--   1 root     root          367 Mar 21  2022 ./etc/httpd/conf.modules.d/01-cgi.conf
-rw-r--r--   1 root     root          496 Mar 21  2022 ./etc/httpd/conf.modules.d/README
-rw-r--r--   1 root     root        12005 Mar 21  2022 ./etc/httpd/conf/httpd.conf
...output omitted...
```

## DNF를 사용하여 소프트웨어 패키지 설치 및 업데이트

`dnf` 명령을 사용하여 소프트웨어 패키지를 찾고 설치 및 업데이트

### DNF를 사용하여 소프트웨어 패키지 관리

- DNF(Dandified YUM)는 Red Hat Enterprise Linux 9에서 YUM 대신 Package Manager로 채택
  - yum은 python2 기반, dnf는 python3 기반
- DNF 명령의 기능은 YUM 명령과 동일함. 
- 호환성을 위해 YUM 명령도 DNF에 대한 심볼릭 링크로 계속 유지

```shell
[user@host ~]$ ls -l /bin/ | grep yum | awk '{print $9 " " $10 " " $11}'
yum -> dnf-3
yum-builddep -> /usr/libexec/dnf-utils
yum-config-manager -> /usr/libexec/dnf-utils
yum-debug-dump -> /usr/libexec/dnf-utils
yum-debug-restore -> /usr/libexec/dnf-utils
yumdownloader -> /usr/libexec/dnf-utils
yum-groups-manager -> /usr/libexec/dnf-utils
```

- `rpm` 명령을 사용하여 패키지를 설치할 수 있지만, `rpm`은 패키지 리포지토리와 연결해 바로 동작하거나 여러 소스의 종속성을 자동으로 해결하도록 설계되어 있지 않음
- `dnf` 명령을 사용하면 소프트웨어 패키지 및 해당 종속성을 설치, 업데이트, 제거하고 해당 정보를 가져올 수 있음

### DNF를 이용해 소프트웨어 찾기

```shell
# dnf list: 설치되어 사용 가능한 패키지 출력
[user@host ~]$ dnf list 'http*'
Available Packages
http-parser.i686               2.9.4-6.el9    rhel-9.0-for-x86_64-appstream-rpms
http-parser.x86_64             2.9.4-6.el9    rhel-9.0-for-x86_64-appstream-rpms
httpcomponents-client.noarch   4.5.13-2.el9   rhel-9.0-for-x86_64-appstream-rpms
httpcomponents-core.noarch     4.4.13-6.el9   rhel-9.0-for-x86_64-appstream-rpms
httpd.x86_64                   2.4.51-5.el9   rhel-9.0-for-x86_64-appstream-rpms
httpd-devel.x86_64             2.4.51-5.el9   rhel-9.0-for-x86_64-appstream-rpms
httpd-filesystem.noarch        2.4.51-5.el9   rhel-9.0-for-x86_64-appstream-rpms
httpd-manual.noarch            2.4.51-5.el9   rhel-9.0-for-x86_64-appstream-rpms
httpd-tools.x86_64             2.4.51-5.el9   rhel-9.0-for-x86_64-appstream-rpms

# dnf search {KEYWORD}: 이름 및 요약 필드에 있는 키워드만으로 패키지 출력
# 이름이나 요약 필드 + 설명 필드에 'web server'가 포함된 패키지 검색을 위해 all 추가 가능
[user@host ~]$ dnf search all 'web server'
================== Summary & Description Matched: web server ===================
nginx.x86_64 : A high performance web server and reverse proxy server
pcp-pmda-weblog.x86_64 : Performance Co-Pilot (PCP) metrics from web server logs
========================= Summary Matched: web server ==========================
libcurl.x86_64 : A library for getting files from web servers
libcurl.i686 : A library for getting files from web servers
======================= Description Matched: web server ========================
freeradius.x86_64 : High-performance and highly configurable free RADIUS server
git-instaweb.noarch : Repository browser in gitweb
http-parser.i686 : HTTP request/response parser for C
http-parser.x86_64 : HTTP request/response parser for C
httpd.x86_64 : Apache HTTP Server
mod_auth_openidc.x86_64 : OpenID Connect auth module for Apache HTTP Server
mod_jk.x86_64 : Tomcat mod_jk connector for Apache
mod_security.x86_64 : Security module for the Apache HTTP Server
varnish.i686 : High-performance HTTP accelerator
varnish.x86_64 : High-performance HTTP accelerator
...output omitted...

# dnf info {PACKAGENAME}: 설치에 필요한 디스크 공간을 포함한 자세한 패키지 정보 출력
[user@host ~]$ dnf info httpd
Available Packages
Name         : httpd
Version      : 2.4.51
Release      : 5.el9
Architecture : x86_64
Size         : 1.5 M
Source       : httpd-2.4.51-5.el9.src.rpm
Repository   : rhel-9.0-for-x86_64-appstream-rpms
Summary      : Apache HTTP Server
URL          : https://httpd.apache.org/
License      : ASL 2.0
Description  : The Apache HTTP Server is a powerful, efficient, and extensible
             : web server.

# dnf provides {PATHNAME}: 지정된 경로 이름과 일치하는 패키지 표시
# 시스템 관리자가 파일의 출처를 확인하거나, 삭제된 파일을 복구하기 위해 어떤 패키지를 설치해야 하는지 알아낼 때 주로 사용
[user@host ~]$ dnf provides /var/www/html
httpd-filesystem-2.4.51-5.el9.noarch : The basic directory layout for the Apache HTTP Server
Repo        : rhel-9.0-for-x86_64-appstream-rpms
Matched from:
Filename    : /var/www/html
```



### DNF를 사용해 소프트웨어 설치 및 제거

```shell
[root@host ~]# dnf install httpd
Dependencies resolved.
================================================================================
 Package          Arch   Version       Repository                          Size
================================================================================
Installing:
 httpd            x86_64 2.4.51-5.el9  rhel-9.0-for-x86_64-appstream-rpms 1.5 M
Installing dependencies:
 apr              x86_64 1.7.0-11.el9  rhel-9.0-for-x86_64-appstream-rpms 127 k
 apr-util         x86_64 1.6.1-20.el9  rhel-9.0-for-x86_64-appstream-rpms  98 k
 apr-util-bdb     x86_64 1.6.1-20.el9  rhel-9.0-for-x86_64-appstream-rpms  15 k
 httpd-filesystem noarch 2.4.51-5.el9  rhel-9.0-for-x86_64-appstream-rpms  17 k
 httpd-tools      x86_64 2.4.51-5.el9  rhel-9.0-for-x86_64-appstream-rpms  88 k
 redhat-logos-httpd
                  noarch 90.4-1.el9    rhel-9.0-for-x86_64-appstream-rpms  18 k
Installing weak dependencies:
 apr-util-openssl x86_64 1.6.1-20.el9  rhel-9.0-for-x86_64-appstream-rpms  17 k
 mod_http2        x86_64 1.15.19-2.el9 rhel-9.0-for-x86_64-appstream-rpms 153 k
 mod_lua          x86_64 2.4.51-5.el9  rhel-9.0-for-x86_64-appstream-rpms  63 k

Transaction Summary
================================================================================
Install  10 Packages

Total download size: 2.1 M
Installed size: 5.9 M
Is this ok [y/N]: y
Downloading Packages:
(1/10): apr-1.7.0-11.el9.x86_64.rpm             6.4 MB/s | 127 kB     00:00
(2/10): apr-util-bdb-1.6.1-20.el9.x86_64.rpm    625 kB/s |  15 kB     00:00
(3/10): apr-util-openssl-1.6.1-20.el9.x86_64.rp 1.9 MB/s |  17 kB     00:00
...output omitted...
Total                                            24 MB/s | 2.1 MB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1
  Installing       : apr-1.7.0-11.el9.x86_64                               1/10
  Installing       : apr-util-bdb-1.6.1-20.el9.x86_64                      2/10
  Installing       : apr-util-openssl-1.6.1-20.el9.x86_64                  3/10
...output omitted...
Installed:
  apr-1.7.0-11.el9.x86_64              apr-util-1.6.1-20.el9.x86_64
  apr-util-bdb-1.6.1-20.el9.x86_64     apr-util-openssl-1.6.1-20.el9.x86_64
...output omitted...
Complete!

# Package name을 지정하지 않으면, 전체 업데이트
[root@host ~]# dnf update

# 설치된 소프트웨어 패키지 제거
[root@host ~]# dnf remove httpd
```

```shell
# 설치되어 있고 사용 가능한 커널 출력
# 커널도 dnf와 같은 패키지매니저로 업데이트 가능
# 커널은 일반 소프트웨어와 달리 기존 버전을 덮어쓰지 않고, 새 버전을 추가로 설치함
[user@host ~]$ dnf list kernel
Installed Packages
kernel.x86_64                       5.14.0-70.el9                        @System

# 현재 실행중인 커널의 버전과 릴리즈 출력
[user@host ~]$ uname -r
5.14.0-70.el9.x86_64

# 현재 실행중인 커널의 버전과 릴리즈 및 추가정보 출력
[user@host ~]$ uname -a
Linux workstation.lab.example.com 5.14.0-70.el9.x86_64 #1 SMP PREEMPT Thu Feb 24 19:11:22 EST 2022 x86_64 x86_64 x86_64 GNU/Linux
```



### DNF를 사용해 소프트웨어 그룹 설치 및 제거

- `dnf` 명령은 두 종류의 패키지 그룹을 설치할 수 있음
- 일반(regular) 그룹: 패키지 모음(collection)
- 환경 그룹: 일반 그룹의 모음(collection)
- 모음(collection)이 제공하는 패키지나 그룹은 아래와 같이 분류될 수 있음
  - mandatory: 그룹이 설치된 경우 설치해야 함
  - default: 그룹이 설치된 경우 일반적으로 설치됨
  - optional: 구체적으로 요청되지 않은 한, 그룹을 설치할 때 설치되지 않음

```shell
# 설치되어 사용 가능한 그룹 출력
[user@host ~]$ dnf group list
Available Environment Groups:
   Server with GUI
   Server
   Minimal Install
...output omitted...
Available Groups:
   Legacy UNIX Compatibility
   Console Internet Tools
   Container Management
...output omitted...

# 그룹에 대한 정보 출력
[user@host ~]$ dnf group info "RPM Development Tools"
Group: RPM Development Tools
 Description: Tools used for building RPMs, such as rpmbuild.
 Mandatory Packages:
   redhat-rpm-config
   rpm-build
 Default Packages:
   rpmdevtools
 Optional Packages:
   rpmlint


# mandatory, default, dependencies 패키지 설치
[root@host ~]# dnf group install "RPM Development Tools"
...output omitted...
Installing Groups:
 RPM Development Tools

Transaction Summary
================================================================================
Install  19 Packages

Total download size: 4.7 M
Installed size: 15 M
Is this ok [y/N]: y
...output omitted...
```



### 트랜잭션 히스토리 보기

- 모든 설치 및 제거 트랜잭션은 `/var/log/dnf.rpm.log` 파일에 기록

```shell
# 트랜잭션 로그 조회
[user@host ~]$ tail -5 /var/log/dnf.rpm.log
2022-03-23T16:46:43-0400 SUBDEBUG Installed: python-srpm-macros-3.9-52.el9.noarch
2022-03-23T16:46:43-0400 SUBDEBUG Installed: redhat-rpm-config-194-1.el9.noarch
2022-03-23T16:46:44-0400 SUBDEBUG Installed: elfutils-0.186-1.el9.x86_64
2022-03-23T16:46:44-0400 SUBDEBUG Installed: rpm-build-4.16.1.3-11.el9.x86_64
2022-03-23T16:46:44-0400 SUBDEBUG Installed: rpmdevtools-9.5-1.el9.noarch

# 설치 및 제거 트랜잭션 요약
[root@host ~]# dnf history
ID     | Command line              | Date and time    | Action(s)      | Altered
--------------------------------------------------------------------------------
     7 | group install RPM Develop | 2022-03-23 16:46 | Install        |   20
     6 | install httpd             | 2022-03-23 16:21 | Install        |   10 EE
     5 | history undo 4            | 2022-03-23 15:04 | Removed        |   20
     4 | group install RPM Develop | 2022-03-23 15:03 | Install        |   20
     3 |                           | 2022-03-04 03:36 | Install        |    5
     2 |                           | 2022-03-04 03:33 | Install        |  767 EE
     1 | -y install patch ansible- | 2022-03-04 03:31 | Install        |   80

# ID 6 명령 이전으로 트랜잭션 되돌리기
[root@host ~]# dnf history undo 6
...output omitted...
Removing:
 apr-util-openssl x86_64 1.6.1-20.el9 @rhel-9.0-for-x86_64-appstream-rpms  24 k
 httpd            x86_64 2.4.51-5.el9 @rhel-9.0-for-x86_64-appstream-rpms 4.7 M
...output omitted...
```



### DNF Command 요약

| 작업:                                              | 명령:                         |
| :------------------------------------------------- | :---------------------------- |
| 설치되어 사용 가능한 패키지를 이름으로 나열합니다. | `dnf list [NAME-PATTERN]`     |
| 설치되어 사용 가능한 그룹을 나열합니다.            | `dnf group list`              |
| 패키지를 키워드로 검색합니다.                      | `dnf search KEYWORD`          |
| 패키지 세부 정보를 표시합니다.                     | `dnf info PACKAGENAME`        |
| 패키지를 설치합니다.                               | `dnf install PACKAGENAME`     |
| 패키지 그룹을 설치합니다.                          | `dnf group install GROUPNAME` |
| 모든 패키지를 업데이트합니다.                      | `dnf update`                  |
| 패키지를 제거합니다.                               | `dnf remove PACKAGENAME`      |
| 트랜잭션 내역을 표시합니다.                        | `dnf history`                 |



### DNF를 사용해 패키지 모듈 스트림 관리

#### BaseOS 및 애플리케이션 스트림

- RHEL9에서는 2개의 주요 소프트웨어 리포지토리 BaseOS, AppStream을 통해 콘텐츠 배포
  - BaseOS
    - RHEL 운영 체제의 핵심 구성 요소를 제공하는 리포지토리
    - 안정적이고 필수적인 소프트웨어 패키지를 포함하며, OS의 기본 기능(커널, 라이브러리, 시스템 도구 등)을 유지하는 데 초점
    - Contents
      - 커널(kernel, kernel-core)
      - 기본 유틸리티(bash, coreutils, glibc)
      - 시스템 관리 도구(systemd, dnf, yum)
      - 네트워킹 도구(iproute, firewalld)
      - 파일 시스템 관련 패키지(httpd-filesystem 등)
  - AppStream
    - 사용자 애플리케이션과 개발 도구를 제공하는 리포지토리로, 더 유연하고 최신 소프트웨어를 설치할 수 있도록 설계
    - BaseOS가 OS의 "기초"라면, AppStream은 그 위에서 실행되는 "애플리케이션"에 초점
    - 동일한 소프트웨어의 서로 다른 버전을 "스트림"으로 제공(예: python 3.9와 python 3.11)
    - Contents
      - 개발 도구(gcc, go, rust)
      - 데이터베이스(postgresql, mariadb)
      - 웹 서버(nginx, apache의 추가 모듈)
      - 언어 런타임(python, nodejs, ruby)
- 2개의 리포지토리는 RHEL 9 시스템의 필수 구성임