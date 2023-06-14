---
layout: post
title:  "Linux Command"
author: Kimuksung
categories: [ Linux ]
tags: [ Linux ]
# 댓글 기능
comments: False
image: "https://ifh.cc/g/1dT5w9.jpg"
---

리눅스 사용하면서 썻던 명령어들을 정리하여두었습니다.  


##### ‘>’ 와 ‘>>’
---
- `>` : 명령어 뒤에 나오는 File에 Write or Overwrite
- `>>` : 명령어 뒤에 나오는 파일에 추가할 때 사용 Append

##### globs
---
Glob Patterns로, 패턴 매칭 역할.
- `?`
    - 정확하게 한글자만 매칭
    - a,ab,ac,ad,abc,abcd 파일이 있다고 할 때 → ab,ac,ad 파일만 표현
        
        ```bash
        $ ls data/ma??.txt
        > data/main.txt, data/mail.txt ..
        ```
        
- `*`
    - 문자열의 길이와 상관없이 어떤 문자열과 부합(공백 포함)
    - 특수 기호 = linux `/`, window `\` 은 제외
    - a,ab,abc,abc.txt,abc.mp3 → 모든 파일을 다 찾을 수 있다.
    
    ```bash
    $ ls a*
    ```
    
- `**`
    - 0개 이상의 하위 Directory를 매칭시켜 검색
    - 하위 디렉토리 아래에서 특정 데이터를 찾고 싶은 경우
    
    ```bash
    $ ls **/*.md
    ```
    
- `{}`
    - 선택지를 모두 합쳐서 찾을 때
    
    ```bash
    $ ls {*.md,*.MD}
    ```
    
##### grep
---

- 특정 문자열을 찾을 때 사용
- 

```bash
$ grep "abc" test.txt
$ echo 'test 123' > test.txt & cat test.txt | grep 12
```

##### curl
---
- http를 활용하여 return 값 확인
- -o = 명령 결과 저장
- -s = silent
- -L = Redirect Header를 무시
```bash
$ curl "www.google.com"
$ curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest | grep browser_download_url
$ curl -o /tmp/launches.json -L url
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

##### wget
### NC

---

Netcat = nc로 TCP/UDP을 사용하여 서버 연결 확인 및 서버가 되어 확인 가능하다.

- 서버에 연결여부 확인
    - nc [서버 IP] [대상 PORT]
    - nc -v [서버 IP] [PORT] = -v 옵션은 상세한 정보 요청
    - nc -zv [IP,URL] [PORT] = Connection 연결 없이 report
        
        ```bash
        $ nc ip port
        > 
        #성공
        대기상태
        #실패
        Ncat: No route to host.
        
        $ nc -v ip port
        > 
        #성공
        Ncat: Version x.xx
        Ncat: Connected to ip:port.
        #실패
        Ncat: No route to host.
        
        $ nc -zv ip port
        ```
        
- 서버가 되어 연결 여부 확인
    - nc -l [PORT] = 서버 구성
    
    ```bash
    # Server
    $ nc -l 30000
    
    $ Client
    $ nc ip port
    ```