---
title: "[PS-programmers] 괄호 변환"

categories:
  - PS
tags:
  - programmers
  - level2
  - 재귀
  - 구현
---

### **문제 설명**

카카오에 신입 개발자로 입사한 **"콘"**은 선배 개발자로부터 개발역량 강화를 위해 다른 개발자가 작성한 소스 코드를 분석하여 문제점을 발견하고 수정하라는 업무 과제를 받았습니다. 소스를 컴파일하여 로그를 보니 대부분 소스 코드 내 작성된 괄호가 개수는 맞지만 짝이 맞지 않은 형태로 작성되어 오류가 나는 것을 알게 되었습니다.수정해야 할 소스 파일이 너무 많아서 고민하던 "콘"은 소스 코드에 작성된 모든 괄호를 뽑아서 올바른 순서대로 배치된 괄호 문자열을 알려주는 프로그램을 다음과 같이 개발하려고 합니다.

### **용어의 정의**

**'('** 와 **')'** 로만 이루어진 문자열이 있을 경우, '(' 의 개수와 ')' 의 개수가 같다면 이를 **`균형잡힌 괄호 문자열`**이라고 부릅니다.그리고 여기에 '('와 ')'의 괄호의 짝도 모두 맞을 경우에는 이를 **`올바른 괄호 문자열`**이라고 부릅니다.예를 들어, `"(()))("`와 같은 문자열은 "균형잡힌 괄호 문자열" 이지만 "올바른 괄호 문자열"은 아닙니다.반면에 `"(())()"`와 같은 문자열은 "균형잡힌 괄호 문자열" 이면서 동시에 "올바른 괄호 문자열" 입니다.

'(' 와 ')' 로만 이루어진 문자열 w가 "균형잡힌 괄호 문자열" 이라면 다음과 같은 과정을 통해 "올바른 괄호 문자열"로 변환할 수 있습니다.

`1. 입력이 빈 문자열인 경우, 빈 문자열을 반환합니다.
2. 문자열 w를 두 "균형잡힌 괄호 문자열" u, v로 분리합니다. 단, u는 "균형잡힌 괄호 문자열"로 더 이상 분리할 수 없어야 하며, v는 빈 문자열이 될 수 있습니다.
3. 문자열 u가 "올바른 괄호 문자열" 이라면 문자열 v에 대해 1단계부터 다시 수행합니다.
  3-1. 수행한 결과 문자열을 u에 이어 붙인 후 반환합니다.
4. 문자열 u가 "올바른 괄호 문자열"이 아니라면 아래 과정을 수행합니다.
  4-1. 빈 문자열에 첫 번째 문자로 '('를 붙입니다.
  4-2. 문자열 v에 대해 1단계부터 재귀적으로 수행한 결과 문자열을 이어 붙입니다.
  4-3. ')'를 다시 붙입니다.
  4-4. u의 첫 번째와 마지막 문자를 제거하고, 나머지 문자열의 괄호 방향을 뒤집어서 뒤에 붙입니다.
  4-5. 생성된 문자열을 반환합니다.`

**"균형잡힌 괄호 문자열"** p가 매개변수로 주어질 때, 주어진 알고리즘을 수행해 **"올바른 괄호 문자열"**로 변환한 결과를 return 하도록 solution 함수를 완성해 주세요.

### 해결 방법 및 회고

문제 설명을 자세히 읽어보면 "1단계 부터 다시 수행합니다", "재귀적으로 수행한 결과를 이어 붙입니다" 등의 문장을 보고 재귀를 사용해서 접근해야하는것을 알 수 있었다. 이러한 재귀 관련 문제들을 몇문제 풀어보면서 느낀것이지만 문제를 풀때 첫번째 난관이 재귀적인 문제인지 파악하는것이고 재귀문제라면 반복되는 구조를 발견하는것이 어려운 것 같다. 그러나 다행히도? 위의 문제에서는 문제 해결을 위한 조건들을 명시해주어서 접근하기가 수월했던 것 같다.

문제에서 요구하는 조건들을 잘 파악해서 재귀함수를 잘 정의하면 쉽게 풀리는 문제이다. 그 외에 여러 문자열 처리나 올바른 괄호 문자열 확인을 위한 스택 구현등이 필요한데 평소 자료구조에 익숙하고 문자열 처리 관련 문제를 몇번 풀어보았다면 수월하게 해결할 수 있었다.

### 코드

```java
import java.util.*;

class Solution {
    int[] prenCnt = new int[2];

    public String solution(String p) {
        String answer = "";

        if (isRight(p))
            return p;
        else {
            //균형잡힌 문자열에 대한 처리 로직
            return findAnswer(p);
        }

    }

    private String findAnswer(String w) {
        System.out.println("w : " + w);
        if (w.isEmpty())
            return "";
        else {
            //2단계 균형잡힌 괄호 문자열 u,v분리
            prenCnt[0] = 0;
            prenCnt[1] = 0;

            for (int i = 0; i < w.length(); i++){
                if ((prenCnt[0] > 0 && prenCnt[1] > 0) && prenCnt[0] == prenCnt[1])
                    break;

                if (w.charAt(i) == '(')
                    prenCnt[0]++;
                else if (w.charAt(i) == ')')
                    prenCnt[1]++;
            }

            String u;
            String v;

            int sub = prenCnt[0] + prenCnt[1];

            if (sub == 0)
                u = "";
            else {
                u = w.substring(0, sub);
            }


            if (w.length() == u.length())
                v = "";
            else
                v = w.substring(prenCnt[0] + prenCnt[1]);

            if (isRight(u)) {
                return u + findAnswer(v);
            } else {
                String emptyStr = "";

                emptyStr += "(";
                emptyStr += findAnswer(v);
                emptyStr += ")";

                //4-4 수행

                String uSubStr;
                String reverseStr = "";

                uSubStr = u.substring(1, u.length()-1);

                //뒤집기 수행
                for (int j = 0; j < uSubStr.length(); j++){
                    if (uSubStr.charAt(j) == '(')
                        reverseStr += ")";
                    else if (uSubStr.charAt(j) == ')')
                        reverseStr += "(";
                }

                emptyStr += reverseStr;

                return emptyStr;
            }
        }
    }

    //올바른 괄호 문자열인지 판단 함수 (스택이용)
    private boolean isRight(String p){
        boolean flag = true;
        ArrayList<Character> stack = new ArrayList<>();
        Character ch;

        for (int i = 0; i < p.length(); i++){

            ch = p.charAt(i);

            // 왼쪽괄호의 경우 무조건 삽입.
            if (ch == '(') {
                stack.add(ch);
            //오른쪼 괄호 입력시 stack의 맨위 확인 후 ( 존재하면 최상위 element pop
            //같은 ) 라면 여전히 삽입.
            } else if (ch == ')') {
                if (!stack.isEmpty()){
                    if (stack.get(stack.size()-1) == '('){
                        stack.remove(stack.size()-1);
                        continue;
                    }
                }
                stack.add(ch);
            }
        }

        if (!stack.isEmpty())
            flag = false;

        return flag;
    }
}
```
