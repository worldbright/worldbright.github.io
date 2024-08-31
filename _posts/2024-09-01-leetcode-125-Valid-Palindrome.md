---
title: "[Easy] LeetCode - 125. Valid Palindrome"
categories:
  - Leet Code
  - Top Interview 150
tags:
  - LeetCode
  - TopInterview150
---
## 문제

A phrase is a **palindrome** if, after converting all uppercase letters into lowercase letters and removing all non-alphanumeric characters, it reads the same forward and backward. Alphanumeric characters include letters and numbers.

Given a string `s`, return `true` _if it is a **palindrome**, or_ `false` _otherwise_.

**Example 1:**

> **Input:** s = "A man, a plan, a canal: Panama"  
> **Output:** true  
> **Explanation:** "amanaplanacanalpanama" is a palindrome.

**Example 2:**

> **Input:** s = "race a car"  
> **Output:** false  
> **Explanation:** "raceacar" is not a palindrome.

**Example 3:**

> **Input:** s = " "  
> **Output:** true  
> **Explanation:** s is an empty string "" after removing non-alphanumeric characters.  
> Since an empty string reads the same forward and backward, it is a palindrome.

**Constraints:**

- `1 <= s.length <= 2 * 105`
- `s` consists only of printable ASCII characters.
## 풀이

```cpp
using namespace std;
class Solution {
public:
    char getLower(char c) {
        if (('a' <= tolower(c) && tolower(c) <= 'z') ||
            ('0' <= c          && c <= '9')) {
            return tolower(c);
        }
        return -1;
    }
    bool isPalindrome(string s) {
        int cur1 = 0;
        int cur2 = s.size() - 1;
        do {
            while (cur1 < s.size() && getLower(s[cur1]) == -1) cur1++;
            while (0 <= cur2       && getLower(s[cur2]) == -1) cur2--;
            if (cur1 < s.size() && 0 <= cur2 && tolower(s[cur1]) != tolower(s[cur2])) {
                return false;
            }
            ++cur1;
            --cur2;
        } while (cur2 - cur1 %3E= 1);
        return true;
    }
}
```

오랜만에 구현이 즐거운, 재미있는 문제였습니다.

**문제 한 줄 요약**

> 주어진 문자열 배열에서 [0-9a-zA-Z]의 문자를 제외한 문자는 제거하고 대문자는 소문자로 변경한 후 순방향, 역방향으로 읽어도 동일한 배열일 경우 true, 아니면 false를 반환하시오.

간단하게 두 포인터로 순방향 역방향에서 [0-9a-zA-Z] 문자를 찾아서 다른 포인트에서 false를 바로 return해버리고, 끝까지 false return이 안되었다면 true를 return하도록 구현했습니다.

코드가 조금 지저분한 것 같아, 이 코드보다 깔끔한 구현이 존재할지 Solutions를 뒤적여 봤는데.. 그렇다 할 멋진 코드는 발견하지 못했습니다.

끝.