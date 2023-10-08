---
layout: post
title:  "leet code python 5"
author: Kimuksung
categories: [ python_leetcode ]
tags: [ leetcode ]
# 댓글 기능
comments: False
# feature tab 여부
featured: False
# story에서 보이게 할지 여부
image: assets/images/leetcode.png
hidden: True
---

5. Longest Palindromic Substring

##### 문제 
---
- 가장 긴 대칭 문자열을 반환해라
- 여러 개가 나온다면 어떤것을 반환해도 상관 없다.

알고리즘
---
- 바로 생각 할 수 있는 것은 각 문자열 시작부터 끝까지 비교 → O(N^3)
- 문자열 시작 지점 부터 좌우로 퍼져나가면서 비교 → O(N^2)

```python
# O(N^3)
class Solution:
    def longestPalindrome(self, s: str) -> str:
        def checkpalindrome(s: str) -> bool:
            return s == s[::-1]
        
        answer = 0
        result = ''
        n = len(s)
        for i in range(n):
            for j in range(i,n):
                if checkpalindrome(s[i:j+1]):
                    if j-i >= answer:
                        answer = max(answer, j-i)
                        result = s[i:j+1]

        return result

# O(N^2)
class Solution:
    def longestPalindrome(self, s: str) -> str:
        result = ''
        result_len = 0

        for i in range(len(s)):
            # 홀수인 경우
            left, right = i, i
            while left >= 0 and right < len(s) and s[left] == s[right]:
                if (right-left+1) > result_len:
                    result = s[left:right+1]
                    result_len = right-left+1
                left -= 1
                right += 1

            # 짝수인 경우
            left, right = i, i+1
            while left >= 0 and right < len(s) and s[left] == s[right]:
                if (right-left+1) > result_len:
                    result = s[left:right+1]
                    result_len = right-left+1
                left -= 1
                right += 1

        return result
```