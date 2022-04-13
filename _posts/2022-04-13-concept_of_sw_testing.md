---
layout : posts
comments : true
title: "Concepts of SW testing"

categories:
  - SW testing
tags:
  - Debugging
  - Testing
  - Coverage criteria
---

# Testing & Debugging

- **Testing** : 실행하고 관찰함으로써 소프트웨어를 평가하는 것.
- Test failure : software failure를 초래하는 test에 대한 실행
- **Debugging** : failure가 주어질 때 fault를 찾는 과정

# Fault & Failure Model(RIPR)

> Four conditions necessary for a failure to be observed

1. Reachability : fault가 포함된 프로그램 위치에 반드시 도달해야한다.
2. Infection : 프로그램에 대한 상태가 반드시 부정확해야한다.
3. Propagation : fault로 인해 영향을 받은 상태가 반드시 incorrect final state 및 output을 유발해야한다.
4. Reveal : tester는 incorrect portion of program state를 반드시 관찰해야한다.

![Untitled](/assets/images/posts/2022-04-13/Untitled.png)

# Coverage Criteria

> Test 탐색공간 범위를 정하는 기준

- 규모가 매우 작은 프로그램이라도 모든 test경우를 전부 탐색해보기 위해서는 매우많은 inputs을 가져야한다.
- Tester들은 막대한 input space를 **탐색**해야한다.
    - **최소**의 input들로 **최대한 많은** problem을 찾기 위해 애써야한다.
- Coverage criteria는 구조화되고, 합리적인 방법으로 input space를 탐색하도록 도움을 준다.

## Advantages of Coverage Criteria

- 효율성과 효용성을 극대화할 수 있다.
- 테스트들을 위한 software artifacts(Source, requirements, design models...)로부터 traceability를 제공한다.
- regression testing을 더 쉽게 만들어 준다.
    - regression testing이란 프로그램이 완성되고 사용하다가 버그가 발생해서 버그 수정을 위해 코드 수정시 testing을 다시하는 방법
        - test case를 다시 만들 필요가 없고 기존 test case에 조금 더 추가해서 재사용
        - Coverage criteria 덕분에 testing을 언제까지 해야하는지 알 수 있다
- 테스터들에게 stopping rule을 제공한다.
