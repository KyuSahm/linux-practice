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