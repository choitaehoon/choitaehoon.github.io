---
title: "카카오 괄호 변환"
date: 2020-03-14 16:23:00 
---

#### 카카오 괄호 변환 문제

이 문제를 전체적으로 읽어 봤는데 구현 능력을 보는 것 같다. 카카오 해설에서 어떻게 풀었는지 궁금해서 찾아 봤다.

![images](https://user-images.githubusercontent.com/33123391/76677016-84179480-660c-11ea-808e-a55ba1baab83.png)

카카오 해설지에서도 괄호쌍을 확인 하는 방법, 재귀 함수 이해 , 문제에 주어진대로 해결할 수 있다는 문구가 적혀 있다. 이 문제에 대한 정답률은 23.1% 이다. 한번 같이 문제를 보고 풀어 보자.

![images](https://user-images.githubusercontent.com/33123391/76677082-6e569f00-660d-11ea-9334-c392f2c0d9e8.png)

문제에 대한 조건들이다. 이 순서 대로 문제들을 풀도록 해보자

* 문제를 풀기 전에 이 메소드가 제공 되는데 solution 함수 안에 풀어야 한다.

  ```java
  public String solution(String p) {
      String answer = "";
      return answer;
  }
  ```

* 1번 조건

  * 입력이 빈 문자열인 경우에 빈 문자열을 반환 하라고 했으니 빈 문자열을 반환 하게 구현 했다.

    ```java
    if (p.isEmpty())
        return "";
    ```

* 2번 조건

  * u

    * 균형 잡힌 괄호 문자열로 더 이상 분리 할 수 없어야 함

  * v

    * 빈 문자열이 될 수 있음

    ```java
    private static boolean checkP(String p) {
    
        boolean isCheck = true;
        Stack<Character> stack = new Stack<>();
        int left = 0, right = 0; // 개수를 세서 균형 잡힌 괄호 문자열인지 확인 한다.
    		
        for(int i=0; i<p.length(); ++i) {
            if (p.charAt(i) == '(') {
                ++left;
                stack.push(p.charAt(i));
            } else {
                ++right;
    
                if (stack.isEmpty()) 
                    isCheck = false;
                else
                    stack.pop();
            }
    
          /*
          	index는 static 변수
          		index가 어디인지 알 수 있다면 그 전 까지는 균형 잡힌 괄호 문자열을 알 수 있기 때문에 설정
            left와 right가 같으면 균형 잡힌 괄호 문자열이다.
          */
            if (left == right) {
                index = i + 1; 
                return isCheck;
            }
        }
    
        return true;
    }
    ```

* 3번, 3-1 조건

  ```java
  boolean isCorrectP = checkP(p);
  String u = p.substring(0, index); // 균형 잡힌 괄호 문자열
  String v = p.substring(index);
  
  if (isCorrectP) {
      return u + solution(v);
  }
  ```

* 4번 조건

  ```java
  String temp = "(" + solution(v) + ")";
  StringBuilder store = new StringBuilder();
  for (int i=1; i<u.length() - 1; ++i) { // 4-4 조건 
      if (u.charAt(i) == '(')
          store.append(")");
      else
          store.append("(");
  }
  
  index = 0;
  return temp + store.toString();
  ```

* 전체 소스

  ```java
  public static String solution(String p) {
      if (p.isEmpty())
          return "";
  
      boolean isCorrectP = checkP(p);
      String u = p.substring(0, index);
      String v = p.substring(index);
  
      if (isCorrectP) {
          return u + solution(v);
      }
  
      String temp = "(" + solution(v) + ")";
      StringBuilder store = new StringBuilder();
      for (int i=1; i<u.length() - 1; ++i) {
          if (u.charAt(i) == '(')
              store.append(")");
          else
              store.append("(");
      }
  
      index = 0;
      return temp + store.toString();
  }
  
  private static boolean checkP(String p) {
  
      boolean isCheck = true;
      Stack<Character> stack = new Stack<>();
      int left = 0, right = 0;
  
      for(int i=0; i<p.length(); ++i) {
          if (p.charAt(i) == '(') {
              ++left;
              stack.push(p.charAt(i));
          } else {
              ++right;
  
              if (stack.isEmpty())
                  isCheck = false;
              else
                  stack.pop();
          }
  
          if (left == right) {
              index = i + 1;
              return isCheck;
          }
      }
  
      return true;
  }
  ```

  * 이 문제의 핵심은 주어진 상황을 주고 그걸 구현 할 수 있는 능력을 보는 것이다.