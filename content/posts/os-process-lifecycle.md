+++
title = '프로세스 라이프사이클과 스케줄링 심층 분석'
date = 2026-01-09T00:15:00+09:00
draft = false

categories = ["CS", "OS"]
tags = ["Process", "Scheduling", "Kernel", "Context Switch"]
+++

이전 포스팅에서는 프로세스의 개념과 구조에 대해 다뤘음. 이번에는 **프로세스의 생성부터 종료까지의 라이프사이클**과, 이를 관리하는 **스케줄링 메커니즘**을 커널 레벨의 관점에서 아주 깊이 있게 낱낱이 파헤쳐보겠음.

## 1. 프로세스의 메모리 구조와 상태

실행 중인 프로세스는 단순히 코드의 집합이 아님. 메모리 상에서 다음과 같은 독자적인 공간을 점유함.

*   **Text (Code) Section**: 실행할 기계어 코드가 저장되는 영역. 읽기 전용으로 공유 가능함.
*   **Data Section**: 전역 변수(Global Variable) 및 정적 변수(Static Variable)가 저장됨.
*   **Heap Section**: 런타임 중에 동적으로 할당되는 메모리 (`malloc`, `new`).
*   **Stack Section**: 함수 호출 시의 임시 데이터(지역 변수, 매개변수, Return Address) 저장.
*   **PCB (Process Control Block)**: 각 프로세스의 메타데이터를 담고 있는 커널 자료구조. **Program Counter (PC)**, 레지스터 상태 등을 포함하여 Context Switching의 핵심이 됨.

> **단일 처리기(Single Processor)의 철칙**: 단일 코어 시스템에서는 **어떤 순간에도 실행 중인 프로세스는 오직 1개**임. 나머지 프로세스들은 CPU 스케줄러가 자신을 선택(Dispatch)해 줄 때까지 얌전히 대기해야 함.

## 2. 프로세스 큐 (Process Queues)

프로세스가 시스템에 진입하면 그 상태에 따라 다양한 큐(Queue)를 이동함. 커널은 이 큐들을 일반적으로 **Linked List** 형태로 관리함.

1.  **Job Queue**: 시스템에 들어온 모든 프로세스의 집합.
2.  **Ready Queue**: 메인 메모리에 상주하며, **즉시 실행 가능(Runnable)**한 상태로 CPU 할당을 기다리는 프로세스들의 집합.
3.  **Device Queue (Waiting Queue)**: I/O 장치의 처리를 기다리거나, 특정 이벤트(Interrupt, Child Process 종료)를 기다리는 큐.

**프로세스의 여정**:
- 스케줄러에 의해 Ready Queue에서 선택되어 CPU를 잡고 실행됨 (**Dispatch**).
- 실행 도중 I/O 요청 시 Device Queue로 이동.
- I/O 완료 시 다시 Ready Queue로 복귀.
- `fork()` 자식 생성 시 자식 종료를 기다리며 Wait Queue로 이동 가능.
- 할당 시간(Time Slice) 만료 시 인터럽트에 의해 다시 Ready Queue로 강제 이동.
- 이 과정은 프로세스가 종료(`exit`)되어 자원을 반납할 때까지 반복됨.

## 3. 스케줄러 (Scheduler) 분류

스케줄러는 실행 빈도와 목적에 따라 크게 두 가지로 나뉨.

### 3.1 단기 스케줄러 (Short-term Scheduler)
- **CPU Scheduler**라고도 불림.
- **역할**: Ready Queue에 있는 프로세스 중 하나를 선택하여 CPU를 할당함.
- **특징**: CPU 효율성을 위해 **매우 빈번하게 실행(ms 단위)**되어야 함. 따라서 스케줄링 알고리즘이 느리면 시스템 전체 성능이 저하됨.

### 3.2 장기 스케줄러 (Long-term Scheduler)
- **Job Scheduler**라고도 불림.
- **역할**: Disk Pool에 있는 프로세스들을 골라 메모리(Ready Queue)로 적재함.
- **특징**: 단기 스케줄러에 비해 실행 빈도수가 훨씬 적음 (분, 초 단위).
- **중요성**: 단기 스케줄러는 I/O 바운드 프로세스(입출력 위주)와 CPU 바운드 프로세스(연산 위주)를 적절히 섞어서 선택해야만 시스템 효율이 극대화됨. 장기 스케줄러가 이 **Degree of Multiprogramming (다중 프로그래밍 정도)**를 조절하여 전체 시스템의 균형을 맞춤.

## 4. 문맥 교환 (Context Switching)

인터럽트가 발생하거나 스케줄링 시점이 오면 **문맥 교환**이 발생함.
이는 현재 실행 중인 프로세스의 상태(Context)를 저장하고, 다음 실행할 프로세스의 저장된 상태를 복구하는 과정임.

1.  **Save Concept**: 현재 CPU 레지스터 값(PC, SP, GPR 등)을 현재 프로세스의 **PCB**에 저장.
2.  **Restore Context**: 다음 실행할 프로세스의 PCB에서 레지스터 값을 복원.
3.  **Overhead**: 문맥 교환이 일어나는 동안 CPU는 아무런 유용한 작업을 하지 못함. 따라서 이는 **순수 오버헤드(Pure Overhead)**임.
4.  **Optimization**: 현대 하드웨어는 여러 레지스터 세트를 제공하거나 TLB Flush를 최소화하는(ASID 사용) 등 오버헤드를 줄이기 위해 노력하지만, 커널 레벨에서도 불필요한 스위칭을 최소화하는 것이 성능의 핵심임.

## 5. 프로세스 생성과 종료 (Creation & Termination)

### 프로세스 생성 (Process Creation)
리눅스에서 프로세스 생성은 주로 `fork()` 시스템 콜을 통해 이루어짐.
- **`fork()`**: 부모 프로세스를 똑같이 복제하여 자식을 생성함. 부모의 주소 공간을 **COW (Copy-On-Write)** 방식으로 공유하다가 쓰기 발생 시 분리함.
- **`exec()`**: `fork()` 후 자식 프로세스의 메모리 공간을 새로운 프로그램의 바이너리로 덮어씌움.
- **트리 구조**: 모든 프로세스는 `init` (PID 1) 프로세스를 루트로 하는 트리 구조를 형성함.

### 프로세스 종료 (Process Termination)
- **`exit()`**: 프로세스가 마지막 문장을 실행하고 OS에 종료를 알림. 모든 자원(메모리, 파일 디스크립터)을 반납함.
- **`wait()`**: 부모 프로세스는 자식의 종료 상태(Status Code)를 회수해야 함.
- **좀비(Zombie)와 고아(Orphan)**:
    - **Zombie**: 자식이 종료되었으나 부모가 `wait()`를 호출하지 않아 PID와 종료 상태가 남아있는 경우.
    - **Orphan**: 자식보다 부모가 먼저 종료된 경우 `init` 프로세스가 입양하여 처리함.
