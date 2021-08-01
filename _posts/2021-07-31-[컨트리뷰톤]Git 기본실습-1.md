---
title: "[컨트리뷰톤] Git/GitHub 기본실습-1"

categories:
  - Git
  - 대외 활동
tags:
  - Git/Github
  - git 명령
---

# 오픈소스 프로젝트 복사 / 다운 받기(Fork & Clone)

우선 Fork하는 행위에 대한 의미를 알아보자. Fork를 이해하려면 우선 commit의 기본개념을 알아야한다.

### 커밋이란?

> 소스파일의 변화분이다.

![commit_log.png](/assets/images/posts/2021-07-31/commit_log.png){: .align-center}{: width="100%", height="100%"}

{:.image-caption}
*그림1(커밋 로그)*

Fork는 Fork한 프로젝트의 commit내역을 모두 복사한것과 동일하다고 생각하면 쉽다. 즉 내가 fork한 프로젝트와는 별개의 복사된 프로젝트라고 보면된다. 복사된 프로젝트이니 당연히 fork한 프로젝트를 수정한다고 해서 원본 프로젝트에 대해 전혀 영향을 줄 수 없다! 만약 내가 fork해온 오픈소스 프로젝트에 대해 버그를 발견하거나 내용추가가 필요한 경우 어떻게 원본 프로젝트에 반영할 수 있을까? 해당내용은 뒤에서 다루도록 하겠다.

![fork_clone.png](/assets/images/posts/2021-07-31/fork_clone.png){: .align-center}{: width="100%", height="100%"}

{:.image-caption}
*그림2(fork와 clone)*

# 개발자가 오픈소스를 읽는 방법(Git project Reading Skill)

## git shortlog

내가 fork해온 오픈소스 프로젝트에서 누가 제일 개발을 많이한지 분명 궁금할 것이다 이럴 때 사용할 수 있는 명령어는 아래와 같다.

```bash
git shortlog -sn | nl
```


![shortlog_sn.png](/assets/images/posts/2021-07-31/shortlog_sn.png){: .align-center}{: width="100%", height="100%"}

{:.image-caption}
*그림3(git shortlog -sn)*

- git shortlog -s(개발자별 commit 개수 요약)
- git shortlog -n(개발자별 commit 개수 순위 정리)

 내가 이번 3개월의 기간동안 컨트리뷰톤을 참여하면서 기여하게될 프로젝트인 "FOSSLight" 프로젝트에 대해서 해당 명령어를 사용해보았다. 그 결과 위와 같은 결과가 나왔다 이 결과는 단지 commit결과의 수를 기준으로 요약한 결과이다.

```bash
git shortlog -sn --no-merges
```

git shortlog 사용시 —no-merges 옵션을 사용하면 Merge commit을 제외한 commit수를 산정할 수 있다.

여기서 Merge commit이란 별다른 수정내역이 없는 commit이다. 단지 다른 사람이 commit한것을 merge했다라는 표시만 남기는 의미를 가지는 commit이다. 따라서 —no-merges 옵션을 사용하면 순수히 코드 변경과 관련된 커밋수만 확인해볼 수 있는것이다.

## git log

전체 소스파일수정 내역(commit)은 몇개인지 알아보고자 할 때 사용하는 명령어는 다음과 같다.

```bash
git log --oneline | wc -l
```

참고로 wc -l 명령은(파일) 라인수 개수를 측정해주는 명령어이다. 또한 명령에 사이에 위치한 bar는 파이프라인 명령어 기호로 파이프라인 기호 앞에 위치하는 명령어의 결과를 파이프라인 기호 뒤의 명령어의 입력으로 넣어주는 기능을하는 명령어이다. 추가적인 내용은 linux command line 관련 문서를 참고하면 좋을듯하다.

git log를 이용하면 commit 각각의 ID를 확인해볼 수 있다.


```bash
git log --oneline
```

![git_log_oneline.png](/assets/images/posts/2021-07-31/git_log_oneline.png){: .align-center}{: width="100%", height="100%"}

{:.image-caption}
*그림3(git log —oneline)*

여기서 노란색으로 표시된 해쉬값이 ID이다. 각 ID는 SHA1 방식으로 해싱된 값으로 고유한 ID를 갖는다. 위의 커밋들 중에서 소스파일 수정내역(commit)dmf 확인해보고 싶다면 git show 명령어를 사용해보자.

```bash
git show <ID>
```

![git_show.png](/assets/images/posts/2021-07-31/git_show.png){: .align-center}{: width="100%", height="100%"}

{:.image-caption}
*그림4(git show)*

필자는 6237748이라는 ID를 가지는 commit에 대해 git show를 하였고 위 그림4는 이에 대한 결과를 나타낸다. git show를 이용하면 해당 커밋에 대해서 수정한 소스파일 개수가 얼마나 되는지 알아볼 수 있다.

![git_show_diff.png](/assets/images/posts/2021-07-31/git_show_diff.png){: .align-center}{: width="100%", height="100%"}

{:.image-caption}
*그림5(소스파일 수정 개수)*

그림5를 보면 grep명령어를 이용해서 "diff —git"이라는 문자열을 검색해본것을 볼 수 있다. diff-git이 소스파일의 수정을 뜻하는 의미인것 같다. 결과를 보면 총 4가지 파일이 수정된것을 확인할 수 있다.

- PartnerMaster.java → PartnerMaster.java
- PartnerMapper.java → PartnerMapper.java
- PartnerServiceImpl.java → ParterServiceImpl.java
- PartnerMapper.xml → PartnerMapper.xml

결과를 자세히 보면

**diff --git a/src/main/java/oss/fosslight/domain/PartnerMaster.java b/src/main/java/oss/fosslight/domain/PartnerMaster.java**

이러한 결과를 볼 수 있는데, diff 파일경로의 가장 앞에 나타나는 a와 b가 의미하는 바는 상태a에서 상태b로 변경되었다는것을 표시하는 문자이고 두 상태 모두 같은 경로의 파일을 나타내고있다. 이는 a → b상태에서 경로 및 파일명 변경이 일어나지 않았고 오직 PartnerMaster.java 파일 내부의 소스코드만이 변경되었다는것을 뜻한다. 나머지 3개의 파일에 대해서도 동일한 결과를 보여주고 있다.

### 특정 기준 수정내역(commit) 리스트 확인 방법

- 특정 소스파일 기준 수정내역 리스트 확인

    ```bash
    git log --oneline -- <파일명>
    ```

- 특정 날짜기준 수정내역 리스트 확인

    ```bash
    #2020년 1월 부터 2020년 6월30일까지 소스 수정내역(commit) 리스트 확인
    git log --oneline --after=2020-01-01 --before=2020-06-30
    ```

- 특정 날짜+파일기준 수정내역 리스트 확인

    ```bash
    git log --oneline --after=2020-06-01 --before=2020-06-30 -- <파일명>
    ```

- 옛날것부터 소스파일 수정내역 보기

    ```bash
    git log --reverse
    ```

    - 최초 커밋을 확인할 때 유용.

    **Primary Notice:** 해당 포스트는 리얼리눅스 송태웅님의 강의를 보고 정리한 내용임을 알립니다.
    {: .notice--primary}
