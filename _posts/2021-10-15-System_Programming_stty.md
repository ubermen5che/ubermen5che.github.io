---
layout : posts
comments : true
title: "[System Programming]Connection Control Studying-stty"

categories:
  - Unix/Linux
  - System Programming
  - kernel
tags:
  - C
  - System Programming
  - stty
---


## Ideas and Skills

- Files과 devices의 유사점
- Files과 devices의 차이점
- connections에 대한 속성
- Race condition과 atomic operations
- device drivers를 제어하는방법
- Streams

# 5.2 Devices Are Just Like Files

- Unix시스템에서 모든 devices는 file로써 다루어진다.
    - device도 file과 마찬가지로 filename, inode number, owner, set of permission bits, last modifed time의 정보를 가지고 있다.
- 우리가 알고있는 파일에 적용되는 작업들(ex : cat, more...)은 terminals과 모든 devices들에 적용된다.


    **5.2.1 Devices Have Filenames**

    - devices파일들은 /dev directory에 존재한다.
        - lp* files : printers
        - fd* files : floppy-disk drives
        - sd* files : SCSI drives 파티션
        - tape file : tape driver 백업파일
        - tty* files : terminals
        - dsp file : 사운드 카드 연결 파일


    **5.2.2 Devices and System Calls**

    - Devices는 file과 연관된 system calls을 제공한다.
        - open, read, write, leeek, close, stat
    - magnetic tape로부터 데이터를 읽어들이는 코드
        -

        ```c
        int fd;
        fd = open("/dev/tape", O_RDONLY);
        lseek(fd, (long)4096, SEEK_SET);
        n = read(fd, buf, buflen);
        close(fd);
        ```

    - 몇몇의 devices는 모든 file operations을 제공하지않는다
        - /dev/mouse file은 write() operation을 제공하지 않음.
        - Terminals은 read(), write()를 지원하지만, lseek()은 지원하지 않는다.

        **5.2.2-1 Terminals Are Just Like Files**

        - Terminal은 키보드와 display unit처럼 행동하는 어떠한 파일이다.
            - telnet, ssh...
        - tty 커맨드는 현재 사용하고있는 terminal을 표시해준다.
        - Terminal에서도 파일에 대한 연산을 그대로 사용할 수 있다.
            - cp, >, mv, ln, rm, cat, ls

## 5.2.3 Properties of Devices file

- Device files도 disk files이 가지고 있는 대부분의 properties들을 가지고있다.
    - inode : kerne에 있는 device driver의 포인터를 가짐.
    - Device driver : 데이터를 읽어들이는 kernel안에 있는 subroutine
    - Major # : 어떤 subroutine이 실제 디바이스를 다루는지 명시함
    - Minor # : subroutin에 넘겨지는 값

## 5.2.4 Device Files and Inodes

- How read works?
    - Kernel은 file descriptor에 대한 inode를 찾는다.
        - file descriptor에 대한 inode를 어디서 찾지?
            - file descriptor는 kernel's global file table의 entry 포인터를 가지고있다(아래 그림 참고)
            - global file table에는 inode정보, offset, access restriction등에 대한 정보를 담고있음.

            ![fd.png](/assets/images/posts/2021-10-15/fd.png){: .align-center}{: width="100%", height="100%"}

        - Inode는 kernel에게 파일의 type을 알려준다.
        - 어떻게 그러면 파일을 read할까?
    - 만약에 해당하는 file이 device file이라면 kernel은 해당 device의 read()에 해당하는 drivce code중 일부를 호출함으로 써 파일을 읽어들인다. (driver마다 read코드는 다르겠지?)
- 위의 로직과 유사한 opration들
    - open, write, lseek, close
