---
layout: post
title:  "leet code python 229"
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

229. Majority Element II

##### 문제
---
- n개의 list에서 특정 숫자의 수가 n/3 보다 많이 나온 횟수

##### 알고리즘
---
- set과 list.count를 활용
- list 내에서 갯수 카운트 = Counter
- list.count는 속도가 느리다고 알고 있기에 Counter로 구성

##### 구현
---
- Counter 라이브러리로 갯수 추출
- filter 함수로 특정 값 이상인 key만 출력

```python
from collections import Counter

class Solution:
    def majorityElement(self, nums: List[int]) -> List[int]:
        n = len(nums)
				n = n/3
        target = Counter(nums)
        result = dict(filter(lambda x: x[1]>n, target.items())).keys()
        return result
```