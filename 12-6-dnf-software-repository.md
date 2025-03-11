## DNF를 사용하여 소프트웨어 패키지 설치 및 업데이트
### 소프트웨어 찾기
- `dnf list {package name}`
  - 사용 가능한 패키지 정보 표시 
```bash
[user@host ~]$ dnf list 'http*'
Available Packages
http-parser.i686               2.9.4-6.el9    rhel-9.0-for-x86_64-appstream-rpms
http-parser.x86_64             2.9.4-6.el9    rhel-9.0-for-x86_64-appstream-rpms
httpcomponents-client.noarch   4.5.13-2.el9   rhel-9.0-for-x86_64-appstream-rpms
``` 
- `dnf search {keyword}`
  - 이름이 확실하지 않을 때 
  - default로는 package name과 summaries만 찾고 실패하면 url이나 설명도 찾는다
  - 모든 메타데이터에서 처음부터 강제로 검색하기 원하면 `all` 명령어 사용 가능 
  - 이름 및 요약 필드에 있는 키워드로 패키지 표시
```bash
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
```
- `dnf info {packageName}`
  - 설치에 필요한 디스크 공간을 포함해 자세한 정보를 반환
```bash
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
```
- `dnf provides PATHNAME`
  - 지정된 경로 or 기능과 와 일치하는 패키지를 표시
```bash
[user@host ~]$ dnf provides /var/www/html
httpd-filesystem-2.4.51-5.el9.noarch : The basic directory layout for the Apache HTTP Server
Repo        : rhel-9.0-for-x86_64-appstream-rpms
Matched from:
Filename    : /var/www/html
```

### 소프트웨어 설치 및 제거
- `dnf install {packageName}`
```bash
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
```
- `dnf update {packageName}`
  - 종속성을 포함하여 지정된 패키지의 이후 버전을 가져와서 설치
  - **PACKAGENAME을 지정하지 않으면 관련 업데이트가 모두 설치**
- `dnf remove {packageName}`

### 소프트웨어 그룹 설치 및 제거
- 그룹: 소프트웨어 컬렉션
- 환경 그룹과 정규 그룹으로 구성
- 환경 그룹은 정규 그룹의 그룹
```bash
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
```
- 그룹 정보 조회
```bash
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
```
- 그룹 설치
```bash
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
- 대표적인 그룹
  -  Apache, nginx 등 웹 서버 패키지 설치
`dnf group install "Basic Web Server"`

### 히스토리 보기
- 모든 설치 및 제거 트랜잭션은 `/var/log/dnf.rpm.log` 파일에 기록
- `dnf history` 명령도 사용 가능
  - `Altered`
    - 명령 실행으로 인해 변경된 패키지 수(패키지의 설치(Install), 업데이트(Update), 삭제(Remove) 등을 포함)
    - EE: 추가 상태 표시 명시적/의존적(Extra/Excluded) 설치

```bash
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
```
- `dnf history undo`로 트랜잭션을 되돌릴 수 있음

### DNF 모듈
- 소프트웨어의 특정 버전이나 설정을 모듈로 묶어서 관리
- 예를 들어, Node.js, Python, PostgreSQL 같은 소프트웨어는 여러 버전이 동시에 제공됨
- 사용자는 원하는 버전(stream)이나 설정(profile)을 선택해서 설치 가능
- 모듈 목록 확인
```bash
dnf module list
Name           Stream       Profiles         Summary
nodejs         14           default          Javascript runtime
nodejs         16           default          Javascript runtime
python38       3.8          default          Python 3.8 programming language
```
- 특정 모듈 활성화 (Enable)
  - 활성화 후 해당 버전만 설치 가능
`dnf module enable nodejs:16`
- `dnf module install nodejs:16` : 모듈 설치

### 패키지 설치와 모듈 설치 차이?
- 패키지 단위 설치 vs 모듈 단위 설치
- `dnf install`로 Node.js 설치
  - 저장소의 기본 최신 버전 설치
- `dnf module install nodejs:16`
  - nodejs의 특정 버전(stream) 선택 설치
  - 특정 프로파일 설치가능
    - 모듈에는 다양한 프로파일이 존재
    - 각 프로파일은 특정 설정 및 구성에 맞게 패키지 집합을 정의
    - ex. default, dev,...
    - `dnf module install nodejs:16/development`: nodejs 모듈의 16 버전에서 개발 프로파일 설치

## DNF 소프트웨어 리포지토리 활성화
- 신뢰할 수 있는 곳에서 소프트웨어 패키지를 다운받아야 함
- 소프트웨어를 공급하는 곳을 리포지토리라 명칭
- 신뢰성을 보장하기 위해 소프트웨어 공급사에서는 pri/public key를 발급받음
  - private key로 소프트웨어에 사인
  - public key로 사인된 소프트웨어를 검증
  - public key로 검증하는 과정을 gpg key 사용 패키지 서명 확인이라 함
- `--nogpgcheck` 옵션은 누락된 GPG 키를 무시하지만 손상되거나 위조된 패키지가 설치될 수 있

### 리포지토리 조회
- `dnf repository all`
  - 사용 가능한 모든 리포지토리 표시
```bash
[user@host ~]$ dnf repolist all
repo id                                repo name                status
rhel-9.0-for-x86_64-appstream-rpms     RHEL 9.0 AppStream       enabled
rhel-9.0-for-x86_64-baseos-rpms        RHEL 9.0 BaseOS          enabled
```
- 리포지토리 활성화 및 비활성화
  - dnf config-manager 사용
  - 활성화
    - rhel-9-server-debug-rpms 리포지토리가 활성화
  `[user@host ~]$ dnf config-manager --enable rhel-9-server-debug-rpms`

### 리포지토리 추가
- `/etc/yum.repos.d/` 디렉터리에 `.repo` 파일을 생성
- `/etc/dnf/dnf.conf` 파일에 [repository] 섹션을 추가
- `.repo` 파일을 사용하고 추가 리포지토리 구성을 위해 `dnf.conf` 파일을 예약하는 것이 좋음
- dnf 명령은 기본적으로 두 위치를 모두 검색하지만 .repo 파일이 우선
- `.repo` 파일에는 리포지토리 URL, 이름, GPG를 사용하여 패키지 서명을 확인할지 여부, 확인하는 경우 신뢰할 수 있는 GPG 키를 가리키는 URL이 포함되어 있습니다
- `dnf config-manager` 명령으로 추가
  - `/etc/yum.repos.d/` 디렉터리에 표시
```bash
[user@host ~]$ dnf config-manager \
--add-repo="https://dl.fedoraproject.org/pub/epel/9/Everything/x86_64/"
Adding repo from: https://dl.fedoraproject.org/pub/epel/9/Everything/x86_64/
```
```bash
[user@host ~]$ cd /etc/yum.repos.d
[user@host yum.repos.d]$ cat \
dl.fedoraproject.org_pub_epel_9_Everything_x86_64_.repo
[dl.fedoraproject.org_pub_epel_9_Everything_x86_64_]
name=created by dnf config-manager from https://dl.fedoraproject.org/pub/epel/9/Everything/x86_64/
baseurl=https://dl.fedoraproject.org/pub/epel/9/Everything/x86_64/
enabled=1
```
## 예시. 리포지토리 추가 및 접근
- servera에 nodejs를 다운받을 수 있는 새로운 리포지토리 구성 후, serverb에서 해당 리포지토리를 추가하여 nodejs 다운로드
### servera에 새로운 리포지토리 구성
- 리포지토리 폴더 생성 
`mkdir custom_repo`
`cd custom_repo`
- nodeje download
  - 의존성까지 함께 다운로드
```bash
$ dnf download --resolve nodejs
```
- ls
  - 아직 저장소로 볼 수는 없음 그저 rmp package가 저장된 directory
  - 리포지토리를 설명하는 메타데이터 필요
```bash
[root@servera ~]# cd custom_repo/
[root@servera custom_repo]# ls
nodejs-16.20.2-3.el9_2.x86_64.rpm
nodejs-docs-16.20.2-3.el9_2.noarch.rpm
nodejs-full-i18n-16.20.2-3.el9_2.x86_64.rpm
nodejs-libs-16.20.2-3.el9_2.x86_64.rpm
npm-8.19.4-1.16.20.2.3.el9_2.x86_64.rpm
```
- 저장소 생성 도구 설치
`dnf install -y createrepo`
- 현재 폴더를 레포로 생성
```bash
[root@servera custom_repo]# createrepo .
Directory walk started
Directory walk done - 5 packages
Temporary output repo path: ./repodata/
Preparing sqlite DBs
Pool started (with 5 workers)
Pool finished

[root@servera custom_repo]# ls
nodejs-16.20.2-3.el9_2.x86_64.rpm
nodejs-docs-16.20.2-3.el9_2.noarch.rpm
nodejs-full-i18n-16.20.2-3.el9_2.x86_64.rpm
nodejs-libs-16.20.2-3.el9_2.x86_64.rpm
npm-8.19.4-1.16.20.2.3.el9_2.x86_64.rpm
repodata
```
- repodata 확인
```bash
[root@servera custom_repo]# ls repodata/
2d249c7-232082491ed5637ad1484a35448a91355439fb806d33e8ed5952da3-filelists.xml.gz
4094953fbfa82a548bb56bc19f37afbddddf342i3calcfc94d34d180fc8198b-filelists.sqlite.bz2
43105c17bec3b29885389c055a158b95aaf44f9eeb9ef1f2dd25b8793e5d1a2a-primary.xml.gz
47775b606a0fa3103366454883c561c314841d1404c067889ad39069c6e673755-other.sqlite.bz2
5346f1c4e7d4a4fa9195e6b935d5cd4b7338169c7da8530969a9bae3fbf9801c-primary.sqlite.bz2
b533fe324ef12ad9106d4181d1b3c5a3fb957686d1c9b799b752add4eb9e71dd-other.xml.gz
```

## serverb에 새로운 리포지토리 활성화
- 현재 리포지토리 확인
```bash
[root@serverb ~]# dnf repolist
repo id                                      repo name
rhel-9.3-for-x86_64-appstream-rpms          Red Hat Enterprise Linux 9.3 AppStream (dvd)
rhel-9.3-for-x86_64-baseos-rpms             Red Hat Enterprise Linux 9.3 BaseOS (dvd)

[root@serverb ~]# ls -l /etc/yum.repos.d/
total 8
-rw-r--r--. 1 root root  358 Nov  9  2023 redhat.repo
-rw-r--r--. 1 root root  366 Nov  9  2023 rhel_dvd.repo

[root@serverb ~]# cat /etc/yum.repos.d/rhel_dvd.repo
[rhel-9.3-for-x86_64-baseos-rpms]
baseurl = http://content.example.com/rhel9.3/x86_64/dvd/BaseOS
enabled = true
gpgcheck = false
name = Red Hat Enterprise Linux 9.3 BaseOS (dvd)

[rhel-9.3-for-x86_64-appstream-rpms]
baseurl = http://content.example.com/rhel9.3/x86_64/dvd/AppStream
enabled = true
gpgcheck = false
name = Red Hat Enterprise Linux 9.3 AppStream (dvd)
```
- 기존 리포지토리 백업
  - `.repo`확장자만 인지하기 때문에 백업한 리포지토리를 인지할 수 없음
```bash
[root@serverb ~]# rename .repo .back /etc/yum.repos.d/*
[root@serverb ~]# ls -l /etc/yum.repos.d/
total 8
-rw-r--r--. 1 root root 358 Nov  9  2023 redhat.back
-rw-r--r--. 1 root root 366 Nov  9  2023 rhel_dvd.back
[root@serverb ~]# dnf repolist
No repositories available
```
- 새로운 리포지토리 추가
```bash
[root@serverb ~]# dnf config-manager --add-repo="http://servera.lab.example.com/custom_repo"
Adding repo from: http://servera.lab.example.com/custom_repo
[root@serverb ~]# dnf repolist
repo id                              repo name
servera.lab.example.com_custom_repo  created by dnf config-manager from http://servera.lab.example.com/custom_repo
[root@serverb ~]# cat /etc/yum.repos.d/servera.lab.example.com_custom_repo.repo
[servera.lab.example.com_custom_repo]
name=created by dnf config-manager from http://servera.lab.example.com/custom_repo
baseurl=http://servera.lab.example.com/custom_repo
enabled=1
[root@serverb ~]#
```
- nodejs 다운로드
  - 추가된 리포지토리에서 받아온다
```bash
[root@serverb ~]# yum install nodejs
created by dnf config-manager from http://servera.lab.example.com/custom 1.6 MB/s | 3.0 kB     00:00
Dependencies resolved.
================================================================================
Package              Arch        Version                 Repository        Size
================================================================================
Installing:
nodejs               x86_64      1:16.14.0-4.el9_0       servera.lab.example.com_custom_repo   219 k
Installing dependencies:
nodejs-libs          x86_64      1:16.14.0-4.el9_0       servera.lab.example.com_custom_repo    14 M
Installing weak dependencies:
nodejs-docs          noarch      1:16.14.0-4.el9_0       servera.lab.example.com_custom_repo   6.9 M
nodejs-full-i18n     x86_64      1:16.14.0-4.el9_0       servera.lab.example.com_custom_repo   8.1 M
npm                  x86_64      1:8.3.1-1.16.14.0.4.el9_0  servera.lab.example.com_custom_repo   12.2 M

Transaction Summary
================================================================================
```
### 기타 궁금증
- yum 리파지토리가 여러개인데 우선순위를 설정할 수 있나?
  - 가능
  - priority 패키지 설치
  ```bash
  yum install yum-priorities     # RHEL/CentOS 7 이하
  dnf install dnf-plugins-core   # RHEL/CentOS 8 이상
  ```
  - `/etc/yum.repos.d/` 우선순위 설정
  ```bash
  [repo_id]
  name=Repository Name
  baseurl=http://repository.url
  enabled=1
  priority=10
  ```
  - 일반적으로 기본 우선순위는 99이며, 값이 작을수록 우선순위가 높음
  - 중요한 리포지토리에는 낮은 숫자(예: 1~10)를 지정하고, 덜 중요한 리포지토리에는 높은 숫자를 지정