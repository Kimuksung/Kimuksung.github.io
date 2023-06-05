---
layout: post
title:  "Python-Json"
author: Kimuksung
categories: [ Python ]
tags: [ Python, Json ]
image: assets/images/python.png
comments: False
---


##### loads()
---
- Json 문자열 → Python Object

```jsx
import json

json_string = ''
json_object = json.loads(json_string)
```

##### load()
---
- Json file → Python Object

```jsx
import json

with open('test.json') as f:
	json_object = json.load(f)

```

##### Dumps()
---
- Python Object → Json 문자열

```jsx
import json

json_string = json.dump(json_object)
```