---
created: 2025-11-12 13:22
updated: 2025-11-12 13:22
status:
  - draft
tags:
  - Array
source: Programmers
---
# 📝 TIL 2025-11-12 - JadenCase 문자열 만들기

> **관련 주제:** [[MOC_TBD]] 

## 🚀 개요 및 핵심 (Summary)

## **문제 설명**

JadenCase란 모든 단어의 첫 문자가 대문자이고, 그 외의 알파벳은 소문자인 문자열입니다. 단, 첫 문자가 알파벳이 아닐 때에는 이어지는 알파벳은 소문자로 쓰면 됩니다. (첫 번째 입출력 예 참고)  
문자열 s가 주어졌을 때, s를 JadenCase로 바꾼 문자열을 리턴하는 함수, solution을 완성해주세요.

## **제한 조건**

- s는 길이 1 이상 200 이하인 문자열입니다.
- s는 알파벳과 숫자, 공백문자(" ")로 이루어져 있습니다.
    - 숫자는 단어의 첫 문자로만 나옵니다.
    - 숫자로만 이루어진 단어는 없습니다.
    - 공백문자가 연속해서 나올 수 있습니다.

## **입출력 예**

| s                       | return                  |
| ----------------------- | ----------------------- |
| "3people unFollowed me" | "3people Unfollowed Me" |
| "for the last week"     | "For The Last Week"     |

---

## 📚 상세 내용 (Deep Dive)
> **코딩 오류 및 피드백**
> Cast 상황과 변환 함수의 사용을 적절히 판단할 것
```java
// 제출 답안
public class Main {
    public String result(String str) {
        str = str.toLowerCase();
        String output = " ";
        for (char c : str.toCharArray()) {
            if (output.charAt(output.length() - 1) == ' ') {
                if (97 <= c && c <= 122) {
                    output += Character.toUpperCase(c);
                } else {
                    output += c;
                }
            } else {
                output += c;
            }
        }

        return output.substring(1);
    }

    public static void main(String[] args) {
        Main T = new Main();
        String str = "3people unFollowed Me";
        System.out.println(T.result(str));
    }
}
```
> **AI 개선안**
> `StringBuilder`는 문자열 덧붙임 연산에 효율적
> `isStart` 플래그로 "단어의 시작"을 관리 → 직관적이고 명확
```java
// AI 피드백 1
public String result(String str) {
    str = str.toLowerCase();
    StringBuilder sb = new StringBuilder(" ");

    for (char c : str.toCharArray()) {
        if (sb.charAt(sb.length() - 1) == ' ') {
            if ('a' <= c && c <= 'z') {
                sb.append(Character.toUpperCase(c));
            } else {
                sb.append(c);
            }
        } else {
            sb.append(c);
        }
    }

    return sb.substring(1);
}

// AI 피드백 2
public String result(String str) {
    StringBuilder sb = new StringBuilder();
    boolean isStart = true;

    for (char c : str.toLowerCase().toCharArray()) {
        if (c == ' ') {
            isStart = true;
            sb.append(c);
        } else if (isStart) {
            sb.append(Character.toUpperCase(c));
            isStart = false;
        } else {
            sb.append(c);
        }
    }

    return sb.toString();
}
```
```java
// 다른 사람 풀이
class Solution {
  public String solution(String s) {
        String answer = "";
        String[] sp = s.toLowerCase().split("");
        boolean flag = true;

        for(String ss : sp) {
            answer += flag ? ss.toUpperCase() : ss;
            flag = ss.equals(" ") ? true : false;
        }

        return answer;
  }
}
```