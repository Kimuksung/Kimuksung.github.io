---
layout: post
title:  "leet code python 3"
author: Kimuksung
categories: [ python_leetcode ]
tags: [ leetcode ]
# 댓글 기능
comments: False
# feature tab 여부
featured: False
# story에서 보이게 할지 여부
hidden: True
image: assets/images/leetcode.png
---

3. Longest Substring Without Repeating Characters

##### 문제
---
- 문자열 s가 주어진다.
- 문자가 중복되지 않으면서 가장 긴 길이를 출력하라

##### 알고리즘
---
- 처음 드는 생각에는 이중포문을 생각하였으나 리밋 값 이상
- slide window를 사용하여 구현

##### 구현
---
- start, end를 두고 하나씩 이동해가면서 구현
- end는 각 문자열을 한번씩만 순회
- 중복이 발생하면 start를 이동하는 방식으로 구성
- `a in list`와 `a in set` 사용 시 set이 속도가 더 빠르다.

<a href="https://ibb.co/Y0tcjGz"><img src="https://i.ibb.co/vXJ3z87/2.jpg" alt="2" border="0"></a><br /><a target='_blank' href='https://imgbb.com/'></a><br />

```python
# with list
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        answer = 0
        start = 0
        
        for end in range(len(s)):            
            while s[end] in s[start:end]:
                start += 1

            answer = max(answer, end-start+1)
            
        return answer

# with set
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        charSet = set()
        answer = 0
        start = 0
        
        for end in range(len(s)):            
            while s[end] in charSet:
                charSet.remove(s[start])
                start += 1
            charSet.add(s[end])

            answer = max(answer, end-start+1)
            
        return answer
```