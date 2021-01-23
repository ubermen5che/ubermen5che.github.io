---

title: "Logic and Computer Science - 논리학 기초 - Statements"
categories:
  - Logic
  - Computer Science
tags:
  - Logic
  - Computer Science
  - 논리
  - 컴퓨터과학
---
### What is an statements?
statements는 참 또는 참일 가능성이 있는 문장이다. statements를 만들때 어떤 참또는 거짓을 보일 수 있는 문장을 만들어야한다.

>
* 모든 한국시민들은 한국에 살고 있다.
* 서울에 있는 모든 기업들은 사무실에서 레드 카펫을 사용하고있다.
* 애플은 마이크로소프트보다 더 많은 자본을 가지고 있다.

위의 예를 잘 살펴보면, 각 statements는 참을 나타내려고 시도하고 있는 것처럼 보인다. 물론 첫 번쨰, 두 번째 문장들이 거짓이고 세 번째 문장은 참이지만 논리학의 관점에서 보았을 때, 각 문장들은 statement로서 자격을 갖추었다. 왜냐하면 참일 가능성이 있는 문장을 나타내고 있기 때문이다.

영어에서는 이러한 문장들을 "declarative"하다고불린다. 논리학에서는 오직 declaratives만이 statements의 자격을 가진다. 아래의 다양한 종류의 문장들이 있는데, 이 문장들을 비교하고 대조해보자.

>
1. Question or "interrogative" - What time is it?
2. Commands or "imperative" - Close the door!
3. Exclamations or "exclamatory" - Ouch!
4. Performatives - "I promise," "I thee wed"
5. Declarative - "Carbon is an element"


1,2,3,4와 같은 종류의 문장의 경우 참 또는 거짓을 논할 수 없는 문장이다. 따라서 오직 5번과 같은 종류의 문장만 statements라고 말할 수 있다.


### Simple and Compound statements

오직 하나의 진실만을 공표하는 statement를 **simple statement**라고 한다.
>
Ann is home
>
Bob is home

이러한 문장들은 각각 어떤 것이 참인지 그리고 어떤 사건에 대한 오직 하나의 상황 또는 상태가 참인지 볼 수 있다. 논리적인 arguments를 작성할 때, 나의 전제들을 내가 분해할 수 있을 만큼 simple statements로 분해하는 것은 좋은 연습이 될 수 있다. 이 연습이 나의 argument에 statements에 대한 진리를 결정하는데 도움을 줄 수 있다.

만약 내 전제에서 하나 이상의 진리를 선언하고 싶다면 어떻게 할 것인가? 이 경우 **statment operator**를 사용하여 두개이상의 simple statments를
합칠 수 있다. statement operator는 simple statements를 연결하여 **compound statment**를 만들어준다. 예를 들어,

>
Ann is hime AND Bob is home
>
The screen door has a hole in it OR I'm seeing things

위의 각 문장에서, AND, OR에 의해 합쳐진 두개의 simple statments 볼 수 있다. 이러한 연산자들은 simple statments를 사용하여 하나 이상의 진리를 선언하도록 허용해준다. 추후에 이루어질 포스팅에서는 다양한 연산자들을 알아볼 것이다.




**Primary Notice:** Logic and Computer Science 시리즈는 마이크로소프트에서 제공하는 강좌를 토대로 내용을 정리하여 작성되었음을 알려드립니다.
{: .notice--primary}
