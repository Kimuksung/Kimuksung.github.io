---
layout: post
title:  "Python list vs array?"
author: Kimuksung
categories: [ Python ]
tags: [ Python ]
image: assets/images/python.png
comments: False
---

Python에서 List와 Array를 어떻게 구별하여 쓰는지 알아보겠습니다.

Python에서는 일반적으로 Array를 지원하지 않는다.

Python은 `동적 할당`인 `List`를 사용하기 때문이다.

Numpy를 통하여 `정적 할당`인 `Array`를 표현 가능하다.

동일한 고정된 크기를 가지고 있으며, 크기 변경 시 기존의 Array는 삭제하고 새로운 Array를 생성해야 합니다.

##### List
---
- index를 가지고 있다.
- 메모리 주소가 연속적이지 않을 수 있다.
- 포인터를 통해 다음 데이터의 위치를 가리키고 있다.
- 동적 크기 할당 가능
- 검색 성능이 좋지 않다.
- Python 에서는 메모리 상의 연속적인 위치에 배치 → index 를 사용하여 접근이 가능하다.

##### Array
---
- C코드로 작성된 코드에 의해 동작하여 더 빠르다.
- 크기를 지정하기 때문에 직접적인 접근이 가능하다.