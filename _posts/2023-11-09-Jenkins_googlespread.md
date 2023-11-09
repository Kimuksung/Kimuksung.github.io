---
layout: post
title:  "Fact table 구글 시트에 반영하기 with Jenkins"
author: Kimuksung
categories: [ Jenkins ]
tags: [ Jenkins, Python ]
image: "https://i.ibb.co/xzTs9SF/jenkins.png"
comments: False
---

##### 요청 사항
---
- 매 시간 마다 구글 시트에 사내 지표를 상세히 알고 싶다.  
- 빠른 작업을 위해 Jenkins에 구성

<br>

작업 계획
1. 연동 파일 작성
2. redshift 연결을 통하여 Dataframe 추출
3. 구글 시트에 반영
4. 동적으로 만들기 위하여 script 화

<br>

##### 1. 연동 파일 작성
---
- env 폴도 내에 ini 파일을 생성하여 계정 정보 및 연동 정보 관리
- 연동 파일까지 포함해서 관리하기 위해 Pythonpath ⇒ 연동 파일 + 모듈 상위 파일로 지정
- 연결 host/db/유저/패스워드

```python
[dw_user]
host = redshift.amazonaws.com
database = db_name
user = user_name
pw = pw
scope = googlespread_scope를 , 로 구분하여 나열
```

- requirements.txt

```bash
pandas==2.0.2
redshift-connector==2.0.911
gspread==5.7.2
gspread-dataframe==3.3.0
oauth2client==4.1.3
```

##### 2. Redshift 연결
---
- redshift_connector, pandas 라이브러리 사용

```python
class WclubDataMart:
    def __init__(self):
        self._host = config.get('dw_user', 'host')
        self._database = config.get('dw_user', 'database')
        self._dw_id = config.get('dw_user', 'user')
        self._dw_pw = config.get('dw_user', 'pw')

    def wclub_redshift_sql_result(self, sql: str) -> Optional[pd.DataFrame]:
        assert type(sql) is str

        conn = redshift_connector.connect(
            host=self._host,
            database=self._database,
            user=self._dw_id,
            password=self._dw_pw,
            auto_create=True
        )

        cursor: redshift_connector.Cursor = conn.cursor()
        cursor.execute(sql)
        result = cursor.fetch_dataframe()

        return None if result.empty else result
```

##### 3. 구글 시트 반영
---
- gspread, gspread_dataframe, oauth2client 라이브러리 사용
- 구글 스프레드의 row, col은 1,1부터 시작한다.
- clear 옵션을 주어 삭제 후 업로드하도록 구성
- index는 표현할 이유가 없어서 제외, 빠르게 표현하기 위해 header 추가

```python
def upload_df_sheet(self, df, sheet_name, row=1, col=1, sheet_url='url',
            clear=True, include_index=False, include_column_header=True):

        scope = config.get('dw_user', 'scope').split(",")
        json_file_name = '/environment/google_sheet.json'
        credentials = ServiceAccountCredentials.from_json_keyfile_name(json_file_name, scope)
        gc = gspread.authorize(credentials)

        doc = gc.open_by_url(sheet_url)
        worksheet = doc.worksheet(sheet_name)
        if clear:
            worksheet.clear()

        # Dataframe -> Google Sheet    
        gd.set_with_dataframe(worksheet, df, row, col, include_index, include_column_header)
				return True
```

##### 4. 동적 script 만들기
---
- bash file을 만들어 동적으로 생성
- 변수 이름은 대문자로 ( 대소문자 불가 )
- 동적으로 할당하기 위해서는 `“$variable”`
- Parser를 이용하여 동적으로 실행

```bash
#!/bin/bash
SQL="
select 
    *
from table
limit 1;
"

SHEET_NAME="sheet_name"

python3 /var/jenkins_home/wclub-dw/dw_sql_upload.py --sql "$SQL" --sheet_name "$SHEET_NAME"
```

```bash
if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('--sql', type=str)
    parser.add_argument('--sheet_name', type=str)
    args = parser.parse_args()

    info = DataMart()
    result = info.wclub_redshift_sql_result(args.sql)

    info.upload_df_sheet(df=result, sheet_name=args.sheet_name)
```

