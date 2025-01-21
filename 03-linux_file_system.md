# Linux 파일 시스템 계층 구조 개념 설명

![img](https://static.ole.redhat.com/rhls/courses/rh124-9.3/images/files/linuxfs/images/files/hierarchy.svg)

Linux 시스템은 **반전된 단일 트리**로 구성된 파일 시스템에 모든 파일을 저장한다.



## Special directories

| 위치    | 목적                                                         |
| :------ | :----------------------------------------------------------- |
| `/boot` | 부팅 프로세스를 시작하기 위한 파일, linux 커널               |
| `/dev`  | 시스템에서 하드웨어에 액세스하는 데 사용하는 특수 장치 파일<br />파일 시스템에서 공간을 차지하지 않음<br />e.g. dev/vda(첫 번째 가상디스크), dev/vda1~dev/vda4(첫 번째 가상디스크의 파티션) |
| `/etc`  | 시스템별 구성 파일                                           |
| `/home` | 일반 사용자가 데이터와 구성 파일을 저장하는 홈 디렉터리      |
| `/root` | 관리자 수퍼유저의 홈 디렉터리                                |
| `/run`  | 마지막 부팅 이후 시작된 프로세스의 런타임 데이터. 이 데이터에는 프로세스 ID 파일과 잠금 파일이 포함. 이 디렉터리의 내용은 재부팅 시 다시 생성. |
| `/tmp`  | 어디에서나 쓸 수 있는 임시 파일용 공간. 10일 동안 액세스, 변경 또는 수정되지 않은 파일은 이 디렉터리에서 자동으로 삭제. `/var/tmp` 디렉터리도 임시 디렉터리로, 30일 이상 액세스, 변경 또는 수정되지 않은 파일은 자동으로 삭제. |
| `/usr`  | 설치된 소프트웨어, 공유 라이브러리, 포함 파일 및 읽기 전용 프로그램 데이터. `/usr` 디렉터리의 중요한 하위 디렉터리에는 다음 명령이 포함.<br />`/usr/bin`: 사용자 명령(바이너리)<br />`/usr/sbin`: 시스템 관리 명령(root 사용자만 사용할 수 있는 바이너리)<br />`/usr/local`: 로컬 사용자 지정 소프트웨어 |
| `/var`  | 재부팅 후에도 유지되는 시스템별 가변 데이터. 동적으로 변경되는 파일(예: 데이터베이스, 캐시 디렉터리, 로그 파일, 프린터로 전송된 문서, 웹 사이트 콘텐츠) 관리 |

구체적인 special 디렉터리 구성은 RHEL 버전에 따라 조금씩 다를 수 있다.



## 이름으로 파일 지정

절대 경로와 상대 경로를 이해하고, 작업 디렉터리의 변경, 내용 조회 등을 다룸

### 절대 경로

- 절대 경로는 파일 시스템 계층 구조에서 파일의 정확한 위치를 지정하는 정규화된 이름
-  루트(`/`) 디렉터리에서 시작되며 특정 파일에 도달하기 위해 지나야 하는 각 하위 디렉터리를 포함
- 슬래시(`/`)가 첫 번째 문자로 오는 파일 이름은 절대 경로 이름이라는 간단한 규칙에 따라 인식
- e.g. /var/log/messages

### 현재 작업 디렉터리 및 상대 경로

- 상대 경로는 고유한 위치를 식별하며 작업 디렉터리에서 해당 위치에 도달하는 데 필요한 경로만 지정
- 상대 경로 이름은 슬래시 이외의 문자 가 첫 번째 문자로 사용된 경로 이름이 상대 경로 이름이라는 규칙을 따름
- e.g. /var 위치가 현재 작업 디렉터리일 때, log/messages

### 파일 시스템의 경로 탐색

- 명령어
  - $ pwd
    - 해당 쉘에 대한 현재 작업 디렉터리의 전체 경로 이름을 표시
  - $ ls
    - 지정된 디렉터리(지정된 디렉터리가 없는 경우에는 현재 작업 디렉터리)에 대한 디렉터리 내용을 표시
    - 자주 사용하는 옵션들
      - `-l`(라인 형식)
      - `-a`(*숨겨진* 파일을 포함한 모든 파일)
      -  **`-R`(모든 하위 디렉터리의 내용을 포함하기 위한 재귀 옵션)**
  - $ cd
    - 쉘의 현재 작업 디렉터리를 변경
    - 명령의 인수를 지정하지 않으면 홈 디렉터리로 변경
    - **$ cd -d**
      - **이전에 있었던 이전 디렉터리로 변경**
  - $ touch
    - 파일의 수정 없이 타임스탬프를 현재 날짜와 시간으로 업데이트
    - 빈 파일 생성에 유용함



## CLI Tool을 이용한 파일 관리

파일과 디렉터리를 생성, 복사, 이동 및 제거

### 디렉터리 생성

- $ mkdir {생성할 디렉터리 경로}
  - **$ mkdir {생성할 디렉터리 경로 a} {생성할 디렉터리 경로 b} {생성할 디렉터리 경로 c}**
    - **command 한 번에 여러 디렉터리 생성 가능**
  - $ mkdir -p {생성할 디렉터리 경로}
    - 생성할 디렉터리 경로 중간에 존재하지 않는 디렉터리가 있으면 자동 생성

### 파일 및 디렉터리 복사

- $ cp {복사할 파일 경로} {복사해서 저장할 파일 경로}
  - **$ cp {복사할 파일 a 경로} {복사할 파일 b 경로} {복사할 파일 c 경로} {복사해서 저장할 파일 경로}**
    - **command 한 번에 여러 파일 복사 가능**
  - $ cp -r {복사할 디렉터리 경로} {복사해서 저장할 디렉터리 경로}

### 파일 및 디렉터리 이동

- $ mv {before path a} {after path b}
  - before path a를 after path b로 이동하거나, 이름 변경
  - -v 옵션을 사용하여, 작업내용 출력 가능

### 파일 및 디렉터리 제거

- $ rm {제거할 파일 경로}
  - $ rm -r {제거할 디렉터리 경로}



## 파일 간 링크 만들기

하드 링크와 심볼릭(또는 "소프트") 링크 이해

### 하드 링크

```
[user@host ~]$ pwd
/home/user
[user@host ~]$ ls -l newfile.txt
-rw-r--r--. 1 user user 0 Mar 11 19:19 newfile.txt
```

- $ ls -l 명령으로 하드링크 개수 확인 가능
- $ ln {기존 파일} {생성할 링크 파일} 명령으로 기존 파일을 가리키는 하드 링크(다른 파일) 생성 가능
- $ ls -li 명령으로 각 파일의 inode 출력 가능

```
[user@host ~]$ ln newfile.txt /tmp/newfile-hlink2.txt
[user@host ~]$ ls -l newfile.txt /tmp/newfile-hlink2.txt
-rw-rw-r--. 2 user user 12 Mar 11 19:19 newfile.txt
-rw-rw-r--. 2 user user 12 Mar 11 19:19 /tmp/newfile-hlink2.txt

[user@host ~]$ ls -il newfile.txt /tmp/newfile-hlink2.txt
8924107 -rw-rw-r--. 2 user user 12 Mar 11 19:19 newfile.txt
8924107 -rw-rw-r--. 2 user user 12 Mar 11 19:19 /tmp/newfile-hlink2.txt
```

#### 하드 링크의 한계

- 하드 링크는 일반 파일에서만 사용 가능
- 하드 링크는 두 파일이 동일한 파일 시스템에 있는 경우에만 사용 가능

### 소프트 링크(심볼릭 링크)

심볼릭 링크는 일반 파일이 아니라 기존 파일 또는 디렉터리를 가리키는 특별한 유형의 파일

- $ ln 명령어에 `-s` 옵션을 붙여 사용
- 심볼릭 링크는 서로 다른 파일 시스템에 있는 두 개의 파일을 연결 가능
- 심볼릭 링크는 일반 파일뿐 아니라 디렉터리나 특수 파일을 가리킬 수 있음
- 하드 링크와 다르게 원본 파일이 삭제된 경우 찾지 못해 오류 발생

```
[user@host ~]$ ln -s /home/user/newfile-link2.txt /tmp/newfile-symlink.txt
[user@host ~]$ ls -l newfile-link2.txt /tmp/newfile-symlink.txt
-rw-rw-r--. 1 user user 12 Mar 11 19:19 newfile-link2.txt
lrwxrwxrwx. 1 user user 11 Mar 11 20:59 /tmp/newfile-symlink.txt -> /home/user/newfile-link2.txt
[user@host ~]$ cat /tmp/newfile-symlink.txt
Symbolic Hello World
```

```
[user@host ~]$ rm -f newfile-link2.txt
[user@host ~]$ ls -l /tmp/newfile-symlink.txt
lrwxrwxrwx. 1 user user 11 Mar 11 20:59 /tmp/newfile-symlink.txt -> /home/user/newfile-link2.txt
[user@host ~]$ cat /tmp/newfile-symlink.txt
cat: /tmp/newfile-symlink.txt: No such file or directory
```



## Shell extension을 활용한 파일 이름 일치

Bash shell의 패턴 일치 기능을 효율적으로 실행하기

| 패턴        | 일치                                                         |
| :---------- | :----------------------------------------------------------- |
| *           | 0개 이상의 문자로 이루어진 모든 문자열                       |
| ?           | 모든 단일 문자                                               |
| [*abc…*]    | 대괄호에 포함된 한 문자                                      |
| [!*abc…*]   | 대괄호에 포함되지 *않은* 한 문자                             |
| [^*abc…*]   | 대괄호에 포함되지 *않은* 한 문자                             |
| [[:alpha:]] | 알파벳 문자                                                  |
| [[:lower:]] | 소문자                                                       |
| [[:upper:]] | 대문자                                                       |
| [[:alnum:]] | 알파벳 문자 또는 숫자                                        |
| [[:punct:]] | 공백 또는 영숫자가 아닌 출력 가능한 문자                     |
| [[:digit:]] | 0에서 9 사이의 한 자리 숫자                                  |
| [[:space:]] | 탭, 줄바꿈, 캐리지 리턴, 용지 공급 또는 공백을 포함할 수 있는 단일 공백 문자 |



### 중괄호{} 확장

```
[user@host glob]$ echo {Sunday,Monday,Tuesday,Wednesday}.log
Sunday.log Monday.log Tuesday.log Wednesday.log
[user@host glob]$ echo file{1..3}.txt
file1.txt file2.txt file3.txt
[user@host glob]$ echo file{a..c}.txt
filea.txt fileb.txt filec.txt
[user@host glob]$ echo file{a,b}{1,2}.txt
filea1.txt filea2.txt fileb1.txt fileb2.txt
[user@host glob]$ echo file{a{1,2},b,c}.txt
filea1.txt filea2.txt fileb.txt filec.txt
```

중괄호 확장을 실제로 사용하면 여러 개의 파일이나 디렉터리를 생성 가능

