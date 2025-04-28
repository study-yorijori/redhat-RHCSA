# 시스템 간 안전한 파일 동기화
- 로컬 파일 또는 디렉터리의 내용을 원격 서버 사본과 효율적으로 안전하게 동기화
## 원격 파일 및 디렉터리 동기화
- `rsync`: 시스템의 파일을 다른 시스템에 복사하는 방법
- 변경된 파일 부분만 동기화하여 복사되는 데이터를 최소화하는 알고리즘을 사용
- 두 서버 간에 두 파일 또는 디렉터리가 유사한 경우 `rsync` 명령은 파일 시스템 간 차이점만 복사

## rsync option

| 옵션 | 설명 |
|--|------|
-r | 재귀적으로 전체 디렉터리 트리 동기화
-l | 심볼릭 링크 동기화
-p | 권한 보존
-t | 타임스탬프 보존
-g | 그룹 소유권 보존
-o | 파일 소유권 보존 
-a | 아카이브 모드 (권한, 심볼릭 링크, 타임스탬프 유지, 디렉토리 재귀 복사 등)
-v | 자세히 출력 (verbose)
-z | 전송 중 압축
-h | 사람이 읽기 쉬운 크기 표시 (human-readable)
--progress | 전송 진행 상황 표시
--delete | 목적지에서 소스에 없는 파일 삭제 (동기화할 때 유용)

## rsync 사용
- 두 시스템 중 하나를 소스로 사용하여 로컬 파일 또는 디렉터리 내용을 원격 시스템의 파일 또는 디렉터리와 동기화할 수 있음
- `sftp` 명령과 마찬가지로 `rsync` 명령은 `[user@]host:/path` 형식으로 원격 위치를 지정합니다. 원격 위치가 소스 또는 대상 시스템일 수 있지만 두 시스템 중 하나는 로컬이어야 함
~~~bash
[root@host ~]# rsync -av /var/log hosta:/tmp (or  rsync -av hosta:/var/log /tmp)
root@hosta's password: password
receiving incremental file list
log/
log/README
log/boot.log
...output omitted...
sent 9,783 bytes  received 290,576 bytes  85,816.86 bytes/sec
total size is 11,585,690  speedup is 38.57
~~~
~~~bash
[root@host ~]# rsync -av /var/log /tmp
~~~

## rsync의 "압축 지원" (-z 옵션)
- rsync -z를 쓰면 네트워크를 통한 데이터 전송 시에 압축
- zip 처럼 로컬 파일 자체를 압축하는 건 아니고 보내기 전에 데이터를 압축하고, 받는 쪽에서는 다시 푸는 방식
- 네트워크가 느릴 때 데이터 양이 줄어드니까 전송 속도 개선
- CPU를 더 쓰긴 하지만, 네트워크 병목이 있는 경우 선택
- 이미 압축된 파일들 (예: .zip, .tar.gz, .jpg)은 -z 옵션을 써도 크기가 거의 줄지 않아서 의미가 없음

## rsync의 "부분 복구" (Delta Transfer Algorithm)
- "델타 전송(Delta Transfer)" 라고 부르는 기술.
- 이전에 rsync를 통해 수신자 B 서버에는 이미 예전 버전의 file.txt가 존재. A 서버에서 조금 수정된 file.txt를 B 서버에 동기화하려고 rsync를 실행
- 작동 방식
  - 송신자(sender)와 수신자(receiver) 모두 파일을 블록 단위로 나눔
  - 보통 블록 크기는 512바이트, 1KB, 4KB 같은 크기(`--block-size`)
  1. 수신자
    - 수신자는 자기 파일의 각 블록에 대해 해시값을 계산
    - 수신자는 블록별 [offset, size, weak checksum, strong checksum] 목록을 만듬
    - 이 목록을 송신자 A에게 전송
  2. 송신자 
    - 송신자도 자기 파일의 각 블록에 대해 해시값을 계산
    - 수신자가 보낸 블록 목록과 비교
    - 다른 부분만 전송

## 커넥션이 끊어진 경우 복구
- 전송 도중 연결이 끊겨도 (네트워크 오류 등) 다음에 rsync를 다시 실행하면 기존에 전송된 부분과 비교해서 아직 덜 복사된 부분만 이어서 전송
- `--partial`: 전송 도중 끊겼을 때, 수신자(receiver) 쪽에 "지금까지 받은 데이터"를 남겨두는 옵션
- 기본적으로 rsync는 전송 실패하면 받던 파일을 삭제
-  `--partial` 을 써서 끊긴 파일을 지우지 않고 남겨 이어받기 할 수 있도록 함
- 대용량 파일 복사 시 유용

