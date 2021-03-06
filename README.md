# Linux
## Linux shell
### What is sh?
``sh (or the Shell Command Language)`` is a programming language described by the [**POSIX standard**](https://pubs.opengroup.org/onlinepubs/009695399/utilities/xcu_chap02.html). It has many implementations ``(ksh88, Dash, ...)``. ``Bash`` can also be considered an implementation of sh (see below).

Because ``sh`` is a **specification**, not an implementation, **``/bin/sh`` is a symlink (or a hard link) to an actual implementation on most POSIX systems.**

### What is Bash?
Bash started as an **``sh``-compatible implementation** (although it predates the POSIX standard by a few years), but as time passed it has acquired many extensions. Many of these extensions may change the behavior of valid POSIX shell scripts, so by itself Bash is not a valid POSIX shell. Rather, it is a dialect of the POSIX shell language.

**Bash supports a ``--posix`` switch, which makes it more POSIX-compliant.** It also tries to mimic POSIX if invoked as ``sh``.

### sh = bash?
**For a long time, ``/bin/sh`` used to point to ``/bin/bash`` on most GNU/Linux systems.** As a result, it had almost become safe to ignore the difference between the two. But that started to change recently.

Some popular examples of systems where **``/bin/sh`` does not point to ``/bin/bash`` (and on some of which ``/bin/bash`` may not even exist)** are:

1. Modern Debian and Ubuntu systems, which **symlink ``sh`` to ``dash`` by default**;
2. ``Busybox``, which is usually run during the Linux system boot time as part of   ``initramfs``. It uses the  ``ash`` shell implementation.
3. ``BSD`` systems, and in general any non-Linux systems. ``OpenBSD`` uses ``pdksh``, a descendant of the ``KornShell``. FreeBSD's ``sh`` is a descendant of the original Unix Bourne shell. ``Solaris`` has its own ``sh`` which for a long time was not POSIX-compliant; a free implementation is available from the ``Heirloom project``.

**How can you find out what ``/bin/sh`` points to on your system?**
The complication is that ``/bin/sh`` could be a symbolic link or a hard link. If it's a symbolic link, a portable way to resolve it is:
```bash
# example
% file -h /bin/sh
/bin/sh: symbolic link to bash
# in case of my ubuntu linux
gusami@master:~$ file -h /bin/sh
/bin/sh: symbolic link to dash
```
If it's a hard link, try
```bash
# example
% find -L /bin -samefile /bin/sh
/bin/sh
/bin/bash
# in case of my ubuntu linux
gusami@master:~$ find -L /bin -samefile /bin/sh
/bin/sh
/bin/dash
```
In fact, the ``-L`` flag covers both symlinks and hardlinks, but the disadvantage of this method is that it is not portable — POSIX does not require ``find`` to support the ``-samefile`` option, although both ``GNU find`` and ``FreeBSD find`` support it.

### Shebang line
Ultimately, **it's up to you to decide which one to use, by writing the «shebang» line as the very first line of the script.**

E.g.
```bash
#!/bin/sh
```
will use ``sh`` (and whatever that happens to point to),

```bash
#!/bin/bash
```
will use ``/bin/bash`` if it's available (and fail with an error message if it's not). Of course, you can also specify another implementation, e.g.
```bash
#!/bin/dash
```
### Which one to use
For my own scripts, I prefer ``sh`` for the following reasons:
- it is standardized
- it is much simpler and easier to learn
- it is portable across POSIX systems — even if they happen not to have ``bash``, they are required to have ``sh``

There are advantages to using ``bash`` as well. Its features make programming more convenient and similar to programming in other modern programming languages. These include things like scoped local variables and arrays. **Plain ``sh`` is a very minimalistic programming language.**

## Boot sequence
![boot_sequences.png](./images/boot_sequences.png)
- 자세한 내용은 [까망눈연구소 링크](https://wogh8732.tistory.com/72)를 참조 바람
  - Step 01: 바이오스
  - Step 02: 디스크에 있는 ``Master Boot Record``의 코드를 메모리로 로딩하고 수행
  - Step 03: ``Master Boot Record``는 실제 부트 로더를 로딩
  - Step 04: 부트 로더가 커널의 이미지를 메모리상에 로딩
  - Step 05: 커널이 수행할 여러가지 프로그램들과 드라이버들을 ``Init Ram Disk``라고하는 이미지를 가지고 Ram상에 가상으로 디스크를 생성
    - 기본적으로 시스템이 동작하게 됨
  - Step 06: 시스템이 동작하는 과정에서 제일 먼저 ``/sbin/init``라는 프로세스가 동작
    - 모든 프로세스들의 조상 프로세스
  - Step 07: ``/sbin/init`` 프로세스를 기준으로 기타 수행에 필요한 프로세스를 동작시킴
    - GUI와 관련된 프로세스도 기동해서 바탕화면이 보임

## Systemd란?
- init이 퇴출되고 init이 하던 작업을 물려받은 대단한 systemd 프로세스
- Systemd는 부팅부터 서비스관리 로그관리등의 시스템 전반적인 영역에 걸쳐있는 프로세스
- 부팅시에도 병렬로 실행되어서 상당히 부팅속도가 빨리짐

### 등장 배경
- 예전의 PID 1이었던 init은 현재로부터 수 십 년 전에 처음 소개된 프로그램인데 그 때의 구조를 거의 바꾸지 않고 계속 기능이 추가되며 날이 갈수록 복잡해지는 프로그램들로 인해 효율이 떨어졌습니다. 그리고 그 구조라는 것이 시작할 프로그램을 구동하는 쉘 스크립트를 특정 run-level의 rc 디렉토리에 추가하는 것이고, init은 부팅 과정에서 단계적으로 run-level을 올려가며 해당 run-level에 포함된 스크립트들을 순차적으로 실행시키니 설정의 난잡함 뿐만 아니라 속도마저 느렸습니다.
이런 문제들을 해결하기 위해 Red Hat에 근무하는 Lennart Poettering과 Kay Sievers란 사람들이  systemd를 만들기 시작했습니다. 이렇게 만들어진 systemd는 당연하지만 init보다 우월한 성능과 직관적인 설정을 가지게 되었습니다. 의존성을 해치지 않으며 가능한 한 병렬로 시작 프로그램들을 실행시키는 것으로 부팅 속도를 끌어올리고, 프로그램 실행을 위한 파일로는 쉘 스크립트가 아니라 .service라는 systemd만의 unit을 통해 체계적이면서 가독성이 좋게 설정이 가능해졌습니다.
게다가 systemd는 단지 init 뿐만 아니라 다른 프로그램들의 기능들마저 뺏어오기까지 합니다. 실제로 컴퓨터의 네임서버 주소를 설정하는 resolvconf의 자리를 systemd-resolved가, DHCP 서버에서 IP를 받아와서 네트위크 인터페이스에 설정하는 dhcpcd의 자리를 systemd-networkd가 대체할 수 있습니다. 이외에도 시스템 내부의 udev가 systemd에 포함되는 등 여러 방면에서 systemd의 존재가 강력해지고 있습니다. 이렇게 systemd가 여러 영역을 아우르는 것을 보고 혹자들은 하나만 잘하자 라는 UNIX의 철학에 어긋난다고 말하기도 합니다. 이건 제 개인적인 생각입니다만 UNIX 철학같은걸 따지기보다 실제 사용에 있어서 편리성과 앞으로의 발전 가능성을 보는 것이 더 바람직하다고 생각합니다. 그리고 systemd는 PID 1의 임무를 착실하게 수행하고 있으며 다른 부분에 대해서는 systemd-*의 이름으로 따로 존재하니 UNIX 철학에 너무 벗어나지 않는 것처럼 보이기도 합니다.

- 먼저 systemd 구성을 알아봅시다.
  - systemd : init 데몬
  - systemd-journald : 다른 데몬(프로세스)들의 출력(syslog, 표준, 에러 출력), 로그 저장 데몬
  - systemd-logind : 사용자 로그인, 세션 등 관리 데몬
  - systemd-udevd : 장치 관리자 데몬
  - systemd-networkd : 네트워크 관리 데몬. DHCP 뿐만 아니라 Virtual Lan 설정까지 가능
  - systemd-resolved : DNS 해석 데몬
  - systemd-timesyncd : NTP로 컴퓨터 시간 동기화 데몬
  - systemd-boot : UEFI 부트로더

- 데몬이나 systemd의 사용자 설정파일은 ``/etc/systemd/``에 존재
- 부팅속도 체크
```bash
# systemd-analyze
Startup finished in 288ms (kernel) + 714ms (initrd) + 15.430s (userspace) = 16.434s
```
```bash
# systemd-analyze blame
          2.509s network.service
          2.230s mariadb.service
           845ms kdump.service
           405ms postfix.service
           319ms lvm2-monitor.service
           244ms tuned.service
           125ms iprupdate.service
            95ms iprinit.service
```
### 정보
- 1번 프로세스id를 가지고 있음
```bash
#ps -p 1 ef
  PID TTY      STAT   TIME COMMAND
    1 ?        Ss     1:24 /usr/lib/systemd/systemd --switched-root --system --deserialize 23
```
- 설정 파일
  - ``/etc/systemd`` 디렉토리에 위치
```bash
#ls /etc/systemd
bootchart.conf  journald.conf  logind.conf  system  system.conf  user  user.conf
```
- 디렉토리 정보
```bash
/etc/systemd/ : configure
/lib/systemd/ : 바이너리 실행파일이 존재
/lib/systemd/system/ : Service, Target이 위치
```
### 서비스 제어
- 서비스 목록 확인
```bash
# systemctl list-unit-files
```
- 서비스 시작
```bash
# systemctl start [서비스명]
```
- 서비스 종료
```bash
# systemctl stop [서비스명]
```
- 서비스 재시작
```bash
# systemctl restart [서비스명]
```
- 서비스 활성화
```bash
# systemctl enable [서비스명]
```
- 서비스 비활성화
```bash
# systemctl disable [서비스명]
```
- 서비스 갱신
```bash
# systemctl reload [서비스명]
```
### 시스템 명령
```bash
# systemctl halt ( halt )
# systemctl reboot ( 리부팅 )
```
### 로그관리
- 로그는 systemd-journald를 통해 관리가 된다. 제어는 journalctl를 사용하면 된다.
- 이벤트 조회
```bash
# journalctl /sbin/crond
```
- 특정일자부터 이벤트로그 조회방법
```bash
# journalctl --since=today
```
- 이벤트 속성조회 ( debug, info, err 가능 )
```bash
# journalctl -p err
```
### 로그 설정
- 로그파일을 처리해주는 배치로 잘 관리하지 않을 경우에 아래와 같이 파일 사이즈와 기간을 설정
```bash
# vi /etc/systemd/journald.conf
SystemMaxUse=500M
MaxFileSec=1month
```

## Linux 명령어
### 명령어 도움말
  - 페이지 이동 방법
    - 이전 페이지 : page up 키 or b
    - 다음 페이지 : page down 키 or 스페이스바
    - 종료 : q
```bash
$ man 명령어
```
### nslookup
- 원하는 도메인의 ip 확인
```bash
$man nslookup    
NAME
       nslookup - query Internet name servers interactively

SYNOPSIS
       nslookup [-option] [name | -] [server]

DESCRIPTION
       Nslookup is a program to query Internet domain name servers.  Nslookup has two modes: interactive and non-interactive. Interactive mode allows the user to query name servers for
       information about various hosts and domains or to print a list of hosts in a domain. Non-interactive mode is used to print just the name and requested information for a host or
       domain.
```
- 사용예
```bash
gusami@master:~$nslookup www.daum.net
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
www.daum.net	canonical name = www-daum-uj2suuqw.kgslb.com.
Name:	www-daum-uj2suuqw.kgslb.com
Address: 211.249.220.24
```
### tee
- 표준 입력에서 읽어서 표준 출력과 파일에 쓰는 명령어
```bash
$man tee
NAME
       tee - read from standard input and write to standard output and files

SYNOPSIS
       tee [OPTION]... [FILE]...

DESCRIPTION
       Copy standard input to each FILE, and also to standard output.

       -a, --append
              append to the given FILEs, do not overwrite

       -i, --ignore-interrupts
              ignore interrupt signals

       -p     diagnose errors writing to non pipes

       --output-error[=MODE]
              set behavior on write error.  See MODE below

       --help display this help and exit

       --version
              output version information and exit
```
- 사용예 1: 파일 생성
```bash
gusami@master:~$echo "hello" | tee hello.txt
hello
gusami@master:~$cat hello.txt
hello
```
- 사용예 2: 파일에 첨부하기
```bash
gusami@master:~$ echo "hello world" | tee -a hello.txt
hello world
gusami@master:~$ cat hello.txt
hello
hello world
```
### df
- 마운트 된 파일 시스템 별로 disk 사용량을 모니터링 할 때 사용
  - ``-h``: print sizes in human readable format (e.g., 1K 234M 2G)
```bash
DF(1)                                                        User Commands                                                       DF(1)

NAME
       df - report file system disk space usage

SYNOPSIS
       df [OPTION]... [FILE]...

DESCRIPTION
       This manual page documents the GNU version of df.  df displays the amount of disk space available on the file system containing
       each file name argument.  If no file name is given, the space available on all currently mounted file systems is  shown.   Disk
       space  is  shown in 1K blocks by default, unless the environment variable POSIXLY_CORRECT is set, in which case 512-byte blocks
       are used.

       If an argument is the absolute file name of a disk device node containing a mounted file system, df shows the  space  available
       on that file system rather than on the file system containing the device node.  This version of df cannot show the space avail‐
       able on unmounted file systems, because on most kinds of systems doing so requires very nonportable intimate knowledge of  file
       system structures.

OPTIONS
       Show information about the file system on which each FILE resides, or all file systems by default.

       Mandatory arguments to long options are mandatory for short options too.

       -a, --all
              include pseudo, duplicate, inaccessible file systems

       -B, --block-size=SIZE
              scale sizes by SIZE before printing them; e.g., '-BM' prints sizes in units of 1,048,576 bytes; see SIZE format below

       -h, --human-readable
              print sizes in powers of 1024 (e.g., 1023M)

       -H, --si
              print sizes in powers of 1000 (e.g., 1.1G)

       -i, --inodes
              list inode information instead of block usage

       -k     like --block-size=1K

       -l, --local
              limit listing to local file systems

       --no-sync
              do not invoke sync before getting usage info (default)

       --output[=FIELD_LIST]
              use the output format defined by FIELD_LIST, or print all fields if FIELD_LIST is omitted.

       -P, --portability
              use the POSIX output format

       --sync invoke sync before getting usage info

       --total
              elide all entries insignificant to available space, and produce a grand total

       -t, --type=TYPE
              limit listing to file systems of type TYPE

       -T, --print-type
              print file system type

       -x, --exclude-type=TYPE
              limit listing to file systems not of type TYPE

       -v     (ignored)

       --help display this help and exit

       --version
              output version information and exit

       Display  values are in units of the first available SIZE from --block-size, and the DF_BLOCK_SIZE, BLOCK_SIZE and BLOCKSIZE en‐
       vironment variables.  Otherwise, units default to 1024 bytes (or 512 if POSIXLY_CORRECT is set).

       The SIZE argument is an integer and optional unit (example: 10K is 10*1024).  Units are K,M,G,T,P,E,Z,Y  (powers  of  1024)  or
       KB,MB,... (powers of 1000).

       FIELD_LIST  is a comma-separated list of columns to be included.  Valid field names are: 'source', 'fstype', 'itotal', 'iused',
       'iavail', 'ipcent', 'size', 'used', 'avail', 'pcent', 'file' and 'target' (see info page).
```
- 사용예 1: Root file system 디스크 용량 확인  
```bash
gusami@docker-ubuntu:~$df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda5        20G   12G  7.1G  62% /
```
- 사용예 2: 전체 파일 시스템 디스크 용량 확인
```bash
gusami@docker-ubuntu:~$df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            1.9G     0  1.9G   0% /dev
tmpfs           393M  1.2M  392M   1% /run
/dev/sda5        20G   12G  7.1G  62% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/loop0       56M   56M     0 100% /snap/core18/2284
/dev/loop1      219M  219M     0 100% /snap/gnome-3-34-1804/72
/dev/loop2       66M   66M     0 100% /snap/gtk-common-themes/1515
/dev/loop3      248M  248M     0 100% /snap/gnome-3-38-2004/87
/dev/loop4       55M   55M     0 100% /snap/snap-store/558
/dev/loop5       66M   66M     0 100% /snap/gtk-common-themes/1519
/dev/loop6       44M   44M     0 100% /snap/snapd/14295
/dev/loop7       33M   33M     0 100% /snap/snapd/12704
/dev/loop8      128K  128K     0 100% /snap/bare/5
/dev/loop9       62M   62M     0 100% /snap/core20/1270
/dev/loop10      51M   51M     0 100% /snap/snap-store/547
/dev/loop11      56M   56M     0 100% /snap/core18/2253
/dev/loop12     219M  219M     0 100% /snap/gnome-3-34-1804/77
/dev/sda1       511M  4.0K  511M   1% /boot/efi
tmpfs           393M  8.0K  393M   1% /run/user/1000
```
### du
- 디렉토리 별로 disk 사용량을 모니터링 할 때 사용
```bash
DU(1)                                                        User Commands                                                       DU(1)

NAME
       du - estimate file space usage

SYNOPSIS
       du [OPTION]... [FILE]...
       du [OPTION]... --files0-from=F

DESCRIPTION
       Summarize disk usage of the set of FILEs, recursively for directories.

       Mandatory arguments to long options are mandatory for short options too.

       -0, --null
              end each output line with NUL, not newline

       -a, --all
              write counts for all files, not just directories

       --apparent-size
              print  apparent  sizes,  rather  than disk usage; although the apparent size is usually smaller, it may be larger due to
              holes in ('sparse') files, internal fragmentation, indirect blocks, and the like

       -B, --block-size=SIZE
              scale sizes by SIZE before printing them; e.g., '-BM' prints sizes in units of 1,048,576 bytes; see SIZE format below

       -b, --bytes
              equivalent to '--apparent-size --block-size=1'

       -c, --total
              produce a grand total

       -D, --dereference-args
              dereference only symlinks that are listed on the command line

       -d, --max-depth=N
              print the total for a directory (or file, with --all) only if it is N or fewer levels below the command  line  argument;
              --max-depth=0 is the same as --summarize

       --files0-from=F
              summarize  disk usage of the NUL-terminated file names specified in file F; if F is -, then read names from standard in‐
              put

       -H     equivalent to --dereference-args (-D)

       -h, --human-readable
              print sizes in human readable format (e.g., 1K 234M 2G)

       --inodes
              list inode usage information instead of block usage

       -k     like --block-size=1K

       -L, --dereference
              dereference all symbolic links

       -l, --count-links
              count sizes many times if hard linked

       -m     like --block-size=1M

       -P, --no-dereference
              do not follow any symbolic links (this is the default)

       -S, --separate-dirs
              for directories do not include size of subdirectories

       --si   like -h, but use powers of 1000 not 1024

       -s, --summarize
              display only a total for each argument

       -t, --threshold=SIZE
              exclude entries smaller than SIZE if positive, or entries greater than SIZE if negative

       --time show time of the last modification of any file in the directory, or any of its subdirectories

       --time=WORD
              show time as WORD instead of modification time: atime, access, use, ctime or status

       --time-style=STYLE
              show times using STYLE, which can be: full-iso, long-iso, iso, or +FORMAT; FORMAT is interpreted like in 'date'

       -X, --exclude-from=FILE
              exclude files that match any pattern in FILE

       --exclude=PATTERN
              exclude files that match PATTERN

       -x, --one-file-system
              skip directories on different file systems

       --help display this help and exit

       --version
              output version information and exit

       Display values are in units of the first available SIZE from --block-size, and the DU_BLOCK_SIZE, BLOCK_SIZE and BLOCKSIZE  en‐
       vironment variables.  Otherwise, units default to 1024 bytes (or 512 if POSIXLY_CORRECT is set).

       The  SIZE  argument  is  an integer and optional unit (example: 10K is 10*1024).  Units are K,M,G,T,P,E,Z,Y (powers of 1024) or
       KB,MB,... (powers of 1000).
```
- 사용예 1: Root directory 아래의 모든 디렉토리의 디스크 용량 확인
  - ``-h``: print sizes in human readable format (e.g., 1K 234M 2G)
```bash
gusami@docker-ubuntu:~$ du -h /
# 현재 디렉토리를 기준으로 바로 아래 디렉토리 용량 정보
[DevSmartworks@bastion-instance ~]$ du --max-depth=1
6140	./.cache
4	./.config
12	./.ssh
9572	./.kube
12	./.oci
4	./.docker
20	./monitoring
0	./.pki
4	./.vim
19128	.
```
### iptables
- 참고 자료: https://server-talk.tistory.com/169
- CentOS에는 패킷을 필터링 기능을 가지고 있는 netfiler가 존재합니다 IPTABLES는 netfilter를 관리하기 위한 툴
#### iptables란?
- Netfilter 프로젝트에서 개발되었으며 2.4.x 커널 배포 시점 부터 리눅스 일부분으로 제공
- Netfilter에서 제공하는 프레임 워크를 이용한 패킷에 대한 연산(필터링 등)을 수행하게 설계된 함수를 네트워크 스택으로 후킹하는데 사용하는 명령어
- iptables 정책은 정렬된 규칙집합으로 생성
  - 규칙은 특정 분류의 패킷에 대해 취해야 조치를 커널에 알려주며 하나의 iptables의 규칙은 테이블 내에 있는 하나의 체인에 적용
#### iptables 기능
- 상태 추적 기능
  - 방화벽을 지나가는 모든 패킷에 대한 상태를 추적하여 메모리에 기억 기능
  - 기존의 연결을 가정하여 접근할 경우 메모리에 저장된 목록과 비교하여 차단시키는 지능화된 공격 차단 기능
  - 특정 패킷을 차단 또는 허용하는 기능, 서버의 접근제어, 방화벽기능 구현 기능
- 향상된 매칭 기능
  - 방화벽의 기본적인 매칭 정보인 패킷의 IP주소와 포트 번호 뿐만아니라 다양한 매칭 기능
  - 현재의 연결 상태, 포트 목록, 하드웨어 MAC 주소, 패킷 발신자의 유저나 그룹 프로세스, IP 헤더의 TOS등 여러가지 조건을 이용한 세부적인 필터링 기능
- 포트 포워딩 기능 내장
  - NAT(Network Address Translation - 사설IP와 공인IP를 변환해주거나 포트 변환 ) 기능을 자체적으로 포함하고 있어 사용 가능
  - 브리지 등의 다양한 네트워크 구조를 지원하여 원하는 방식대로 네트워크 구성이 가능  
#### iptables의 동작 원리
![iptables_principle](./images/iptables_principle.png)
- PREROUTING(입력패킷) 기능을 통해 POSTROUNTING(출력 패킷)으로 진행
  - 이과정 중 INPUT, OUTPUT, FORWARD의 대한 필터링을 거치게 되며, 이러한 패킷의 대한 필터링으로 차단/허용 가능
- PREROUNTING
  - 외부 네트워크에서 방화벽으로 들어올때 필터링하며 사설(내부)망으로 들어가기전이다
- INPUT
  - 인터페이스를 통해 로컬 프로세스로 들어오는 패킷의 처리
  - INPUT에서 패킷 처리(차단/허용) 후 사용자에게 프로세스로 전달
- OUTPUT
  - 패킷이 외부로 보내지기 전에 로컬에서 생성된 네트워크 패킷 변경 방화벽 내에서 실행
- FORWARD
  - 다른 호스트로 통과시켜 보낼 패킷에 대한 처리(차단/허용)
  - 방화벽이나 IPS 등과 같이 지나가는 패킷을 처리
- POSTROUTING
  - 방화벽에서 외부로 나갈때 필터링하며 방화벽에서 외부로 나가기 전임
#### iptables의 Table와 Chain의 관계
![Table_And_Chain](./images/Table_And_Chain.png)
- IPTABLES는 필터링을 적용하는 테이블
  - 규칙에 따라 패킷을 차단하거나 허용하는 역할
- FILTER Table
  - INPUT, OUTPUT, FORWORD 3개의 다음과 같이 chain이 존재
- NAT Table
  - 필터링기능은 없지만, NAT(주소 변환)용으로 사용
  - 외부에서 내부로 오는 패킷 포워딩과 내부에서 외부로 나갈때 다른 IP주소로 나가게 하는게 가능
- MAGLE Table
  - TTL이나 TOS(type of service)같은 특수규칙을 적용하기 위해 사용
  - 즉, 패킷안에 데이터를 변환 또는 조작을 하기 위한 테이블
- raw Table
  - Filter Table의 Connection Tracking 기능을 좀더 자세히 다를때 사용
  - 특정 네트워크는 연결 추적에서 제외하는등의 추적에서 제외하는 등의 추가 설정이 가능
  - conntrack 모듈보다 우선순위를 가짐
- iptables에는 테이블 안에 체인들이 IP 패킷들에 대해 정해진 규칙에 따라 수행
  - 들어오는 패킷에 대해 허용(ACCEPT), 거부(REJECT), 버림(DROP)을 결정하게 됨

## 방화벽 설정
- 출처: https://archijude.tistory.com/392
### 포트 상태 확인
- 열려있는 모든 포트 표시
```bash
# -n: host명으로 표시 안함 (Show numerical addresses instead of trying to determine symbolic host, port or user names)
# -a: 모든 소켓 표시 (Show both listening and non-listening sockets)
# -p: 프로세스ID와 프로그램명 표시 (Show the PID and name of the program to which each socket belongs)
$netstat -anp
```
- Listen 중인 포트 표시
```bash
$netstat -anp | grep LISTEN
```
- 확인하려는 포트번호 상태 확인
```bash
$netstat -anp | grep <port number>
```
- 특정 호스트 포트 상태 확인
  - 특정 호스트로 접속이 불가할 때, ``netcat(nc)`` 네트워크 유틸리티를 이용하여 포트가 막혀 있는지 확인 가능
  - ``netcat`` 이란?
    - 넷캣(netcat)은 TCP, UDP 프로토콜을 사용하는 네트워크에서 네트워크 연결 상태를 읽거나 쓸때 사용하는 유틸리티 프로그램
    - netcat 설치하기: https://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_nc
  - 특정 포트 상태 확인
    - ``$nc -z <host address> <host port>``
    - ``$nc -z www.google.com 80``
  - 특정 호스트의 포트 범위를 지정하여 열린 포트 확인
    - ``$ nc <host address> -z <start port>-<end port>``
    - ``$ nc 10.20.30.40 -z 19-21``
### 포트 열기
- 리눅스 방화벽 설정 명령어인 iptables을 사용하여 포트를 열수 있음
#### iptables를 이용한 설정
- 방화벽 설정 정보 확인하기
  ``$iptables -nL``
```bash
$sudo iptables -nL
[sudo] password for psw: 
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy DROP)
target     prot opt source               destination         
DOCKER-USER  all  --  0.0.0.0/0            0.0.0.0/0           
DOCKER-ISOLATION-STAGE-1  all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

Chain DOCKER (5 references)
target     prot opt source               destination         
ACCEPT     tcp  --  0.0.0.0/0            172.17.0.2           tcp dpt:3306
ACCEPT     tcp  --  0.0.0.0/0            172.17.0.5           tcp dpt:443
ACCEPT     tcp  --  0.0.0.0/0            172.17.0.4           tcp dpt:443
ACCEPT     tcp  --  0.0.0.0/0            172.17.0.3           tcp dpt:8432
ACCEPT     tcp  --  0.0.0.0/0            172.17.0.3           tcp dpt:6901
ACCEPT     tcp  --  0.0.0.0/0            172.21.0.3           tcp dpt:443

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
target     prot opt source               destination         
DOCKER-ISOLATION-STAGE-2  all  --  0.0.0.0/0            0.0.0.0/0           
DOCKER-ISOLATION-STAGE-2  all  --  0.0.0.0/0            0.0.0.0/0           
DOCKER-ISOLATION-STAGE-2  all  --  0.0.0.0/0            0.0.0.0/0           
DOCKER-ISOLATION-STAGE-2  all  --  0.0.0.0/0            0.0.0.0/0           
DOCKER-ISOLATION-STAGE-2  all  --  0.0.0.0/0            0.0.0.0/0           
RETURN     all  --  0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-ISOLATION-STAGE-2 (5 references)
target     prot opt source               destination         
DROP       all  --  0.0.0.0/0            0.0.0.0/0           
DROP       all  --  0.0.0.0/0            0.0.0.0/0           
DROP       all  --  0.0.0.0/0            0.0.0.0/0           
DROP       all  --  0.0.0.0/0            0.0.0.0/0           
DROP       all  --  0.0.0.0/0            0.0.0.0/0           
RETURN     all  --  0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-USER (1 references)
target     prot opt source               destination         
RETURN     all  --  0.0.0.0/0            0.0.0.0/0 
```
- iptables 명령어 옵션
  - ``-I <chain> [rulenum] <rule-specification>``: 새로운 규칙을 추가    
    - Insert one or more rules in the selected chain as the given rule number.  So, if the rule number is 1, the rule or rules are inserted at the head of the chain. This is also the default if no rule number is specified
  - ``-D <chain> [rulenum]`` or ``-D <chain> <rule-specification> ``: 기존의 규칙을 삭제
    - Delete one or more rules from the selected chain.  There are two versions of this command: the rule can be specified as a number in the chain  (starting  at  1 for the first rule) or a rule to match
  - ``-p``: 패킷의 프로토콜을 명시
  - ``-j``: 규칙에 해당되는 패킷을 어떻게 처리할지를 결정
- 특정포트로 외부에서 접속할 수 있도록 열기 (외부에서 접속할 수 있도록 포트 OPEN)
  - 아래는 외부에서 들어오는(INBOUND) TCP포트 12345의 연결을 받아들인다는 규칙을 1번 방화벽 규칙으로 추가한다는 의미
  - TCP PORT일 경우
    - ``$iptables -I INPUT 1 -p tcp --dport 12345 -j ACCEPT``
  - UDP PORT일 경우
    - ``$iptables -I INPUT 1 -p udp --dport 12345 -j ACCEPT``
- 특정포트로 외부로 나갈 수 있도록 포트 열기
  - TCP PORT일 경우
    - ``$iptables -I OUTPUT 1 -p tcp --dport 9002 -j ACCEPT``
  - UDP PORT일 경우
    - ``$iptables -I OUTPUT 1 -p udp --dport 9002 -j ACCEPT``
- 추가한 설정 조회
  - ``$iptables -L -v``
  - ``-L``: 규칙을 출력
  - ``-v``: 자세히
- 추가한 설정 삭제하는 방법
  - 방법1: 추가한 규칙의 번호로 삭제하는 방법
    - ``$iptables -D INPUT 1``
  - 방법2: 추가했을 때의 명령어에서 "-I"를 "-D"로 바꾸어 주는 방법
    - ``$iptables -D INPUT -p tcp --dport 12345 -j ACCEPT``
    - ``$iptables -D INPUT -p udp --dport 12345 -j ACCEPT``
- 변경사항을 저장하는 방법
  - ``$service iptables save``
  - ``$/etc/init.d/iptables restart``
- 방화벽 활성화 및 비활성화
  - 활성화
    - ``$/etc/init.d/iptables start``
  - 비활성화
    - ``/etc/init.d/iptables stop``
#### firewalld를 사용한 설정
- firewalld 의 필요성
  - 기존 iptables의 한계
    - 룰 변경시 서비스 중지 및 설정 변경
    - 오픈스택이나 KVM과 같은 가상화 호스트에서는 네트워크 변 화가 수시로 발생되므로 필터링 정책에 변경이 필요
    - 응용프로그램 자체에서 필터링 정책을 구성하는 경우 iptables 정책과 충돌되는 등의 문제 야기
  - firewalld가 필요한 이유?
    - KVM , openstack 과 같은 가상화, 클라우드 환경하에서의 필 터링 정책 동적 추가 가능
    - DBUS API를 통한 정보 공유를 통해 정책 충돌 문제 해결
  - DBUS란?
    - 어플리케이션간의 통신을 지원하는 인터페이스
- firewalld vs iptables
  - https://blog.asamaru.net/2015/10/16/centos-7-firewalld/    
```bash
[root@te-kafka ~]# iptables -nL | grep 9092
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:9092 ctstate NEW,UNTRACKED
[root@te-kafka ~]# firewall-cmd --permanent --zone=public --add-port=9093/tcp
success
[root@te-kafka ~]# firewall-cmd --reload
success
[root@te-kafka ~]# firewall-cmd --permanent --zone=public --add-port=2182/tcp
success
[root@te-kafka ~]# firewall-cmd --reload
success
[root@te-kafka ~]#
```
## xclip 프로그램
- 파일에 있던 내용을 복사하는 경우 마우스로 드래그하고 복사할 수 있지만, 잘못 복사하는 경우를 방지하거나 자동화하고 싶은 경우에 xclip 명령어를 사용
- 설치
```bash
sudo apt update
sudo apt install -y xclip
```
- 시나리오 1: 파일의 내용을 클립보드에 저장하는 방법
```bash
$xclip -sel clip < 파일명
$xclip -sel clip < ~/.ssh/id_rsa.pub
```
- 시나리오 2: 명령어 결과를 클립보드에 저장하는 방법
```bash
$명령어 | xclip -sel clip
$tail -n 30 /var/log/syslog | xclip -sel clip
```
- 시나리오 3: 현재 클립보드의 내용을 저장하고 싶은 경우
```bash
xclip -o -sel clip > 파일명
xclip -o -sel clip > config
```
## How to Bring Background Process to Foreground in Linux
- Backgound로 프로세스 동작시키기
```bash
$ vi &
$ crontab &
$ jobs
[1]-  Running                 vi &
[2]+  Running                 crontab &
```
- Bring last background process to foreground
  - If you run fg command without any options or arguments then it will bring back the last process that was pushed to background.
```bash
$ fg
```
- Bring specific process to foreground
  - If you want to bring a specific process to foreground, then mention its job id after fg command. Here is an example to bring the first process, that is, job id 1, to foreground.
```bash
$ fg 1
```
## 압축 및 압축 해제 명령어
- tar 파일 압축
  - ``tar -cvf [압축파일명] [압축할 파일 혹은 폴더 경로]``
```bash
# 현재 디렉토리의 data 디렉토리 전체를 data.tar 파일로 압축
$tar -cvf data.tar data/
```
- tar 파일 해제
  - ``tar -xvf [압축파일명]``
```bash
# 현재 디렉토리에 압축 파일을 해제
$tar -xvf data.tar
```
- tar.gz 파일 압축
  - ``tar -zcvf [압축파일명] [압축할 파일 혹은 폴더 경로]``
```bash
# 현재 디렉토리의 data 디렉토리 전체를 data.tar 파일로 압축
$tar -zcvf data.tar.gz data/
```
- tar.gz 파일 해제
  - ``tar -zxvf [압축파일명]``
```bash
# 현재 디렉토리에 압축 파일을 해제
$tar -zxvf data.tar.gz
```