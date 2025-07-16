# SeLinux 포트 레이블 지정 제어

## SELinux 포트 레이블 지정

- SELinux는 파일 컨텍스트 및 프로세스 유형 레이블 지정 외에도 SELinux 컨텍스트를 사용하여 네트워크 포트에 레이블을 지정함
- SELinux는 네트워크 포트에 레이블을 지정하고 서비스의 타겟 정책에 규칙을 포함하여 네트워크 액세스를 제어함
- 예시
  - SSH 타겟 정책에는 `ssh_port_t` 포트 컨텍스트 레이블이 있는 `22/TCP` 포트가 포함
  - HTTP 정책에서 기본 `80/TCP` 및 `443/TCP` 포트는 `http_port_t` 포트 컨텍스트 레이블 사용
- 타겟 프로세스에서 수신 대기를 위해 포트를 열려고 하면 프로세스와 컨텍스트를 바인딩할 수 있도록 SELinux가 정책에 항목이 포함되어 있는지 확인함
  -  SElinux는 악성 서비스가 다른 합법적 네트워크 서비스에 사용되는 포트를 인계받지 못하도록 차단할 수 있음

## SELinux 포트 레이블 지정 관리

- 새 포트 레이블을 할당하거나, 포트 레이블을 제거/수정/조회하려면 `semanage` 명령을 사용

### 포트 레이블 나열

```shell
# grep으로 서비스의 port 조회
[root@host ~]# grep gopher /etc/services
gopher          70/tcp                          # Internet Gopher
gopher          70/udp

# semanage 명령으로 포트 레이블 할당을 나열
[root@host ~]# semanage port -l
...output omitted...
http_cache_port_t       tcp   8080, 8118, 8123, 10001-10010
http_cache_port_t       udp   3130
http_port_t             tcp   80, 81, 443, 488, 8008, 8009, 8443, 9000
...output omitted...

# SELinux 포트 레이블 할당 조회한 결과에서 서비스 이름으로 필터링
[root@host ~]# semanage port -l | grep ftp
ftp_data_port_t                tcp      20
ftp_port_t                     tcp      21, 989, 990
ftp_port_t                     udp      989, 990
tftp_port_t                    udp      69

# SELinux 포트 레이블 할당 조회한 결과에서 포트 번호를 사용해 필터링
[root@host ~]# semanage port -l | grep -w 70
gopher_port_t                  tcp      70
gopher_port_t                  udp      70
```



### 포트 바인딩 관리

- 기존 포트 컨텍스트 레이블을 사용하여 새 포트에 레이블을 지정할 수 있음
  - options
    - -a: 새 포트 레이블 추가
    - -t: 타겟 레이블
    - -p: 프로토콜

```shell
[root@host~]# semanage port -a -t gopher_port_t -p tcp 71
```

- default 정책에 대한 로컬 변경 사항을 보려면 -C 옵션 사용 가능함

```shell
[root@host~]# semanage port -l -C
SELinux Port Type              Proto    Port Number

gopher_port_t                  tcp      71
```

- 포트 레이블 삭제, 변경

```shell
# -d 옵션과 함께 포트 레이블을 삭제
[root@host ~]# semanage port -d -t gopher_port_t -p tcp 71

#  포트 71/TCP 를 gopher_port_t 에서 http_port_t로 수정
#  -m: modify (이미 등록된 포트의 레이블 변경이 필요할 때)
#  만약 위에서 -d 옵션 명령어를 입력해 삭제된 상태이면 오류 발생함 
[root@server ~]# semanage port -m -t http_port_t -p tcp 71

# 로컬 변경사항 조회
# 위에서 gopher_port_t에서 71번 포트를 제거했으므로 http_port_t의 71번 port만 출력
[root@server ~]# semanage port -l -C
SELinux Port Type              Proto    Port Number

http_port_t                    tcp      71

# http 관련 포트 레이블 출력
[root@server ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      71, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
```

