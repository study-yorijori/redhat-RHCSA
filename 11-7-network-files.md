# 네트워크 구성 파일 편집

- 구성 파일을 편집하여 네트워크 구성을 수정함



## 연결 구성 파일

- Red Hat Enterprise Linux 8부터 네트워크 구성은 `/etc/NetworkManager/system-connections/` 디렉터리에 저장
  - `.nmconnection` 확장자 파일로 관리
  - 이전에는 `/etc/sysconfig/network-scripts/` 사용
  - 이전 위치에 저장한 구성도 계속 작동
- 구성 위치는 `ifcfg` 형식 대신 키 파일 형식을 사용
- `/etc/NetworkManager/system-connections/` 디렉터리는 `nmcli con mod {name}` 명령을 통한 변경 사항을 저장함

### 키 파일 형식

- INIT format을 사용해서 네트워크 connection profile을 저장함
- key, value 구성은 섹션(그룹)으로 저장함
- 파일 관리 방식은 nmcli 명령을 사용해서 수정하는 것을 권장하지만, 파일 직접 수정 후에 `nmcli con reload` 명령으로 Network Manager에 반영하는 방법도 있음

| `nmcli con mod`                  | `*.nmconnection` 파일             | 효과                                                         |
| :------------------------------- | :-------------------------------- | :----------------------------------------------------------- |
| `ipv4.method manual`             | `[ipv4]method=manual`             | IPv4 주소를 정적으로 구성합니다.                             |
| `ipv4.method auto`               | `[ipv4]method=auto`               | DHCPv4 서버에서 구성 설정을 찾습니다. DHCPv4의 정보가 있는 경우에만 고정 주소를 표시합니다. |
| `ipv4.addresses 192.0.2.1/24`    | `[ipv4]address1=192.0.2.1/24`     | 고정 IPv4 주소 및 네트워크 접두사를 설정합니다. 연결 주소가 두 개 이상인 경우 `address2` 키는 두 번째 주소를 정의하고, `address3` 키는 세 번째 주소를 정의합니다. |
| `ipv4.gateway 192.0.2.254`       | `[ipv4]gateway=192.0.2.254`       | 기본 게이트웨이를 설정합니다.                                |
| `ipv4.dns 8.8.8.8`               | `[ipv4]dns=8.8.8.8`               | 이 이름 서버를 사용하도록 `/etc/resolv.conf`를 수정합니다.   |
| `ipv4.dns-search example.com`    | `[ipv4]dns-search=example.com`    | `/etc/resolv.conf` 지시문에서 이 도메인을 사용하도록 `search`를 수정합니다. |
| `ipv4.ignore-auto-dns true`      | `[ipv4]ignore-auto-dns=true`      | DHCP 서버에서 DNS 서버 정보를 무시합니다.                    |
| `ipv6.method manual`             | `[ipv6]method=manual`             | IPv6 주소를 정적으로 구성합니다.                             |
| `ipv6.method auto`               | `[ipv6]method=auto`               | 라우터 알림의 SLAAC를 사용하여 네트워크 설정을 구성합니다.   |
| `ipv6.method dhcp`               | `[ipv6]method=dhcp`               | SLAAC가 아닌 DHCPv6를 사용하여 네트워크 설정을 구성합니다.   |
| `ipv6.addresses 2001:db8::a/64`  | `[ipv6]address1=2001:db8::a/64`   | 고정 IPv6 주소 및 네트워크 접두사를 설정합니다. 두 개 이상의 주소를 연결에 사용하는 경우 `address2` 키는 두 번째 주소를 정의하고, `address3` 키는 세 번째 주소를 정의합니다. |
| `ipv6.gateway 2001:db8::1`       | `[ipv6]gateway=2001:db8::1`       | 기본 게이트웨이를 설정합니다.                                |
| `ipv6.dns fde2:6494:1e09:2::d`   | `[ipv6]dns=fde2:6494:1e09:2::d`   | 이 이름 서버를 사용하도록 `/etc/resolv.conf`를 수정합니다. IPv4와 같습니다. |
| `ipv6.dns-search example.com`    | `[ipv6]dns-search=example.com`    | `/etc/resolv.conf` 지시문에서 이 도메인을 사용하도록 `search`를 수정합니다. |
| `ipv6.ignore-auto-dns true`      | `[ipv6]ignore-auto-dns=true`      | DHCP 서버에서 DNS 서버 정보를 무시합니다.                    |
| `connection.autoconnect yes`     | `[connection]autoconnect=true`    | 부팅 시 이 연결을 자동으로 활성화합니다.                     |
| `connection.id ens3`             | `[connection]id=Main eth0`        | 이 연결의 이름입니다.                                        |
| `connection.interface-name ens3` | `[connection]interface-name=ens3` | 연결은 이 이름을 사용하여 네트워크 인터페이스에 바인딩됩니다. |
| `802-3-ethernet.mac-address …`   | `[802-3-ethernet]mac-address=`    | 연결은 이 MAC 주소를 사용하여 네트워크 인터페이스에 바인딩됩니다. |

### 네트워크 구성 수정

```shell
## servera, serverb 각각의 network connection에 새로운 주소를 추가하고, 새로운 주소로 통신을 테스트하는 예제
# servera의 네트워크 인터페이스 조회
[student@servera ~]$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:00:fa:0a brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    altname ens3
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 8942 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:15:d4:0d brd ff:ff:ff:ff:ff:ff
    altname enp0s4
    altname ens4

# servera의 active connection 조회
[student@servera ~]$ nmcli con show --active
NAME         UUID                                  TYPE      DEVICE
System eth0  5fb06bd0-0bb0-7ffb-45f1-d6edd65f3e03  ethernet  eth0
System eth1  9c92fad9-6ecb-3e6c-eb4d-8a47c6f50c04  ethernet  eth1
lo           0e77d074-dd20-465e-87f5-e3974e6f42ab  loopback  lo

# servera의 지정경로에 profile 파일 존재 확인 가능
[student@servera ~]$ ls /etc/NetworkManager/system-connections/
'System eth0.nmconnection'

# servera의 root 사용자로 전환해서, profile 파일 변경
## root 사용자 전환
[student@servera ~]$ sudo -i
[sudo] password for student: student
[root@servera ~]#

## profile 파일 변경, address2 추가
...output omitted...
[ipv4]
address1=172.25.250.10/24,172.25.250.254
address2=10.0.1.1/24
...output omitted...

# NetworkManager 변경사항 반영
[root@servera ~]# nmcli con reload

# 변경사항을 사용해 connection 활성화
[root@servera ~]# nmcli con up "System eth0"
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)

# 새 IP 주소가 할당되었는지 확인
[root@servera ~]# ip -br addr show eth0
eth0             UP             172.25.250.10/24 10.0.1.1/24 fe80::5054:ff:fe00:fa0a/64

# workstation 시스템에 student 사용자로 돌아감
[root@servera ~]# exit
logout
[student@servera ~]$ exit
logout
Connection to servera closed.
[student@workstation ~]$

# serverb 시스템에 student 사용자로 로그인한 후 root 사용자로 전환
[student@workstation ~]$ ssh student@serverb
...output omitted...
[student@serverb ~]$ sudo -i
[sudo] password for student: student
[root@serverb ~]#

# profile 파일 변경, address2 추가
...output omitted...
[ipv4]
address1=172.25.250.11/24,172.25.250.254
address2=10.0.1.2/24
...output omitted...

# NetworkManager 변경사항 반영
[root@serverb ~]# nmcli con reload

# 변경사항을 사용해 connection 활성화
[root@serverb ~]# nmcli con up "System eth0"
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)

# 새 IP 주소가 할당되었는지 확인
[root@serverb ~]# ip -br addr show eth0
eth0             UP             172.25.250.11/24 10.0.1.2/24 fe80::5054:ff:fe00:fa0b/64

# 새 네트워크 주소를 사용해서 servera와 serverb간 connection test
[root@serverb ~]# ping -c3 10.0.1.1
PING 10.0.1.1 (10.0.1.1) 56(84) bytes of data.
64 bytes from 10.0.1.1: icmp_seq=1 ttl=64 time=1.23 ms
64 bytes from 10.0.1.1: icmp_seq=2 ttl=64 time=0.372 ms
64 bytes from 10.0.1.1: icmp_seq=3 ttl=64 time=0.440 ms

--- 10.0.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2040ms
rtt min/avg/max/mdev = 0.372/0.679/1.225/0.387 ms

# servera로 액세스하기 위해, workstation으로 돌아감
[root@serverb ~]# exit
logout
[student@serverb ~]$ exit
logout
Connection to serverb closed.
[student@workstation ~]$

# servera로 액세스 및 serverb의 새 주소 ping 실행
[student@workstation ~]$ ssh student@servera ping -c3 10.0.1.2
PING 10.0.1.2 (10.0.1.2) 56(84) bytes of data.
64 bytes from 10.0.1.2: icmp_seq=1 ttl=64 time=0.726 ms
64 bytes from 10.0.1.2: icmp_seq=2 ttl=64 time=0.349 ms
64 bytes from 10.0.1.2: icmp_seq=3 ttl=64 time=0.342 ms

--- 10.0.1.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2087ms
rtt min/avg/max/mdev = 0.342/0.472/0.726/0.179 ms
```



# Hostname 및 name resolve conf 관리

- 서버의 정적 hostname과 name resolve을 구성하고 결과를 테스트

### 시스템 호스트 이름 업데이트

```shell
# 시스템 hostname 조회
[root@host ~]# hostname
host.example.com

# /etc/hostname 경로에서 hostname을 관리
# 다른 정보들은 각각 지정된 경로에서 따로 저장 및 관리되며 커널에서 직접 관리한다고 함(by GPT)
[root@host ~]# hostnamectl hostname host.example.com
[root@host ~]# hostnamectl status
   Static hostname: host.example.com
         Icon name: computer-vm
           Chassis: vm 🖴
        Machine ID: ace63d6701c2489ab9c0960c0f1afe1d
           Boot ID: 0edf5ba1830c48adbd6babfa08f0b867
    Virtualization: kvm
  Operating System: Red Hat Enterprise Linux 9.0 (Plow)
       CPE OS Name: cpe:/o:redhat:enterprise_linux:9::baseos
            Kernel: Linux 5.14.0-70.13.1.el9_0.x86_64
      Architecture: x86-64
   Hardware Vendor: Red Hat
    Hardware Model: OpenStack Compute
[root@host ~]# cat /etc/hostname
host.example.com
```

- Red Hat Enterprise Linux 7 이상 버전에서 정적 호스트 이름은 `/etc/hostname` 파일에 저장
  - 이전 버전에서는 호스트 이름을 `/etc/sysconfig/network` 파일에 변수로 저장
- 호스트네임을 관리하는 `systemd-hostnamed`이라는 이름의 system daemon도 있음

### 이름 확인 구성

```shell
[root@host ~]# cat /etc/hosts
127.0.0.1       localhost localhost.localdomain localhost4 localhost4.localdomain4
::1             localhost localhost.localdomain localhost6 localhost6.localdomain6
172.25.254.254 classroom.example.com
172.25.254.254 content.example.com
```

- stub resolver가 호스트 이름을 IP 주소로 변환하거나 그 반대로 변환함
- 먼저 `/etc/hosts` 파일을 사용하여 쿼리를 해결하려고 함

```shell
[root@host ~]# cat /etc/resolv.conf
# Generated by NetworkManager
domain example.com
search example.com
nameserver 172.25.254.254
```

- /etc/hosts에서 항목을 찾지 못하면 `/etc/nsswitch.conf` 파일의 구성에 따라 찾아볼 위치를 결정함
- /etc/nsswitch.conf는 DNS 쿼리를 수행하는 방법을 제어함
  - DNS 서버를 통해 도메인을 해석하는 방법 관리
  - search:  짧은 호스트 이름으로 시도할 도메인 이름 목록. 동일한 파일에서 `search` 또는 `domain`을 설정해야 함. 둘 다 설정하면 마지막 항목만 적용됨.
  - nameserver: 쿼리할 이름 서버의 IP 주소. 이름 서버 하나가 다운된 경우 백업을 제공하기 위해 최대 3개의 이름 서버 지시문을 지정할 수 있음.
    - DNS 요청을 보낼 주소, 즉 로컬이 아닌 네트워크를 통해 호스트네임을 해석하기 위해 사용

```shell
# 이전 DNS 설정을 제공된 새 IP 목록으로 교체
[root@host ~]# nmcli con mod {connection id} ipv4.dns {dns ip}

[root@host ~]# nmcli con down {connection id}
[root@host ~]# nmcli con up {connection id}
[root@host ~]# cat /etc/NetworkManager/system-connections/{connection id}
...output omitted...
[ipv4]
...output omitted...
# google public DNS address임
dns=8.8.8.8;
...output omitted...

# nmcli 명령의 ipv4.dns 옵션 앞에 있는 더하기(+) 또는 빼기(-) 문자는 각각 개별 항목을 추가하거나 제거
[root@host ~]# nmcli con mod {connection id} +ipv4.dns {dns ip}
[root@host ~]# nmcli con mod static-ens3 +ipv6.dns 2001:4860:4860::88
```

```shell
# 정방향 DNS record 조회
[root@host ~]# host servera.lab.example.com
servera.lab.example.com has address 172.25.250.10

# 역방향 record 조회
[root@host ~]# host 172.25.250.10
10.250.25.172.in-addr.arpa domain name pointer servera.lab.example.com.
```

```shell
# servera access
[student@workstation ~]$ ssh student@servera
...output omitted...
[student@servera ~]$ sudo -i
[sudo] password for student: student
[root@servera ~]#

# classroom.example.com DNS 설정 조회
[root@servera ~]# host classroom.example.com
classroom.example.com has address 172.25.254.254

# 정적 호스트 등록, classroom, class 추가
[root@servera ~]# vim /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.25.254.254 classroom.example.com classroom class

# host 명령에서는 DNS가 아니기에 조회되지 않음
[root@servera ~]# host class
Host class not found: 3(NXDOMAIN)
# getent 명령은 NSS(Name Service Switch) DB에서 읽어오는데, 파일 기반이기때문에 정상 출력
[root@servera ~]# getent hosts class
172.25.254.254  classroom.example.com classroom class

# 실제 ping 명령에서 동작 확인
[root@servera ~]# ping -c3 class
PING classroom.example.com (172.25.254.254) 56(84) bytes of data.
64 bytes from classroom.example.com (172.25.254.254): icmp_seq=1 ttl=63 time=1.21 ms
64 bytes from classroom.example.com (172.25.254.254): icmp_seq=2 ttl=63 time=0.688 ms
64 bytes from classroom.example.com (172.25.254.254): icmp_seq=3 ttl=63 time=0.559 ms

--- classroom.example.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2046ms
rtt min/avg/max/mdev = 0.559/0.820/1.214/0.283 ms
```

