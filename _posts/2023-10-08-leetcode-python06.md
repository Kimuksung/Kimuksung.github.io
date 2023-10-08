---
layout: post
title:  "leet code python 6"
author: Kimuksung
categories: [ python_leetcode ]
tags: [ leetcode ]
# 댓글 기능
comments: False
# feature tab 여부
featured: False
# story에서 보이게 할지 여부
image: assets/images/leetcode.png
---

6. Zigzag Conversion

##### 문제
---
- 문자열을 지그재그 형태로 한글자씩 만든다.
- 이 때 주어진 값만큼 내려갔다가 지그재그로 구성

##### 알고리즘
---
- 지그재그의 특성을 잘 파악해야 한다. ( 문제에서도 볼 수 있듯이 매우 싫어요가 많다. )
- 처음과 끝 줄은 Numrows의 특성으로 구성이 가능하다.
- 하지만, 그 외 나머지 중간에 있는 값들은 지그재그의 특성을 추가로 반영해야한다.
- 시작 지점위치에서 Numrows-1만큼 내려가고 대각선으로 Nowrows-1만큼 올라간다.
- Left = 내려가는 부분
- Right = 대각선으로 올라가는 부분을 나누어서 구현하였다.
- 또한, numRows가 1인 경우에는 대각선, 시작점이 발생하지 않으니 주의가 필요

```python
class Solution:
    def convert(self, s: str, numRows: int) -> str:
        move_n = numRows*2-2
        total = len(s)
        answer = ''
        
        if numRows == 1:
            return s

        if total%move_n:
            s += '0'*(move_n-total%move_n)

        split_lst = [s[i:i+move_n] for i in range(0, total, move_n)]
        split_lst = [(s[:numRows], s[numRows:]) for s in split_lst ]

        for i in range(numRows):
            for left, right in split_lst:      
                if left[i] != '0':
                    answer += left[i]
                if 0 < i < numRows-1 and right[-i] != '0':
                    answer += right[-i]

        return answer
```

<a href="https://ibb.co/xKZ25hk"><img src="https://i.ibb.co/twRHCBG/Leetcode-6-Zigzag-2.jpg" alt="Leetcode-6-Zigzag-2" border="0"></a>

<a href="https://ibb.co/Zmb9cTq"><img src="https://i.ibb.co/2NpJcKG/Leetcode-6-Zigzag-3.jpg" alt="Leetcode-6-Zigzag-3" border="0"></a>