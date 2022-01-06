---
layout : posts
comments : true
title: "[컨트리뷰톤] Git/GitHub 기본실습-2"

categories:
  - Git
  - 대외 활동
tags:
  - Git/Github
  - git 명령어
  - rebase
  - 오픈소스
  - branch
---

# 오픈소스 개발참여를 위한 commit 작업(branch & commit)

## Branch란?

브랜치는 다음과 같은 표현으로 간결하게 설명할 수 있다.

> "같은 폴더 다른세상"

아래의 터미널을 보면  check-fosslight branch에서 hi.txt를 생성하고 commit을 한것을 볼 수 있다. 해당 브랜치에서 생성하고 commit한 파일을 main branch에서 찾아볼 수 있을까? 지난 포스팅에서 배운 git log 명령어를 이용하여 commit내역을 보면 확실하게 알 수 있을것이다!

![branch.png](/assets/images/posts/2021-08-06/branch.png){: .align-center}{: width="100%", height="100%"}

{:.image-caption}
*그림1(main외에 브랜치에서 hi.txt생성)*

파이프라인을 명령어를 사용하여 HEAD -1을 하면 가장 상위 1개의 commit내역을 가져올 수 있다.

아래의 사진을 통해 결과를 확인해보자. check-fosslight branch에는 hi.txt파일을 추가한 commit내역을 확인할 수 있다. **그러나 main branch에서는 확인할 수 없었다.**

즉, **같은 폴더 다른세상**이라는 표현이 아주 적절한 표현임을 알 수 있었다.

```bash
git log --oneline | HEAD -1
```

![checkout.png](/assets/images/posts/2021-08-06/checkout.png){: .align-center}{: width="100%", height="100%"}

### 브랜치 관련 명령어 모음
---
**브랜치 생성**

```bash
git checkout -b <브랜치 이름>
```

**브랜치 삭제**

```bash
#삭제하고자 하는 브랜치로 이동
git checkout <브랜치 이름>
git branch -D <브랜치 이름>
```

**현재 프랜치 확인 및 소스파일 상태확인**

```bash
$ git status
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

- 현재는 nothing to commit(수정내역이 없음)

## 소스파일 수정 후 최신역사와의 차이점 확인

```bash
#소스파일 수정 확인 명령어
git diff
```

아래의 사진을 보면 빨간색으로 표시된 부분은 이전 역사와 달리 코드가 삭제된 부분을 나타내고, 초록색으로 나타는 부분은 새롭게 추가된 코드를 보여준다. 코드를 수정하고 commit하기 전에 이전과 달리 어느부분을 수정했는지 한눈에 파악하기 쉽게 만들어주는점에서 아주 유용한 명령어인것 같다.

![git_diff.png](/assets/images/posts/2021-08-06/git_diff.png){: .align-center}{: width="100%", height="100%"}

## stash와 checkout

### Stash

> 임시저장소에 현재까지의 변경사항을 넣어두자!

```bash
git stash
```

stash를 설명하기에 앞서 stash를 적용해볼 수 있는 적절한 시나리오를 생각해보자.

현재 내가 프로젝트내에 어떤 부분에 버그를 수정하기 위해 소스코드를 수정하였지만  commit으로 만들기는 아직 많은 테스트가 필요하기때문에 애매하고 버그 수정전과 후를 비교해보고싶은 경우라고 가정해보자. 이 때 적절하게 사용할 수 있는 명령어가 stash이다. stash를 이용하면 수정한 코드들을 임시 저장함으로써 수정전의 상태로 돌아갈 수 있다. 다시 버그 수정후의 역사로 돌아가고싶다면 임시저장한 코드를 stash pop을 이용해서 원래상태로 돌아갈 수 있다.

```bash
git stash pop
```

![stash.png](/assets/images/posts/2021-08-06/stash.png){: .align-center}{: width="100%", height="100%"}

### Checkout

> .git에 저장되어 있는 최신역사를 꺼낸다!

```bash
git checkout -- <특정 파일>
#ex) git checkout -- README.md
```

checkout은 clone받은 프로젝트 폴더내에 숨김파일로 존재하는 .git폴더에 저장되어 있는(가장 최근 프로젝트 정보를 지니고있다) 프로젝트 log를 꺼내오는 명령어이다. 코드를 수정하다가 도저히 해결하기 어려운 버그를 오류를 만났을 때 깔끔하게 예전상태로 돌아가고싶다면 checkout만한 명령어가 없을것이다. 가장 최근의 기준 프로젝트 로그를 가지고있기 때문에 그냥 .git에서 꺼내오고 복구시키면 그만이기 때문이다.(undo 역할을 함) 브랜치를 변경할 때도 checkout을 사용한다. 같은 맥락으로 .git에 브랜치 정보가 저장되어있어 이것을 꺼내오는것이다.

## Add 명령 취소 및 commit 삭제 명령어

### Add 취소

```bash
git reset
```

### Commit 취소 명령어

```bash
#HEAD~1은 가장 위에서 첫번째 내용을 삭제한다는 의미
git reset --hard HEAD~1
```

**hard, soft 옵션의 차이점**

- hard의 경우 commit 내역 및 정보를 모두 삭제함
- soft는 커밋 내역만 삭제하고 코드 변화분은 그대로 남음
    - 여러개의 commit을 합칠때 유용하게 사용됨

### Commit 수정 명령어

```bash
git commit --amend
```
ammend를 사용하여 commit message를 수정할 수 있다. 아래 사진을 자세히 보면 commit ID가 변경된것을 볼 수 있다. 이전 파일 정보가 하나라도 변경됐을경우 commit ID는 변경이 된다.

![before_ammend.png](/assets/images/posts/2021-08-06/before_ammend.png){: .align-center}{: width="100%", height="100%"}

{:.image-caption}
*ammend 전*

![after_ammend.png](/assets/images/posts/2021-08-06/after_ammend.png){: .align-center}{: width="100%", height="100%"}

{:.image-caption}
*ammend 후*

**Primary Notice:** 해당 포스트는 리얼리눅스 송태웅님의 강의를 보고 정리한 내용임을 알립니다.
{: .notice--primary}
