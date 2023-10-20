---
layout: post
title:  "Tableau 자동화하기 with Python"
author: Kimuksung
categories: [ Tableau ]
tags: [ Tableau, Python]
image: assets/images/tableau.png
# 댓글 기능
comments: False
---

안녕하세요 오늘은 태블로 대시보드 슬랙 자동화하기를 해보려고 합니다.

금일은 대시보드 구성이 되어있다고 가정하고, 파이썬으로 Tableau Cloud 연동 및 이미지 추출하는 부분까지 진행해보았습니다.

결론만 말하면, 파이썬으로 원하는 View의 이미지를 추출은 하였으나 데이터베이스가 연동되어있는 View는 상황에 따라 제대로 못 불러오는 현상이 있습니다.
-> view의 project id 가 제대로 맵핑 되지 않는 것을 확인하였습니다.

이문제는 차후에 해결하고 지금까지의 진행 과정에 대해 서술 합니다.

Python과 Tableau를 연동하기 위해 TSC 라이브러리를 사용했습니다.

##### 1. Tableau Token 발급
---
- Python에서 연동시키기 위하여 현재 사용하고 있는 로그인 방식은 Api로 지원하지 않는다하여 Token을 사용하였습니다.

<a href="https://imgbb.com/"><img src="https://i.ibb.co/LtvGqRq/Untitled-70.png" alt="Untitled-70" border="0"></a>

<a href="https://ibb.co/17J5Vd9"><img src="https://i.ibb.co/sbJGS6F/Untitled-71.png" alt="Untitled-71" border="0"></a>


##### 2. 대시보드 원본 저장 설정
---
- 대시보드 접속 시 클라우드 환경에서는 지속적으로 데이터베이스를 로그인하라고 팝업창이 뜹니다.
- Python과 연동할 때 에러가 발생하기에, 없애주기 위하여 아래와 같이 설정합니다.

<a href="https://imgbb.com/"><img src="https://i.ibb.co/HXbdcJQ/Untitled-72.png" alt="Untitled-72" border="0"></a>
<a href="https://ibb.co/dcYrB33"><img src="https://i.ibb.co/ZJDVdww/Untitled-73.png" alt="Untitled-73" border="0"></a>
<a href="https://imgbb.com/"><img src="https://i.ibb.co/t8JcyYS/2023-10-20-3-05-59.png" alt="2023-10-20-3-05-59" border="0"></a>

##### 3. 사용자 언어 한국어 설정
---
- 이미지 추출 후 확인을 하여 보면 한글이 깨져 있는 것을 볼 수 있습니다.
- 언어가 설정되지 않고, 해당 대시보드의 한글로 설정된 사용자로 변경해주어야 합니다.

<a href="https://ibb.co/G0YNB0H"><img src="https://i.ibb.co/376QG7R/Untitled-75.png" alt="Untitled-75" border="0"></a>
<a href="https://ibb.co/M19f7dy"><img src="https://i.ibb.co/ykVpq2m/Untitled-76.png" alt="Untitled-76" border="0"></a>

##### 4. 오류를 잡기 위한 캐싱 해제
---
- 위 과정을 해결하기 위해 여러 방법을 변경해보고 시도해보았으나, 계속하여 변화없는 이미지가 출력되었습니다.
- 처음에 파일을 잘못 부른것으로 착각하였으나, 캐싱되어서 과거의 이미지를 계속 호출한다고 합니다.
- 아래와 같이 캐싱을 풀어주어서 처리합니다.

```python
image_req_option = TSC.ImageRequestOptions(
		imageresolution=TSC.ImageRequestOptions.Resolution.High, maxage=1
	)

server.views.populate_image(view_item, req_options=image_req_option)
```

##### 5. Python 구성
---
1. ini 파일 구성
    
    ```python
    # config.ini
    [Tableau]
    server_url = https://url.tableau.com/
    token_name = token_id
    token_value = token_pw
    site_id = tableau url 값 중 / 뒤 값
    ```
    
    ```python
    from configparser import ConfigParser
    import tableauserverclient as TSC
    
    def readconfig(section, key):
    	config = ConfigParser()
    	config.read("config.ini")
    	return config.get(section, key)
    
    def readtableau():
    	tableau_infos = ["server_url", "token_name", "token_value", "site_id"]
    	return tuple(map(lambda x: readconfig("Tableau", x), tableau_infos))
    ```
    
2. 태블로 정보 추출
    
    ```python
    def gettableau(server):
    	with server.auth.sign_in(tableau_auth) as a:
    		projects, pagination = server.projects.get()
    		project_metadata = dict()
    		for project in projects:
    			project_metadata[project.name] = project.id
    
    		view_metadata = dict()
    		for view in TSC.Pager(server.views):
    			view_metadata[view.name] = (view.project_id, view.id)
    
    	return project_metadata, view_metadata
    ```
    
3. 이미지 추출
    
    ```python
    if __name__ == "__main__":
    	server_url, token_name, token_value, site_id = readtableau()
    
    	tableau_auth = TSC.PersonalAccessTokenAuth(token_name, token_value, site_id=site_id)
    	server = TSC.Server(server_url, use_server_version=True)
    	server.auth.sign_in(tableau_auth)
    	project_metadata, view_metadata = gettableau(server)
    
    	image_req_option = TSC.ImageRequestOptions(
    		imageresolution=TSC.ImageRequestOptions.Resolution.High, maxage=1
    	)
    
    	with server.auth.sign_in(tableau_auth) as a:
    		req_views = ['view_name', .. ]
    		req_projects = ['project_name']*len(req_views)
    
    		for i, (view_name, project_name) in enumerate(zip(req_views, req_projects)):
    			try:
    				if view_name in view_metadata:
    					print(f'start {i} dashboard')
    					project_id, view_id = view_metadata[view_name]
    					if project_id == project_metadata[project_name]:
    						view_item = server.views.get_by_id(view_id)
    						server.views.populate_image(view_item, req_options=image_req_option)
    						with open(f'./view_image{i}.png', 'wb') as f:
    							f.write(view_item.image)
    					print(f'end {i} dashboard')
    			except Exception as e:
    				print(f'{view_name} Error / {e}')
    
    	server.auth.sign_out()
    ```
    

##### 6. 지속적인 오류 발생
---
- 하나의 Project 대시보드에 6개의 View를 구성하였습니다.
- 위와 같은 방법으로 모든 View를 접속할 때에도 별도의 로그인을 하지 않습니다.
- Python으로 호출 시, 특정 이미지는 계속하여 불러와지고 나머지는 에러가 발생합니다.
- 디버깅으로 오류를 찾아가고 있는데 현재 tableau의 project의 id와 name에 존재하는 값과 view의 projectid가 비매칭되는 현상이 나타나는중입니다.

```python
400074: 잘못된 요청
		뷰 ’7a0dd161-9fe9-493c-87d9-63beda57f614’의 이미지를 쿼리하는 동안 문제가 발생했습니다.
```

```python
projects, pagination = server.projects.get()
[(project.id, project.name) for project in projects]

all_views, pagination = server.views.get()
[(view.name, view.id, view.project_id) for view in all_views]
```