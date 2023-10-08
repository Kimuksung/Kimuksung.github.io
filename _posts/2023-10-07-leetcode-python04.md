---
layout: post
title:  "leet code python 4"
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

4. Median of Two Sorted Arrays

문제 풀이 과정을 아래 이미지로 대채합니다.
- 어려운 문제만 표출되도록 구성하였습니다. (일반적인 코딩테스트 풀이 문제는 아래 태그에서 확인 하면 됩니다.)
```python
class Solution:
    def findMedianSortedArrays(self, nums1: List[int], nums2: List[int]) -> float:
        A, B = nums1, nums2
        total = len(nums1) + len(nums2)
        half = total//2

        if len(A) > len(B):
            A, B = B, A
        
        left, right = 0, len(A)-1

        while True:
            mid = (left+right)//2 #A
            j = half-mid-2 # B
            
            Aleft= A[mid] if mid >= 0 else float("-infinity")
            Aright = A[mid+1] if (mid+1) < len(A) else float("infinity")
            Bleft = B[j] if j >= 0 else float("-infinity")
            Bright = B[j+1] if (j+1) < len(B) else float("infinity")
            print(total,half,mid,j, Aleft, Aright, Bleft,Bright)
            if Aleft <= Bright and Bleft <= Aright:
                print(total,half,mid,j, A, B, max(Aleft, Bleft), min(Aright, Bright))
                if total % 2:
                    return min(Aright, Bright)
                return (max(Aleft, Bleft)+min(Aright, Bright))/2
            elif Aleft > Bright:
                right = mid - 1
            else:
                left = mid + 1
```

<a href="https://ibb.co/Gdhz9FP"><img src="https://i.ibb.co/YNrHXbj/Leetcode4-1.jpg" alt="Leetcode4-1" border="0"></a>
<a href="https://ibb.co/pJRnPPy"><img src="https://i.ibb.co/sy6tPPH/Leetcode4-2.jpg" alt="Leetcode4-2" border="0"></a>
<a href="https://ibb.co/ThVtCN9"><img src="https://i.ibb.co/gg5Jnck/Leetcode4-3.jpg" alt="Leetcode4-3" border="0"></a>
<a href="https://ibb.co/1ddJ19k"><img src="https://i.ibb.co/ZYY15h0/Leetcode4-4.jpg" alt="Leetcode4-4" border="0"></a>
<a href="https://ibb.co/xqK9rSj"><img src="https://i.ibb.co/fdV6TYp/Leetcode4-5.jpg" alt="Leetcode4-5" border="0"></a>
<a href="https://ibb.co/23pSdH1"><img src="https://i.ibb.co/60q8FL7/Leetcode4-6.jpg" alt="Leetcode4-6" border="0"></a>
<a href="https://ibb.co/r0PGwMZ"><img src="https://i.ibb.co/mTsCctH/Leetcode4-7.jpg" alt="Leetcode4-7" border="0"></a>