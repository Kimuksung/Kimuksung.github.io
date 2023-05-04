---
layout: post
title:  "Linux Command"
author: Kimuksung
categories: [ Linux ]
tags: [ Linux ]
# 댓글 기능
comments: False
---

##### grep
---

- 특정 문자열을 찾을 때 사용
- 

```bash
$ grep "abc" test.txt
```

##### curl

---

- http를 활용하여 return 값 확인
- -o = 명령 결과 저장
- -s  = silent

```bash
$ curl "www.google.com"
$ curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest | grep browser_download_url
```

##### cut

---

- 파일, 입력 받은 문자열 split → indexing
- -d = 지정 문자를 구분자로 사용
- -f = 필드 기준으로 잘라내기

```bash
$ cat sed-example.txt
> 
unix is great os. unix is opensource. unix is free os.
learn operating system.
unix linux which one you choose.
unix is easy to learn.unix is a multiuser os.Learn unix .unix is a powerful.

$ cut -c 2-4 < sed-example.txt
> 
nix
ear
nix
nix
```

```bash
$ curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest | grep browser_download_url | grep linux-amd64 | cut -d '"' -f 4
```

##### history

---

- 과거 command 이력들을 보여준다.
- tail 옵션을 주어 최근 N개까지 보여주기 가능

```bash
$ history | tail -5
```

##### wget

---

- 파일 다운로드를 도와준다.

```bash
$ wget url
```