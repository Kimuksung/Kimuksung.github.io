---
layout: post
title:  "redshift 권한 부여"
author: Kimuksung
categories: [ redshift ]
tags: [ redshift 권한 부여 ]
image: "https://ifh.cc/g/rD7FlS.png"
# 댓글 기능
comments: False
# feature tab 여부
featured: False
# story에서 보이게 할지 여부
hidden: False
---

<br>

##### 유저 정보 조회
---
- id
- 유저 이름
- 슈퍼 유저 여부
- 비밀번호 만료일

```sql
select 
    usesysid as user_id,
    usename as username,
    usecreatedb as db_create,
    usesuper as is_superuser,
    valuntil as password_expiration
from pg_user
order by user_id;
```

##### schema, table 권한 조회
---
- 특정 유저의 dml 권한 조회
```sql
SELECT
    *
FROM
    (
    SELECT
        schemaname
        ,objectname
        ,usename
        ,HAS_TABLE_PRIVILEGE(usrs.usename, fullobj, 'select') AS sel
        ,HAS_TABLE_PRIVILEGE(usrs.usename, fullobj, 'insert') AS ins
        ,HAS_TABLE_PRIVILEGE(usrs.usename, fullobj, 'update') AS upd
        ,HAS_TABLE_PRIVILEGE(usrs.usename, fullobj, 'delete') AS del
        ,HAS_TABLE_PRIVILEGE(usrs.usename, fullobj, 'references') AS ref
    FROM
        (
        SELECT schemaname, 't' AS obj_type, tablename AS objectname, QUOTE_IDENT(schemaname) || '.' || QUOTE_IDENT(tablename) AS fullobj FROM pg_tables
        WHERE schemaname !~ '^information_schema|catalog_history|pg_'
        UNION
        SELECT schemaname, 'v' AS obj_type, viewname AS objectname, QUOTE_IDENT(schemaname) || '.' || QUOTE_IDENT(viewname) AS fullobj FROM pg_views
        WHERE schemaname !~ '^information_schema|catalog_history|pg_'
        ) AS objs
        ,(SELECT * FROM pg_user) AS usrs
    ORDER BY fullobj
    )
WHERE (sel = true or ins = true or upd = true or del = true or ref = true)
    and usename = '{username}';
```

##### 권한 부여
---
- schema 권한 부여
- select 권한 부여
- 특정 유저가 만든 테이블 권한 부여
- view 권한 부여

```sql
#grant schema usage;   
GRANT USAGE ON 
SCHEMA {schema_name} TO {id};

# grant select
grant select on all tables 
in schema {schema_name} to {id};

# 마트 구성 시 새로운 테이블 생성하여도 권한 부여
alter default privileges
for user {create_user}
in schema {schema_name}
grant select on tables
to {user};

# grant view
GRANT SELECT ON <view_name> TO <username>;
```

##### 유저 그룹화
---
```sql
create group {group_name} with user ;

alter group {group_name}
add user {id};

alter default privileges 
in schema {schema_name} 
grant select on tables to group {group_name};

# select
SELECT usename, groname 
	FROM pg_user, pg_group
		WHERE pg_user.usesysid = ANY(pg_group.grolist)
			AND pg_group.groname in (SELECT DISTINCT pg_group.groname from pg_group);
```

##### 슈퍼 유저
---
- Database 슈퍼 유저 권한 부여
- 슈퍼 유저 권한 해제

```sql
# 슈퍼 유저 만들기
ALTER USER {username} CREATEUSER;

# 슈퍼 유저 해제
alter user {id} nocreateuser;
```