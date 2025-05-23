# 명령줄 생산성 향상

## 간단한 Bash 스크립트 작성

- Bash 쉘, 쉘 스크립트 및 다양한 RHEL 유틸리티의 고급 기능을 사용해 보다 효율적으로 명령 실행하기

### Bash 쉘 스크립트 생성 및 실행

- Linux 명령을 쉘 스크립트에 결합해 반복되는 실제 문제를 해결할 수 있다
- vim 또는 emacs와 같은 고급 편집기는 Bash 쉘 구문을 이해하기 쉽고 색상 구분으로 강조된 표시 기능 제공 가능
- 좋은 스크립트는 멱등성을 가져야 함

### 명령 인터프리터 지정

- 스크립트의 첫 번째 행은 '#!' 표기법으로 시작함

- 일반적으로 두 문자의 이름인 she-bang 또는 bash-bang을 따서 sharp 또는 hash와 bang이라고 함

- 이 표기법은 파일의 나머지 줄을 처리하기 위한 명령 인터프리터 및 명령 옵션을 나타내는 인터프리터 지시문

- Bash 구문 스크립트 파일의 경우 첫 번째 줄은 다음 지시문과 같음

  ```
  #!/usr/bin/bash
  ```

### Bash 쉘 스크립트 실행

- 쉘 스크립트 파일을 sh 명령으로 실행할 수 있음
- 쉘 스크립트 파일을 일반 명령으로 실행하려면 실행 권한이 있어야 함
- chmod 명령을 사용해 파일 권한을 수정함
  - 필요한 경우 chown 명령을 사용해 특정 사용자 또는 그룹에만 실행 권한을 부여할 수 있음
- Bash 쉘 스크립트가 쉘의 PATH 환경 변수에 나열된 디렉터리에 저장된 경우 컴파일된 명령을 실행하는 것과 유사하게 파일 이름만 사용해 쉘 스크립트를 실행할 수 있음
  - PATH 구문 분석은 검색된 첫 번째 일치 파일을 실행하므로 **기존 명령 이름을 사용하여 스크립트 파일의 이름을 지정하지 않아야 함**
- 스크립트가 PATH 디렉터리에 없는 경우 절대 경로 이름 혹은 상대 경로 이름을 사용해 스크립트를 실행함
  - 절대 경로 이름은 which 명령으로 파일을 쿼리해 확인할 수 있음
  - 상대 경로 이름은 .디렉터리 접두사를 사용해 현재 작업 디렉터리에서부터 시작할 수 있음

### 인용 특수 문자

- Bash 쉘에서 특별한 의미를 갖는 문자와 단어가 있음
  - 특별한 의미가 아닌 리터럴 값으로 사용하려면 스크립트에서 해당 문자를 이스케이프 해야 함
  - 이스케이프는 백슬래시(\\), 작은따옴표('') 또는 큰따옴표("")를 사용해 적용할 수 있음
- 소괄호는 다른 명령을 실행하고 결과 대체함
- 큰따옴표는 쉘 확장을 억제하는데 사용하지만 변수 대체(중괄호 구문)을 사용할 수 있음
  - 작은 따옴표는 포함된 모든 텍스트를 문자 그대로 해석함

```shell
# 소괄호를 사용한 결과 대체
[user@host ~]$ var=$(hostname -s); echo $var
host

# 큰따옴표와 변수 대체의 활용
[user@host ~]$ echo "***** hostname is ${var} *****"
***** hostname is host *****

# 백슬래시 이스케이프로 변수 대체 방지
[user@host ~]$ echo Your username variable is \$USER.
Your username variable is $USER.

# 큰따옴표에서는 변수, 결과 대체 적용
[user@host ~]$ echo "Will variable $var evaluate to $(hostname -s)?"
Will variable host evaluate to host?

# 작은따옴표에서는 문자 그대로 출력
[user@host ~]$ echo 'Will variable $var evaluate to $(hostname -s)?'
Will variable $var evaluate to $(hostname -s)?

# 큰따옴표와 이스케이프 활용
[user@host ~]$ echo "\"Hello, world\""
"Hello, world"

# 작은따옴표는 이스케이프 필요X
[user@host ~]$ echo '"Hello, world"'
"Hello, world"
```



### 쉘 스크립트에서 출력 제공

- echo 명령은 텍스트를 명령의 인수로 전달해서 임의의 텍스트를 표시함
- 출력 리디렉션을 사용해 텍스트를 다른 곳으로 보낼 수 있음

```shell
[user@host ~]$ cat ~/bin/hello
#!/usr/bin/bash

echo "Hello, world"
echo "ERROR: Houston, we have a problem." >&2

# 표준 출력만 출력됨
[user@host ~]$ hello 2> hello.log
Hello, world

# 표준 에러는 hello.log로 저장됨
[user@host ~]$ cat hello.log
ERROR: Houston, we have a problem.
```



# 스크립트의 반복문 및 조건부 구문

## 반복문을 사용하여 명령 반복

```shell
for VARIABLE in LIST; do
COMMAND VARIABLE
done
```

Bash의 for 반복문 구문 형식은 위와 같음

```shell
[user@host ~]$ for HOST in host1 host2 host3; do echo $HOST; done
host1
host2
host3
[user@host ~]$ for HOST in host{1,2,3}; do echo $HOST; done
host1
host2
host3
[user@host ~]$ for HOST in host{1..3}; do echo $HOST; done
host1
host2
host3
[user@host ~]$ for FILE in file{a..c}; do ls $FILE; done
filea
fileb
filec
# rpm -qa | grep kernel: 현재 시스템에 설치된 모든 RPM 패키지 조회해서 kernel 문자열을 가진 패키지 조회
# rpm -q --qf "%{INSTALLTIME}\n" $PACKAGE: 쿼리 결과를 설치 시간 필드로 포맷팅하고 유닉스 타임스탬프로 출력
# date -d 명령어로 유닉스 타임스탬프를 human readable format으로 출력
# $(date -d @$(rpm -q --qf "%{INSTALLTIME}\n" $PACKAGE))에서 @는 date 명령에서 유닉스 타임스탬프값을 다루기 위한 기호
[user@host ~]$ for PACKAGE in $(rpm -qa | grep kernel); \
do echo "$PACKAGE was installed on \
$(date -d @$(rpm -q --qf "%{INSTALLTIME}\n" $PACKAGE))"; done
kernel-tools-libs-5.14.0-70.2.1.el9_0.x86_64 was installed on Thu Mar 24 10:52:40 PM EDT 2022
kernel-tools-5.14.0-70.2.1.el9_0.x86_64 was installed on Thu Mar 24 10:52:40 PM EDT 2022
kernel-core-5.14.0-70.2.1.el9_0.x86_64 was installed on Thu Mar 24 10:52:46 PM EDT 2022
kernel-modules-5.14.0-70.2.1.el9_0.x86_64 was installed on Thu Mar 24 10:52:47 PM EDT 2022
kernel-5.14.0-70.2.1.el9_0.x86_64 was installed on Thu Mar 24 10:53:04 PM EDT 2022

# seq [START] [STEP] [END]: 숫자 시퀀스 출력 명령
[user@host ~]$ for EVEN in $(seq 2 2 10); do echo "$EVEN"; done
2
4
6
8
10
```



## Bash 스크립트 종료 코드

- 스크립트의 모든 내용이 처리되면 스크립트 프로세스가 종료되고 해당 프로세스를 호출한 상위 프로세스로 제어가 전달됨
- 스크립트에 오류 조건이 발생하는 경우는 스크립트가 완료되기 전에 스크립트가 종료됨
- 스크립트를 즉시 종료하고 나머지 스크립트의 처리를 건너뛰려면 exit 명령을 사용함
  - exit 명령은 종료 코드를 나타내는 0과 255 사이의 정수 인수와 함께 사용함
  - 종료 코드 0은 스크립트가 오류 없이 완료되었음을 나타냄
  - $? 변수에서 마지막으로 완료된 명령의 종료 코드를 검색할 수 있음

```shell
[user@host bin]$ cat hello
#!/usr/bin/bash
echo "Hello, world"
exit 0
[user@host bin]$ ./hello
Hello, world
[user@host bin]$ echo $?
0
```



## 문자열 및 디렉터리에 대한 논리 테스트 및 값 비교

- Bash에서는 test 명령을 사용하여 스크립트의 무결성을 확인할 수 있음

- 테스트 관련 연산자

  - 숫자
    - gt: 큰지
    - ge: 크거나 같은지
    - lt: 작은지
    - le: 작거나 같은지
    - eq: 같은지
    - ne: 다른지
  - 문자열
    - = 또는 ==: 같은지
    - !=: 같지 않은지
    - z: 문자열의 길이가 0인지
    - n: 문자열의 길이가 0이 아닌지
  - 기타
    - -f: 일반 파일이 있는지
    - -d: 디렉터리가 있는지
    - -L: 파일이 심볼릭 링크인지
    - -r: 사용자에게 읽기 권한이 있는지

  ```shell
  [user@host ~]$ test 1 -gt 0 ; echo $?
  0
  [user@host ~]$ test 0 -gt 1 ; echo $?
  1
  ```

  - test 명령어를 직접 사용하는 방법 외에도 Bash 테스트 명령 구문[ <TESTEXPRESSION> ] 또는 최신 확장 테스트 명령 구문[[ <TESTEXPRESSION> ]]을 사용해 테스트 가능함
    - [ <TESTEXPRESSION> ]
      - test 명령과 완전히 동일
      - 쉘이 먼저 변수 확장을 하고, 인자를 전달함
    - [[ <TESTEXPRESSION> ]]
      - Bash 내장 명령
      - =~와 같은 정규식 기능 지원

  ```shell
  [user@host ~]$ [[ 1 -eq 1 ]]; echo $?
  0
  [user@host ~]$ [[ 1 -ne 1 ]]; echo $?
  1
  [user@host ~]$ [[ 8 -gt 2 ]]; echo $?
  0
  [user@host ~]$ [[ 2 -ge 2 ]]; echo $?
  0
  [user@host ~]$ [[ 2 -lt 2 ]]; echo $?
  1
  [user@host ~]$ [[ 1 -lt 2 ]]; echo $?
  0
  ```

  ```shell
  # 정규표현식 사용
  [[ "abc123" =~ ^abc[0-9]+$ ]]  # OK (Bash 확장)
  [ "abc123" =~ ^abc[0-9]+$ ]    # 에러 발생
  
  # 변수에 공백이 있을 때
  A="hello world"
  [[ $A = "hello world" ]]  # OK
  
  # [ ... ]는 쉘이 먼저 변수 확장을 하고 그대로 인자로 전달함
  # [ hello world = "hello world" ], 인자가 4개가 되어 에러 발생
  [ $A = "hello world" ]    # 에러 또는 예기치 않음 (공백이 분리됨)
  
  # [ "hello world" = "hello world" ], 인자가 3개가 되어 에러 해결
  [ "$A" = "hello world" ]  # OK
  
  # 복합 조건
  [[ $a -gt 3 && $b -lt 10 ]]  # OK
  [ $a -gt 3 ] && [ $b -lt 10 ]  # 구식 방식
  ```

  

  

## 조건부 구조

```shell
#!/bin/bash

# animal 변수에 키보드 입력 값 저장
read -p "Give me an animal and I will tell you how many legs it has" animal

case $animal in
  person)
    echo -e "A person has 2 legs\n"
  ;;
  
  lion)
    echo -e "A lion has 4 legs\n"
  ;;
  
  bee)
    echo -e "A bee has 6 legs\n"
  ;;
  
  spider)
    echo -e "A spider has 8 legs\n"
  ;;
  
  crab)
    echo -e "A crab has 10 legs\n"
  ;;
  
  # 위에서 매치되지 않은 모든 경우
  *)
    echo -e "Crikey, my creator hasn't taught me aboud that creature\n"
    exit 1
  ;;
esac
```



```shell
if <CONDITION>; then
      <STATEMENT>
      ...
      <STATEMENT>
elif <CONDITION>; then
      <STATEMENT>
      ...
      <STATEMENT>
else
      <STATEMENT>
      ...
      <STATEMENT>
fi
```

- 조건문의 구조는 위와 같음



```shell
[user@host ~]$ systemctl is-active mariadb > /dev/null 2>&1
[user@host ~]$ MARIADB_ACTIVE=$?
[user@host ~]$ sudo systemctl is-active postgresql > /dev/null 2>&1
[user@host ~]$ POSTGRESQL_ACTIVE=$?

# mariadb 서비스가 실행중이면 mysql 실행
# postgresql 서비스가 실행중이면 psql 실행
# mariadb, postresql 서비스 모두 비활성이면 sqlite3 실행
[user@host ~]$ if  [[ "$MARIADB_ACTIVE" -eq 0 ]]; then \
mysql; \
elif  [[ "$POSTGRESQL_ACTIVE" -eq 0 ]]; then \
psql; \
else \
sqlite3; \
fi
```





# 정규 표현식을 사용하여 명령 출력에서 일치하는 텍스트 찾기

- 특정 내용을 찾을 수 있는 패턴 일치 메커니즘을 제공
- vim, grep, less 명령은 정규 표현식을 사용할 수 있으며, Perl, Python, C 등 프로그래밍 언어도 정규표현식을 지원함

```
cat
dog
concatenate
dogma
category
educated
boondoggle
vindication
chilidog
```

위 내용이 담긴 파일에서 `cat`를 검색하면 다음과 같이 `cat` 문자열이 포함된 라인들이 검색됨

```
cat
concatenate
category
educated
vindication
```

- 라인 안에서 `cat` 문자열의 위치와 관계없이 존재하면 검색되는데, 일치 항목의 위치를 제어하려면 **행 앵커** 문자 사용함
- 행 시작 부분에서만 검색하려면 캐럿(^) 문자 사용함
- 행 끝 부분에서만 검색하려면 달러($) 문자 사용함
- 검색 식만 포함된 행을 찾으려면 행 시작 앵커와 끝 앵커를 함께 사용함
  - ^cat$

## 기본 및 확장 정규 표현식

- 정규 표현식에는 기본 정규 표현식과 확장 정규 표현식으로 두 가지 유형이 존재함
- 차이점으로 |, +, ?, (, ), {, } 특수 문자의 동작이 있음
- 확장 정규 표현식 구문에서는 백슬래시 \ 문자를 접두사로 사용하지 않는 경우에만 이 문자에 특별한 의미가 있음

| 기본 구문     | 확장 구문 | 설명                                                         |
| :------------ | :-------- | :----------------------------------------------------------- |
| .             |           | 마침표(`.`)는 모든 단일 문자와 일치합니다.                   |
| ?             |           | 앞의 항목이 선택 사항이며 최대 한 번만 일치합니다.           |
| *             |           | 앞의 항목이 0번 이상 일치합니다.                             |
| +             |           | 앞의 항목이 1번 이상 일치합니다.                             |
| \\{`n`\\}     | {`n`}     | 앞의 항목이 정확히 `n` 번 일치합니다.                        |
| \\{`n`,\\}    | {`n`,}    | 앞의 항목이 `n` 번 이상 일치합니다.                          |
| \\{,`m`\\}    | {,`m`}    | 앞의 항목이 최대 `m` 번 일치합니다.                          |
| \\{`n`,`m`\\} | {`n`,`m`} | 앞의 항목이 `n`번 이상, 최대 `m`번 일치합니다.               |
| [:alnum:]     |           | 영숫자 문자 `[:alpha:]` 및 `[:digit:]`입니다. 'C' 로케일과 ASCII 문자 인코딩에서는 이 표현식이 `[0-9A-Za-z]`와 같습니다. |
| [:alpha:]     |           | 알파벳 문자 `[:lower:]` 및 `[:upper:]`입니다. 'C' 로케일과 ASCII 문자 인코딩에서는 이 표현식이 `[A-Za-z]`와 같습니다. |
| [:blank:]     |           | 공백 문자(공백 및 탭)입니다.                                 |
| [:cntrl:]     |           | 제어 문자입니다. ASCII에서 이러한 문자의 8진수 코드는 000~037 및 177(DEL)입니다. |
| [:digit:]     |           | 숫자 `0 1 2 3 4 5 6 7 8 9`입니다.                            |
| [:graph:]     |           | 그래픽 문자 `[:alnum:]` 및 `[:punct:]`입니다.                |
| [:lower:]     |           | 소문자입니다. 'C' 로케일 및 ASCII 문자 인코딩에서는 `a b c d e f g h i j k l m n o p q r s t u v w x y z`입니다. |
| [:print:]     |           | 출력 가능한 문자는 `[:alnum:]`, `[:punct:]`, 공백입니다.     |
| [:punct:]     |           | 문장 부호 문자입니다. 'C' 로케일 및 ASCII 문자 인코딩에서는 `! " # $ % & ' ( ) * + , - . / : ; < = > ? @ [ \ ] ^ _ ' { |} ~`입니다. |
| [:space:]     |           | 공백 문자입니다. 'C' 로케일에서는 탭, 줄 바꿈, 세로 탭, 용지 공급, 캐리지 리턴, 공백입니다. |
| [:upper:]     |           | 대문자입니다. 'C' 로케일 및 ASCII 문자 인코딩에서는 `A B C D E F G H I J K L M N O P Q R S T U V W X Y Z`입니다. |
| [:xdigit:]    |           | 16진수 `0 1 2 3 4 5 6 7 8 9 A B C D E F a b c d e f`입니다.  |
| \b            |           | 단어의 가장자리에 있는 빈 문자열과 일치합니다.               |
| \B            |           | 단어의 가장자리가 아닌 위치에 있는 빈 문자열과 일치합니다.   |
| \<            |           | 단어의 시작 부분에 있는 빈 문자열과 일치합니다.              |
| \>            |           | 단어의 끝 부분에 있는 빈 문자열과 일치합니다.                |
| \w            |           | 단어 구성 요소와 일치합니다. `[_[:alnum:]]`의 동의어입니다.  |
| \W            |           | 비단어 구성 요소와 일치합니다. `[^_[:alnum:]]`의 동의어입니다. |
| \s            |           | 공백과 일치합니다. `[[:space:]]`의 동의어입니다.             |
| \S            |           | 비공백과 일치합니다. `[^[:space:]]`의 동의어입니다.          |

### 정규 표현식에서 와일드카드 및 승수 사용

- 정규 표현식에서는 점 문자(.)를 와일드카드로 사용하여 단일 행에서 일치하는 단일 문자를 찾음
  - c.t: c, 단일문자 t가 순서대로 포함된 문자열을 검색함
    - cat, concatenate, vindication, cut, c$t 등
- 무제한 와일드카드를 사용할 경우 와일드카드와 일치하는 문자를 예측할 수 없음
- 대괄호 문자를 or 조건으로 사용 가능
  - c[aou]t: c로 시작하고 그 뒤에  a or o or u가 오고 t가 이어진 패턴
    - cat, cot, cut
- 승수는 정규 표현식에서 이전 문자 또는 와일드카드에 적용됨
  - 별표(*) 문자: 바로 앞 문자가 0회 이상 반복(공백도 가능)
    - c[aou]*t: coat, coot, ct 등
    - c.*t: cat, ccoat, culvert, ct 등
  - \\{숫자\\}: 문자 수를 명시
    - c.\\{2\\}t: coat, cart 등 c와 t 사이에 2개 문자 존재
  - 승수?
    - 문자의 반복 개수를 결정하는 기호

## 명령줄에서 정규 표현식 찾기

- grep 명령은 정규 표현식을 사용해 일치하는 데이터를 격리함
- grep 명령을 사용해 단일 파일 또는 여러 파일에서 일치하는 데이터를 찾을 수 있음
- grep을 사용해 여러 파일에서 일치하는 데이터를 찾는 경우 파일 이름, 콜론 문자, 정규 표현식과 일치하는 행이 차례로 출력됨
- grep 명령의 종료 코드
  - 패턴 찾을 수 없으면 1
  - 파일 자체가 없으면 2


```shell
# /usr/share/dict/words 파일에서 computer로 시작하는 데이터 격리
[user@host ~]$ grep '^computer' /usr/share/dict/words
computer
computerese
computerise
computerite
computerizable
computerization
computerize
computerized
computerizes
computerizing
computerlike
computernik
computers

# ps aux 명령의 결과에서 chrony가 포함된 데이터 격리
[root@host ~]# ps aux | grep chrony
chrony     662  0.0  0.1  29440  2468 ?        S    10:56   0:00 /usr/sbin/chronyd
```



## 일반적인 grep 명령 옵션

| 옵션            | 기능                                                         |
| :-------------- | :----------------------------------------------------------- |
| `-i`            | 제공된 정규 표현식을 사용하고 대소문자를 구분하지 않습니다(대소문자 구분 없이 실행). |
| `-v`            | 정규 표현식과 일치하는 항목이 *없는* 행만 표시합니다.        |
| `-r`            | 정규 표현식과 일치하는 데이터를 파일 그룹 또는 디렉터리에서 반복적으로 검색합니다. |
| `-A *`NUMBER`*` | 정규 표현식과 일치하는 항목 다음의 행 *수*를 표시합니다.     |
| `-B *`NUMBER`*` | 정규 표현식과 일치하는 항목 앞의 행 *수*를 표시합니다.       |
| `-e`            | 여러 개의 `-e` 옵션을 사용하면 복수의 정규 표현식을 제공할 수 있으며, 논리 OR과 함께 사용됩니다. |
| `-E`            | 제공된 정규 표현식을 구문 분석할 때 기본 정규 표현식 구문 대신 확장된 정규 표현식 구문을 사용합니다. |
| `-n`            | 발견된 데이터의 행 번호를 함께 출력                          |

```shell
# /etc/httpd/conf/httpd.conf 파일 내용 조회
[user@host ~]$ cat /etc/httpd/conf/httpd.conf
...output omitted...
ServerRoot "/etc/httpd"

#
# Listen: Allows you to bind Apache to specific IP addresses and/or
# ports, instead of the default. See also the <VirtualHost>
# directive.
#
# Change this to Listen on a specific IP address, but note that if
# httpd.service is enabled to run at boot time, the address may not be
# available when the service starts.  See the httpd.service(8) man
# page for more information.
#
#Listen 12.34.56.78:80
Listen 80
...output omitted...

# 대소문자 구분없이 serverroot가 포함된 행을 조회
[user@host ~]$ grep -i serverroot /etc/httpd/conf/httpd.conf
# with "/", the value of ServerRoot is prepended -- so 'log/access_log'
# with ServerRoot set to '/www' will be interpreted by the
# ServerRoot: The top of the directory tree under which the server's
# ServerRoot at a non-local disk, be sure to specify a local disk on the
# same ServerRoot for multiple httpd daemons, you will need to change at
ServerRoot "/etc/httpd"

# 대소문자 구분없이 server가 포함되지 않은 행을 조회
[user@host ~]$ grep -v -i server /etc/hosts
127.0.0.1 localhost.localdomain localhost
172.25.254.254 classroom.example.com classroom
172.25.254.254 content.example.com content
172.25.254.254 materials.example.com materials
### rht-vm-hosts file listing the entries to be appended to /etc/hosts

172.25.250.9	workstation.lab.example.com workstation
172.25.250.254	bastion.lab.example.com bastion
172.25.250.220  utility.lab.example.com utility
172.25.250.220  registry.lab.example.com registry


# #이나 ;로 시작하지 않는 행(주석 제외)을 조회
[user@host ~]$ grep -v '^[#;]' \
/etc/systemd/system/multi-user.target.wants/rsyslog.service
[Unit]
Description=System Logging Service
Documentation=man:rsyslogd(8)
Documentation=https://www.rsyslog.com/doc/

[Service]
Type=notify
EnvironmentFile=-/etc/sysconfig/rsyslog
ExecStart=/usr/sbin/rsyslogd -n $SYSLOGD_OPTIONS
ExecReload=/usr/bin/kill -HUP $MAINPID
UMask=0066
StandardOutput=null
Restart=on-failure

LimitNOFILE=16384

[Install]
WantedBy=multi-user.target


# 'pam_unix', 'user root', 'Accepted publickey'가 하나라도 포함된 데이터를 조회
[root@host ~]# cat /var/log/secure | grep -e 'pam_unix' \
-e 'user root' -e 'Accepted publickey' | less
Mar  4 03:31:41 localhost passwd[6639]: pam_unix(passwd:chauthtok): password changed for root
Mar  4 03:32:34 localhost sshd[15556]: Accepted publickey for devops from 10.30.0.167 port 56472 ssh2: RSA SHA256:M8ikhcEDm2tQ95Z0o7ZvufqEixCFCt+wowZLNzNlBT0
Mar  4 03:32:34 localhost systemd[15560]: pam_unix(systemd-user:session): session opened for user devops(uid=1001) by (uid=0)

# 'student', 'devops', 'lolly' 값이 포함된 데이터를 조회
[student@workstation ~]$ grep -E 'student|devops|lolly' /etc/passwd
student:x:1000:1000:Student User:/home/student:/bin/bash
devops:x:1001:1001:Devops User:/home/devops:/bin/bash
lolly:x:1003:1003::/home/lolly:/bin/bash

# # 문자를 제외한 문자로 시작하는 데이터(주석 제외)를 조회
[root@servera ~]# grep ^[^#] /etc/ssh/sshd_config
Include~~
AuthorizedKeysFile ~~
ClientAliveInterval 60
Subsystem sftp ~~
PasswordAuthentication ~~
```

