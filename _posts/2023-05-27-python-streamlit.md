---
layout: post
title:  "Streamlit(Python Dashboard)"
author: Kimuksung
categories: [ Streamlit, Python ]
tags: [ Streamlit, Python, Dashboard ]
image: "https://ifh.cc/g/P0Dxnd.png"
comments: False
---

회사에서 Dashboard를 요청하게 되어 고민하던 도중 아래와 같이 많은 대시보드 library를 검색하게 되었다.  
전사 직원 누구나 어디서든 손쉽게 접근이 가능할 수 있도록이라는 목적에 맞추어 **웹 배포**가 가능한 Streamlit을 활용하여 구성하도록 결정하였다.  
Streamlit은 웹배포가 깃허브 연동 혹은 Docker로 구성이 가능하다.  
<br/>

##### Python 시각화 [라이브러리](https://zzsza.github.io/development/2018/08/24/data-visualization-in-python/)
---
- matplotlib
    - 기본 수준의 그리기 제공
- pandas
    - matplolib에 기반한 UI 제공
    - matplotlib을 기반으로 여러 색상, 통계용 차트 추가
- Seaborn
    - matplotlib을 기반으로 여러 색상, 통계용 차트 추가
- Streamlit
    - Web 배포가 가능하며, Dataframe을 기반으로 손쉽게 구현 가능
    - 주로 ML에서 데이터를 보기 위해 pyplot과 함께 사용된다.
<br/>
<br/>

##### 시각화가 필요한 이유?
---
- 시각화는 뇌에 가장 높은 인상 전달
- 차트로 다루기 어렵기 때문에 시각화 필요
- `동일한 수치`라도 다양한 시각화 방법을 통해 그려지고 `해석될 수 있다`
    
    ![데이터시각화필요이유2.png](https://ifh.cc/g/r1f8nD.jpg)

<br/>

##### Streamlit 설치 및 실행
---
```bash
# install
$ pip3 install streamlit
```

```bash
# execute
$ streamlit run app.py
```

- 라이브러리 - [링크](https://docs.streamlit.io/library/api-reference/charts/st.altair_chart)
- 상세 정보 - [링크](https://blog.zarathu.com/posts/2023-02-01-streamlit/)

##### 시각화 해보기
---
- Dataframe 보여주기

```sql
import streamlit as st

st.write(df)
```
![dataframe](https://ifh.cc/g/8c1YYd.png)

##### Select box

```bash
option = st.selectbox('Select Line',second_category.keys())
```

![dataframe](https://ifh.cc/g/GRMDa1.png)

##### Multi box

```bash
option = st.multiselect("Select class", df['col'].unique().tolist()))
```

![dataframe](https://ifh.cc/g/AS2gQ8.png)

##### Line Chart

- st Library 활용

```bash
st.line_chart(df, use_container_width=True)
```

![dataframe](https://ifh.cc/g/GRMDa1.png)

##### [altair_chart](https://discuss.streamlit.io/t/how-to-build-line-chart-with-two-values-on-y-axis-and-sorded-x-axis-acording-string/9490)

- Bar
    
    ```bash
    bars = alt.Chart(df).mark_bar().encode(
            x=alt.X("name"),
            y=alt.Y("value")
        ).properties(title="처음 성공한 매칭의 연결 시간")
    st.altair_chart(bars, use_container_width=True)
    ```
    
    ![dataframe](https://ifh.cc/g/Fo2KgZ.png)
    
    ![dataframe](https://ifh.cc/g/lLA0gD.png)
    
- line
    - 여러 값을 한번에 보여주는 것이 가능하다.
    
    ```bash
    lines = alt.Chart(df).mark_line().encode(
      x=alt.X('dates'),
      y=alt.Y('value'),
      color=alt.Color("name")
    ).properties(title="일별 차트 [cnt-유저수/sum-총합)")
    st.altair_chart(lines, use_container_width=True)
    ```
    
    ![dataframe](https://ifh.cc/g/9mjkcH.png)
    
##### iframe 연동하기
    ---
    ```python
    <iframe
      src="link"
      height="1000"
      style="width:100%;border:none;"
    ></iframe>
    ```