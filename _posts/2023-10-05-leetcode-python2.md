---
layout: post
title:  "leet code python 2"
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

2. Add Two Numbers

##### 문제
---
- 2개의 Linked-List가 주어진다.
- 각 Linked-List의 숫자값은 역으로 넣어준다.
- 2개의 합을 구하여 다시 Linked-List로 나타내라

##### 알고리즘
---
- Class는 단순하게 단방향 Linked List
- 값을 추출 하는 경우에는 문자열로 추출(역순으로 하기 위함)
- 결과 값을 앞에서부터 pop하여 Linked-List로 구현
- Linked-List 처음 값은 의미없는 값임으로 생략

##### 구현
---
- ListNode는 값과 다음 Node를 가르킨다.

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next

class Solution:
    def extract_val(self, lst: Optional[ListNode]) -> str:
        answer = ''
        while lst:
            answer += str(lst.val)
            lst = lst.next
        return answer

    def addTwoNumbers(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        # 값 추출
        l1_result, l2_result = self.extract_val(l1), self.extract_val(l2)
        print(l1_result, l2_result)
        # 역순으로 설정
        l1_result, l2_result = l1_result[::-1], l2_result[::-1]
        
        result = int(l1_result) + int(l2_result)
        result = list(str(result))
   
        answer = ListNode()

        while result:
            temp = ListNode()
            temp.val = int(result.pop(0))
            answer.next, temp.next = temp, answer.next
        answer = answer.next
        return answer
```