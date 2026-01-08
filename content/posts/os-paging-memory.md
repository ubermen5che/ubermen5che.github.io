+++
title = '운영체제 Paging과 메모리 관리'
date = 2026-01-08T22:55:00+09:00
draft = false

categories = ["CS", "OS"]
tags = ["Paging", "Memory", "Kernel"]
+++

운영체제(OS)에서 메모리 관리는 시스템 성능과 안정성에 직결되는 핵심 주제임. 오늘은 그 중에서도 현대 OS의 표준 메모리 관리 기법인 **Paging(페이징)**에 대해 알아보겠음.

## 1. Paging이란 무엇인가?

Paging은 **외부 단편화(External Fragmentation) 문제를 해결하기 위해 고안된 메모리 관리 기법임.**

과거에는 프로세스를 물리 메모리에 **연속적(Contiguous)**으로 할당해야 했음. 이로 인해 메모리 중간중간에 작은 빈 공간들이 남게 되어, 공간의 합은 충분하지만 실제로는 큰 프로세스를 할당할 수 없는 외부 단편화 문제가 발생했었음.

Paging은 이 고정관념을 깬 방식임.
- **Physical Memory**를 **Frame**이라는 고정 크기 블록으로 나눔.
- **Logical Memory (Process)**를 **Page**라는 같은 크기의 블록으로 나눔.
- 프로세스의 Page들을 물리 메모리의 비연속적인 Frame들에 흩뿌려서 할당함.

## 2. Page Table: 주소 변환의 핵심

프로세스의 데이터가 물리 메모리 여기저기에 흩어져 있다면, CPU는 어떻게 올바른 주소를 찾아갈까? 여기서 **Page Table**이 등장함.

CPU가 바라보는 **Logical Address**는 두 부분으로 나뉨:
- **Page Number (p)**: Page Table의 인덱스로 사용됨.
- **Page Offset (d)**: 페이지 내에서의 변위임.

**주소 변환 과정**:
1. CPU가 Logical Address `(p, d)`를 생성함.
2. Page Table에서 `p`에 해당하는 항목을 찾아 **Physical Frame Number (f)**를 얻음.
3. 이를 Offset `d`와 결합하여 Physical Address `(f, d)`를 완성함.

> **참고**: 각 프로세스마다 별도의 Page Table을 가지며, OS는 이를 관리하기 위해 **PTBR (Page Table Base Register)**를 사용하여 문맥 교환 시 테이블을 교체함.

## 3. Fragmentation (단편화)

### 외부 단편화 (External Fragmentation) 해결
Paging을 사용하면 물리 메모리의 어떤 빈 Frame이라도 할당해줄 수 있기 때문에, 외부 단편화는 완벽하게 해결됨.

### 내부 단편화 (Internal Fragmentation) 발생
하지만 **내부 단편화**라는 새로운 문제가 발생함.
- 프로세스의 크기가 Page 크기의 배수가 아닐 때, 마지막 Page의 남은 공간은 낭비됨.
- **최악의 경우**: 프로세스가 `Task Size = n * Page Size + 1 Byte`를 요구한다면, 마지막 1 Byte를 저장하기 위해 새로운 4KB(일반적인 Page Size) 페이지 전체를 할당해야 함. 결과적으로 `Page Size - 1 Byte` 만큼의 메모리가 낭비되는 셈임.

## 4. Page Table의 구조적 문제와 개선

현대 OS(64bit)에서 Page Table 자체의 크기는 매우 큼. 모든 주소 공간을 하나의 통짜 테이블로 관리하는 것은 비효율적임. 이를 해결하기 위한 기법들을 정리해봄.

### 4.1 계층적 페이징 (Hierarchical Paging)
Page Table 자체를 다시 Page 단위로 쪼개는 방식임.
- **Multi-level Paging**: Page Table을 여러 단계(Page Manager, Page Middle, Page Table 등)로 나누어, 사용되지 않는 영역에 대한 테이블은 아예 생성하지 않음으로써 메모리를 절약함.
- Linux Kernel(x86_64)은 **4-level** 또는 **5-level paging**을 사용함.

### 4.2 TLB (Translation Lookaside Buffer)
Page Table은 메인 메모리(RAM)에 위치함. 따라서 CPU가 데이터에 접근하려면 최소 2번의 메모리 접근(Page Table 접근 1회 + 실제 데이터 접근 1회)이 필요하여 속도가 느려짐.

이를 해결하기 위해 **TLB**라는 특수 고속 하드웨어 캐시를 사용함.
- 자주 사용하는 `Page Number -> Frame Number` 변환 정보를 TLB에 저장함.
- CPU는 먼저 TLB를 확인하고(TLB Hit), 없을 경우에만 메모리의 Page Table을 참조(TLB Miss)하여 성능 저하를 막음.

---

Paging은 현대 OS가 효율적으로 메모리를 가상화하고 보호하기 위한 필수적인 메커니즘임. 이를 이해하는 것은 시스템의 성능 저하 원인(TLB Thrashing 등)을 분석하거나 커널 레벨의 최적화를 수행하는 데 중요한 기초가 됨.
