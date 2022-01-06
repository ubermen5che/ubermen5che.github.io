---
title: "[컨트리뷰톤] 오픈소스 협업을 위한 Git/Github 고급과정-1"

categories:
  - Git
  - 대외 활동
tags:
  - Git/Github
  - git 명령어
  - rebase
  - 오픈소스
---

# Rebase의 필요성

평소 개인프로젝트를 할때 당연히 혼자만 commit을 하게된다. 또한 학교에서 팀프로젝트를 하게되는 경우 다수의 팀원들과 git을 이용하여 프로젝트를 진행하지만 모두가 push권한을 가지고 하는경우가 많은것 같다. 이는 협업이 아니며 분업이라고 해야할 것이다. 오픈소스 프로젝트 특성상 다수의 사람이 PR을 날리게 된다.

만약 내가 수정하고 있는 모듈과 다른사람이 수정한 모듈에 연관관계가 있는경우, 내가 PR한것 말고 다른 contributor가 날린 PR이 먼저 merge된다면 내가 fork한 프로젝트는 예전 base(이전 commit 목록)가 되기때문에 충돌이 발생한다. 이때 필요한것이 rebase인데 충돌이 어떻게 해결되는지 원리를 이해하기 위해서는 rebase 명령어에 대한 자세한 사항을 알아둘 필요가 있다. 우선 rewind에 대해 알아보자.

---
# Commit 과거 시점으로 되감기(rewind)

아래 명령어는 commit을 과거 시점으로 되감기위해 사용하는 명령어이다. -i 옵션은 interactive를 의미한다.

```bash
git rebase -i --root
```

아래 그림은 명령어의 결과를 보여준다. 자세히 보면 commit ID와 commit message가 보이는데 여기서 주의해야할점은 git log 명령어의 순서와는 반대인것이다. git log는 가장 최근 commit이 상단에 위치하는데 반해서 rebase명령어는 가장 오래된 commit을 최상단에 보여준다. 그림1과 그림2를 보면 정말로 그런것을 볼 수 있다.

![rebase_i.png](/assets/images/posts/2021-08-01/rebase_i.png){: .align-center}{: width="100%", height="100%"}

{:.image-caption}
*그림1(rebase -i)*

![git_log.png](/assets/images/posts/2021-08-01/git_log.png){: .align-center}{: width="100%", height="100%"}

{:.image-caption}
*그림2(git log —oneline)*

필자는 두번째로 가장 오래된 commit 시점으로 rewind를 해볼것이다.

**Rewind 순서**

1. git rebase -i —root
2. 되감고 싶은 시점에 해당하는 commit에 가장 앞에 위치한 pick을 edit으로 바꾼다.

![rebase_i.png](/assets/images/posts/2021-08-01/rebase_i.png){: .align-center}{: width="100%", height="100%"}

{:.image-caption}
*그림2(git rebase_i)*

![rebase_i_result.png](/assets/images/posts/2021-08-01/rebase_i_result.png){: .align-center}{: width="100%", height="100%"}

{:.image-caption}
*그림2(git rebase -i 결과)*


그럼 다음과 같은 결과 메세지를 볼 수 있다. 864cf0f... ID를 가지는 commit 시점으로 바뀐것이다.

정말로 그런지 git log —oneline으로 확인해보자!!!

![rewind_result.png](/assets/images/posts/2021-08-01/rewind_result.png){: .align-center}{: width="100%", height="100%"}

{:.image-caption}
*그림2(rewind 결과)*


정말로 해당 시점으로 되감아진것을 확인할 수 있다. 마치 타임머신처럼 기능을 한다🤩

타임머신을 탔는데 다시 돌아올 방법이 없다면 정말 곤란할 것 같다... 다행히 rebase명령어는 다시 이전 시점으로 돌아올 수 있는 옵션을 제공한다.

```bash
git rebase --continue
```

---
# Rebase 실습1

## 가장 오래된 역사 부터 두번째 commit 이후에 새로운 commit 3개 넣기

위에 rewind에 관한 내용을 잘 숙지했다면 어려운 내용은 아닌것 같다.

가장 오래된 역사 두번째 commit으로 rewind후 commit을 3개 만들면 되는 간단한 실습이었다.

![practice_1.png](/assets/images/posts/2021-08-01/practice_1.png){: .align-center}{: width="100%", height="100%"}

{:.image-caption}
*그림2(practice 1 해답)*


마찬가지로 git log로 결과를 확인해보자. 확인해본결과 Add knapsack... commit이후 3개의 commit이 잘 들어간것을 확인할 수 있다.

![rebase_i.png](/assets/images/posts/2021-08-01/practice_1_result.png){: .align-center}{: width="100%", height="100%"}

{:.image-caption}
*그림2(practice 1 git log)*

**Primary Notice:** 해당 포스트는 리얼리눅스 송태웅님의 강의를 보고 정리한 내용임을 알립니다.
{: .notice--primary}
