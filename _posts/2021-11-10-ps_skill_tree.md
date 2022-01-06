---
layout : posts
comments : true
title: "[PS-programmers] 스킬트리"

categories:
  - PS
tags:
  - programmers
  - level2
  - 구현
---


### **문제 설명**

선행 스킬이란 어떤 스킬을 배우기 전에 먼저 배워야 하는 스킬을 뜻합니다.

예를 들어 선행 스킬 순서가 `스파크 → 라이트닝 볼트 → 썬더`일때, 썬더를 배우려면 먼저 라이트닝 볼트를 배워야 하고, 라이트닝 볼트를 배우려면 먼저 스파크를 배워야 합니다.

위 순서에 없는 다른 스킬(힐링 등)은 순서에 상관없이 배울 수 있습니다. 따라서 `스파크 → 힐링 → 라이트닝 볼트 → 썬더`와 같은 스킬트리는 가능하지만, `썬더 → 스파크`나 `라이트닝 볼트 → 스파크 → 힐링 → 썬더`와 같은 스킬트리는 불가능합니다.

선행 스킬 순서 skill과 유저들이 만든 스킬트리를 담은 배열 skill_trees가 매개변수로 주어질 때, 가능한 스킬트리 개수를 return 하는 solution 함수를 작성해주세요.

### 입출력 예

![Untitled](/assets/images/posts/2021-11-10/Untitled.png)

### 해결 방법

우선 입력으로 들어온 skill에 대해서 순서를 저장할 수 있는 `Map<String, Integer>` 변수와 어떤 스킬들이 있는지에 대한 정보를 저장할 수 있는 `List<Character>` 변수 하나씩 만들어준다. 그 후 skill_trees에 주어진 `String[]` 을 순회하면서 element 각각에 대해 기존에 저장해놓은 스킬들의 순서정보 및 스킬 존재 정보를 이용해서 해를 구하는 방식이다.

예를들어 "BACDE"일때 우선 input으로 들어온 skill에 존재하는 스킬인지 검사한다(존재하지 않는 스킬이면 고려하지 않아도 된다). 검사이후에는 A와 E가 skill에 존재하지 않는 스킬이기 때문에 "BACDE"에서 제외시킬 수 있다. 그러면 순서를 유지한채로 제외시키면 "BCD"가 남는다. 이 때 각각의 스킬들에 대해서 스킬 대한 순서를 저장한 map 변수에서 순서정보를 가져오면 "BCD" → 2,1,3임을 확인할 수 있는데 순서가 뒤바뀐것을 알 수 있다.

위와같은 방식으로 알고리즘을 적용해보면 "CBADF"의 경우는 "CBD"가 남게되고 "CBD" → 1,2,3임으로 올바른 스킬트리임을 알 수 있다. 그 다음 element로는 "AECD"인데 유효한 스킬검사 이후에는 "CD"만 남게되고 마찬가지로 "CD" → 1,3임을 알 수 있다. 스킬D는 B를 찍지 않고서는 찍을 수 없기때문에 유효하지 않은 스킬트리이다. 마지막으로 "BDA"의 경우 유효한 스킬들만 남기면 "BD". 순서를 나타내면 "BD" -> 1,2이므로 유효한 스킬트리이다. 따라서 유효한 스킬트리들만 카운터 할 경우 결과로 2가 나오게되어 테스트케이스를 통과 할 수 있게된다.

### 코드

```java
import java.util.*;

class Solution {
    public int solution(String skill, String[] skill_trees) {
        int answer = 0;

        HashMap<Character, Integer> map = new HashMap<>();
        ArrayList<Character> skills = new ArrayList<>();
        ArrayList<Integer> skillOrder = new ArrayList<>();

        for (int i = 0; i < skill.length(); i++){
            skills.add(skill.charAt(i));
            map.put(skill.charAt(i), i);
        }

        System.out.println(skills);
        for (int i = 0; i < skill.length(); i++){
            System.out.println(map.get(skill.charAt(i)));
        }

        for (int i = 0; i < skill_trees.length; i++){
            String tree = skill_trees[i];
            for (int j = 0; j < tree.length(); j++){
                if (!skills.contains(tree.charAt(j)))
                    continue;
                skillOrder.add(map.get(tree.charAt(j)));
            }

            if (skillOrder.size() > skill.length())
                continue;

            boolean flag = true;
            int ord;

            for (int k = 0; k < skillOrder.size(); k++){
                ord = skillOrder.get(k);
                if (ord != k){
                    flag = false;
                    break;
                }
            }

            if (flag == true)
                answer++;

            skillOrder.clear();
        }

        return answer;
    }
}
```
