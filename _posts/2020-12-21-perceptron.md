---
title: "딥러닝 - 퍼셉트론(perceptron)"
categories:
  - AI
  - Deep-Learning
tags:
  - perceptron
  - AI
  - Deep-Learning
---
<script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML"></script>

이번 글에서는 **퍼셉트론** 알고리즘에 대해 알아보겠습니다. 퍼셉트론은 신경망(딥러닝)의 기원이 되는 알고리즘이기 때문에 퍼셉트론의 구조를 이해하는 것은 딥러닝을 배우는데에 있어서 중요한 아이디어를 배우는 일입니다.

### 퍼셉트론이란?

퍼셉트론은 다수의 신호를 입력으로 받아 하나의 신호를 출력합니다. 아래 그림[1-1]은 2개의 신호를 받은 퍼셉트론의 예를 보여줍니다. $$x_1$$과 $$x_2$$는 입력 신호, $$y$$는 출력 신호, $$w_1$$과 $$w_2$$는 가중치를 뜻합니다. 그림의 원을 **뉴런** 혹은 **노드**라고 부릅니다. 입력신호가 뉴런에 보내질 때는 각각의 고유한 **가중치(weight)**가 곱해집니다. 뉴런에서 보내온 신호의 총합이 정해진 한계를 넘어설 때만 1을 출력합니다(이를 뉴런을 활성화한다라고 표현하기도 합니다). 일반적으로 '한계'를 **임계값**이라 하며, 기호$$θ$$로 나타냅니다.

위의 내용을 수식으로 표현하면, 이래와 같습니다.

$$
y =
\begin{cases}
0(w_1x_1 + w_2x_2)\leq θ &  \\
1(w_1x_1 + w_2x_2)> θ
\end{cases}
$$

퍼셉트론은 복수의 입력 신호 각각에 고유한 가중치를 부여합니다. 가중치가 크면 해당 신호가 그 만큼 더 중요함을 나타냅니다.

<p align="center">
    <img src="https://user-images.githubusercontent.com/76172759/102718800-7de9dd80-432d-11eb-9cf9-e55d594e5787.png" alt="perceptron thumbnail" width=300 />
</p>
<p align="center" style="color:gray">
    <em>[그림 1-1] 입력이 2개인 퍼셉트론</em>
</p>
