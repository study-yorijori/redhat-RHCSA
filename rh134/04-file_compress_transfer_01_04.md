# 파일 보관 및 전송
## 압축된 tar 아카이브 관리

- `tar`를 사용하여 파일 및 디렉터리를 압축된 파일로 보관하고 기존 `tar` 아카이브의 내용을 추출함



### 명령줄에서 아카이브 생성

- *아카이브*는 여러 파일을 포함하는 단일 일반 파일 또는 장치 파일
- 장치 파일은 테이프 드라이브, 플래시 드라이브 또는 기타 이동식 미디어일 수 있음
-  일반 파일을 사용하는 경우 보관은 `zip` 유틸리티 및 대부분의 운영 체제에서 널리 사용되는 유사 변형과 비슷함
-  `tar` 유틸리티는 아카이브를 생성, 관리 및 추출하는 일반적인 명령
-  `tar` 명령을 사용하여 여러 파일을 단일 아카이브 파일로 수집함
- *tar 아카이브* 는 개별 파일을 추출할 수 있도록 인덱스가 있는 파일 메타데이터 및 데이터의 구조적 시퀀스
- 파일을 생성하는 동안 지원되는 압축 알고리즘 중 하나를 사용하여 압축할 수 있음
- `tar` 명령은 추출하지 않고 아카이브 내용을 표시할 수 있으며, 압축된 아카이브와 압축되지 않은 아카이브에서 모두 원본 파일을 직접 추출할 수 있음



#### tar 유틸리티의 옵션

- tar 작업을 수행하려면 하기 명령 작업 중 하나가 필요함

  - -c or --create:  아카이브 파일을 생성함
  - -t or --list: 아카이브 내용을 표시함
  - -x or --extract: 아카이브를 추출함

- 자주 포함되는 tar 명령의 일반 옵션

  - -v or --verbose: tar 작업 중에 보관되거나 추출되는 파일 표시함

  - -f or --file: 생성하거나 열려는 아카이브 파일 명시

  - -p or --preserve-permissions: 추출할 때 원래의 파일 권한을 보존함

  - --xattrs: 확장된 속성 지원을 활성화하고 확장된 파일 속성을 저장함

    - 리눅스 파일 시스템에는 기본적으로 소유자, 그룹, 권한 등 메타데이터가 있음
    - 여기에 더해, 확장 속성(xattr) 이란 파일이나 디렉터리에 자유롭게 붙일 수 있는 추가적인 키–값 쌍(name–value pair)임
      - user.* : 사용자 정의 속성 (예: `user.comment`, `user.mime_type` 등)
      - system.posix_acl_\* : POSIX ACL(접근 제어 목록)
      - security.selinux : SELinux 보안 컨텍스트
    - 이 속성들은 `ls`나 `stat`으로 보이지 않지만, `getfattr`/`setfattr` 명령으로 읽고 쓸 수 있음

  - --selinux: SELinux 컨텍스트 지원을 활성화하고 SELinux 파일 컨텍스트를 저장함

    - SELinux?

      -  Security-Enhanced Linux의 약자로, 미국 NSA 등에서 개발한 리눅스 커널 보안 모듈

      - 전통적인 DAC(Discretionary Access Control, 소유자·그룹 기반 권한) 외에 MAC(Mandatory Access Control) 을 도입하여, 프로세스와 파일 등에 `보안 컨텍스트(security context)`라는 추가적인 레이블(label)을 달고 엄격히 권한을 통제
        ```
        user_u:object_r:admin_home_t:s0
        │      │        │             └─ 레벨(level)
        │      │        └─ 타입(type, 프로세스/파일의 역할을 식별)
        │      └─ 역할(role, 사용자 트랜잭션 범위)
        └─ SELinux 사용자(user_u)
        ```

      - `tar --selinux` 를 주면, SELinux 파일 컨텍스트(`security.selinux` xattr)를 아카이브에 함께 저장함

- 알고리즘을 선택하는 데 사용되는 tar 명령 압축 옵션

  - -a or --auto-compress: 아카이브 접미사를 사용하여 사용할 알고리즘을 결정함
  - -z or --gzip: `gzip` 압축 알고리즘을 사용하므로 `.tar.gz` 접미사가 생성됨
  - -j or --bzip2: `bzip2` 압축 알고리즘을 사용하므로 `.tar.bz2` 접미사가 생성됨
  - -J or --xz: `xz` 압축 알고리즘을 사용하므로 `.tar.xz` 접미사가 생성됨



#### 아카이브 생성

```shell
# myapp1.log, myapp2.log, myapp3.log 세 개의 파일을 묶은 mybackup.tar 하나의 아카이브 파일 생성
[user@host ~]$ tar -cf mybackup.tar myapp1.log myapp2.log myapp3.log
[user@host ~]$ ls mybackup.tar
mybackup.tar
```

```shell
# /etc 디렉터리의 모든 내용을 묶은 /root/etc-backup.tar 하나의 아카이브 파일 생성
[root@host ~]# tar -cf /root/etc-backup.tar /etc
tar: Removing leading `/' from member names
```

- root 경로의 디렉터리를 압축할때는 선행되는 슬래시를 제거함
  - 압축을 해제할 때, 원본을 덮어쓰는 현상을 방지하기 위함

#### 아카이브 내용 표시

```shell
# 아카이브 파일의 내용 출력, 선행 슬래시는 제거됨
[root@host ~]# tar -tf /root/etc.tar
etc/
etc/fstab
etc/crypttab
etc/mtab
...output omitted...
```



#### 아카이브 내용 추출

```shell
[root@host ~]# mkdir /root/etcbackup
[root@host ~]# cd /root/etcbackup
# 아카이브 파일의 내용 출력
[root@host etcbackup]# tar -tf /root/etc.tar
etc/
etc/fstab
etc/crypttab
etc/mtab
...output omitted...
# 아카이브 파일의 내용 추출
[root@host etcbackup]# tar -xf /root/etc.tar
```

### 압축된 아카이브 생성

- **`gzip`** 압축 - 가장 빠른 이전 방법으로, 여러 플랫폼에서 광범위하게 사용할 수 있습니다.
- **`bzip2`** 압축 - 더 작은 아카이브를 생성하지만 `gzip`보다 광범위하게 사용되지 않습니다.
- **`xz`** 압축 - 최신 방법으로, 사용 가능한 방법 중 최적 압축 비율을 제공합니다.

```shell
[root@host ~]# tar -czf /root/etcbackup.tar.gz /etc
tar: Removing leading `/' from member names

[root@host ~]$ tar -cjf /root/logbackup.tar.bz2 /var/log
tar: Removing leading `/' from member names

[root@host ~]$ tar -cJf /root/sshconfig.tar.xz /etc/ssh
tar: Removing leading `/' from member names
```



#### 압축된 아카이브 내용 추출

```shell
# gzip 압축을 나타내는 -z 옵션으로 압축 해제 시도 실패
# tar -xJf /root/etcbackup.tar.xz 사용해야 함
[root@host ~]# tar -xzf /root/etcbackup.tar.xz

gzip: stdin: not in gzip format
tar: Child returned status 1
tar: Error is not recoverable: exiting now
```



## 시스템간 안전한 파일 전송

- SSH를 사용하여 원격 시스템에 파일을 안전하게 전송하거나 원격 시스템의 파일을 전송



### 보안 파일 전송 프로그램을 사용해 원격 파일 전송

- `OpenSSH` 제품군은 원격 시스템에서 쉘 명령을 안전하게 실행함
  - OpenSSH:  “SSH(Secure Shell)” 프로토콜의 **오픈 소스 구현**
- *SFTP(보안 파일 전송 프로그램)* 를 사용하여 대화형으로 SSH 서버에 업로드하거나 파일을 다운로드함
  - `OpenSSH` 제품군의 일부
- `sftp` 명령이 포함된 세션은 보안 인증 메커니즘 및 SSH 서버와의 암호화된 데이터 전송을 사용함

```shell
[user@host ~]$ sftp remoteuser@remotehost
remoteuser@remotehost's password: password
Connected to remotehost.
sftp>

sftp> help
Available commands:
bye                                Quit sftp
cd path                            Change remote directory to 'path'
chgrp [-h] grp path                Change group of file 'path' to 'grp'
chmod [-h] mode path               Change permissions of file 'path' to 'mode'
chown [-h] own path                Change owner of file 'path' to 'own'
...output omitted...

# 원격 호스트의 pwd
sftp> pwd
Remote working directory: /home/remoteuser
# 로컬 호스트의 pwd
sftp> lpwd
Local working directory: /home/user


sftp> mkdir hostbackup
sftp> cd hostbackup

# 로컬 호스트의 /etc/hosts 파일들을 원격 호스트 현재 경로에 복사
sftp> put /etc/hosts
Uploading /etc/hosts to /home/remoteuser/hostbackup/hosts
/etc/hosts                                 100%  227     0.2KB/s   00:00

# 로컬 호스트의 directory를 재귀적으로 원격 호스트 현재 경로에 복사
sftp> put -r directory
Uploading directory/ to /home/remoteuser/directory
Entering directory/
file1                                      100%    0     0.0KB/s   00:00
file2                                      100%    0     0.0KB/s   00:00
sftp> ls -l
drwxr-xr-x    2 student  student        32 Mar 21 07:51 directory

# 원격 호스트의 /etc/yum.conf를 로컬 호스트의 현재 디렉터리로 다운로드
sftp> get /etc/yum.conf
Fetching /etc/yum.conf to yum.conf
/etc/yum.conf                              100%  813     0.8KB/s   00:00
sftp> exit
[user@host ~]$

# 대화형 세션을 열지 않고 단일 명령줄에서 sftp를 사용해서 원격 파일을 가져오기
[user@host ~]$ sftp remoteuser@remotehost:/home/remoteuser/remotefile
Connected to remotehost.
Fetching /home/remoteuser/remotefile to remotefile
remotefile                                                       100%    7    15.7KB/s   00:00
```

