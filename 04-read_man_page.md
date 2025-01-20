# Read Man Page
- 모든 명령어나 옵션을 다 외워서 적용할 수 없다
- 필요에 맞는 명령어를 찾고 파악하는 방법을 배운다

## man command
- system에서 사용할 수 있는 command는 도움말 페이지를 제공
- 페이지는 `/usr/share/man` 디렉터리의 하위 디렉터리에 저장되어 있음
- man 명령을 사용하여 명령줄에서 액세스할 수 있음
- 도움말 페이지는 크기가 커서 여러 섹션으로 분할되어 구분함

| 섹션 | 내용 유형                  | 설명                                   | 예시 명령어           |
|------|---------------------------|----------------------------------------|-----------------------|
| 1    | 사용자 명령               | 실행 가능한 프로그램 및 셸 프로그램     | `ls`, `cd`           |
| 2    | 시스템 호출               | 사용자 공간에서 호출된 커널 루틴       | `open`, `read` (C)   |
| 3    | 라이브러리 함수           | 프로그램 라이브러리에서 제공           | `printf`, `malloc`   |
| 4    | 특수 파일                 | 예: 장치 파일                          | `/dev/null`, `/dev/sda` |
| 5    | 파일 형식                 | 많은 구성 파일 및 구조의 경우           | `/etc/passwd`        |
| 6    | 게임 및 화면 보호기       | 오락 프로그램 섹션                     | `fortune`, `sl`      |
| 7    | 법례, 표준 및 기타        | 프로토콜 및 파일 시스템                | `man`, `info`        |
| 8    | 시스템 관리 및 권한 명령  | 유지 관리 작업                         | `sudo`, `systemctl`  |
| 9    | Linux 커널 API            | 내부 커널 호출                         | `syscall` (C 코드에서) |

```bash
# fortune
You're not my type.  For that matter, you're not even my species!!!
Future looks spotty.  You will spill soup in late evening.
You worry too much about your job.  Stop it.  You are not paid enough to worry.
Your love life will be... interesting.
```
```bash
      ====        ________                ___________
  _D _|  |_______/        \__I_I_____===__|_________|
   |(_)---  |   H\________/ |   |        =|___ ___|   
   /     |  |   H  |  |     |   |         ||_| |_||   
  |      |  |   H  |__--------------------| [___] |   
  | ________|___H__/__|_____/[][]~\_______|       |   
  |/ |   |-----------I_____I [][] []  D   |=======|__
__/ =| o |=-~~\  /~~\  /~~\  /~~\ ____Y___________|__
 |/-=|___|=O=====O=====O=====O   |_____/~\___/        
  \_/      \__/  \__/  \__/  \__/      \_/           

```
- man example
  - 예시나 관련 파일, 버그에 관한 정보도 알 수 있음
```bash
NAME
       echo - display a line of text

SYNOPSIS
       echo [SHORT-OPTION]... [STRING]...
       echo LONG-OPTION

DESCRIPTION
       Echo the STRING(s) to standard output.

       -n     do not output the trailing newline

       -e     enable interpretation of backslash escapes

       -E     disable interpretation of backslash escapes (default)

       --help display this help and exit
AUTHOR
       Written by Brian Fox and Chet Ramey.

COPYRIGHT
       Copyright © 2013 Free Software Foundation, Inc.  License GPLv3+: GNU GPL version
       3 or later <http://gnu.org/licenses/gpl.html>.
       This  is free software: you are free to change and redistribute it.  There is NO
       WARRANTY, to the extent permitted by law.

SEE ALSO
       The full documentation for echo is maintained as a Texinfo manual.  If the  info
       and echo programs are properly installed at your site, the command

              info coreutils 'echo invocation'
```
- `-k` option으로 연관된 manual을 검색할 수 있음
  - `passwd (1ossl)`은 OpenSSL의 일부로 제공되는 passwd 명령어에 대한 설명
```bash
[user@host ~]$ man -k passwd
chgpasswd (8)        - update group passwords in batch mode
chpasswd (8)         - update passwords in batch mode
fgetpwent_r (3)      - get passwd file entry reentrantly
getpwent_r (3)       - get passwd file entry reentrantly
...
passwd (1)           - update user's authentication tokens
passwd (1ossl)       - OpenSSL application commands
passwd (5)           - password file
passwd2des (3)       - RFS password encryption
```
- `-K` option으로도 검색 가능하지만 전체 페이지를 보며 검색하는 것이라 리소스와 시간이 더 많이 걸림

## read man command
- 'q' 를 사용해 man page 종료
- '/{keyward}'로 man page 내부에서 검색 가능

