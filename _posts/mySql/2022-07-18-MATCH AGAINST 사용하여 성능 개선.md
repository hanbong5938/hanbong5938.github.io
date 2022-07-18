---
layout : post
title : "MATCH AGAINST 사용하여 성능 개선"
category : MySQL
---

## 원인
mysql에서 content 검색시 `like` 를 사용하거나 `in`, `<>` 을 사용하여 검색하는 경우 
모든 데이터를 검색해야 하기에 access type 이 All 로 실행이 된다.

### `like` 사용시
```mysql
explain format = json

select *
from company c
join employer e on c.id = e.company_id
where c.name like '%이름%'
;

```

### 결과 값
```json
{
  "query_block": {
    "select_id": 1,
    "table": {
      "table_name": "c",
      "access_type": "ALL",
      "possible_keys": ["PRIMARY"],
      "rows": 1,
      "filtered": 100,
      "attached_condition": "c.`name` like '%이름%'"
    },
    "block-nl-join": {
      "table": {
        "table_name": "e",
        "access_type": "ALL",
        "possible_keys": ["company_with_pk"],
        "rows": 1,
        "filtered": 100
      },
      "buffer_type": "flat",
      "buffer_size": "1Kb",
      "join_type": "BNL",
      "attached_condition": "e.company_id = c.`id`"
    }
  }
}
```

### `in` 사용시
```mysql
explain format = json
    
select *
from company c
join employer e on c.id = e.company_id
where c.name in ('이름')
```

### 결과
```json
{
  "query_block": {
    "select_id": 1,
    "table": {
      "table_name": "c",
      "access_type": "ALL",
      "possible_keys": ["PRIMARY"],
      "rows": 1,
      "filtered": 100,
      "attached_condition": "c.`name` = '이름'"
    },
    "block-nl-join": {
      "table": {
        "table_name": "e",
        "access_type": "ALL",
        "possible_keys": ["company_with_pk"],
        "rows": 1,
        "filtered": 100
      },
      "buffer_type": "flat",
      "buffer_size": "1Kb",
      "join_type": "BNL",
      "attached_condition": "e.company_id = c.`id`"
    }
  }
}
```

### `<>` 사용시
```mysql
explain format = json
    
select *
from company c
join employer e on c.id = e.company_id
where c.name <> '이름'
;
```

```json
{
  "query_block": {
    "select_id": 1,
    "table": {
      "table_name": "c",
      "access_type": "ALL",
      "possible_keys": ["PRIMARY"],
      "rows": 1,
      "filtered": 100,
      "attached_condition": "c.`name` <> '이름'"
    },
    "block-nl-join": {
      "table": {
        "table_name": "e",
        "access_type": "ALL",
        "possible_keys": ["company_with_pk"],
        "rows": 1,
        "filtered": 100
      },
      "buffer_type": "flat",
      "buffer_size": "1Kb",
      "join_type": "BNL",
      "attached_condition": "e.company_id = c.`id`"
    }
  }
}
```

이를 해결하기 위해서 

`FULLTEXT`를 사용하면 조금 더 효율적으로 처리가 가능하다.

`FULLTEXT` 사용하기 위해서는 index를 생성해줘야한다.
### 
```mysql
ALTER TABLE company
    ADD FULLTEXT(name);
```

생성 이후

### 최소 글자수 확인
```mysql
show variables like '%ft_min%';
```
위의 쿼리를 이용하여 MATCH AGAINST 사용시 최소 글자 검색 수를 조회할 수 있다.
`ft_min_word_len`의 숫자보다 작은 경우 무시되게 된다.

my.cnf 이나 my.ini 파일에서 `ft_min_word_len=4`(innodb 인경우 `innodb_ft_min_token_size=2`) 를 원하는 숫자로 수정 해준 뒤 mysql 재시작 해주면 적용된다.

만약 이미 인덱스가 생성되어 있는 경우, 아래의 쿼리를 사용하여 index 재생성이 필요하다
```mysql
REPAIR TABLE company QUICK;
```


```mysql
explain format = json
select *
from company c
join employer e on c.id = e.company_id
where MATCH(c.name) AGAINST('이름');
```
```json
{
  "query_block": {
    "select_id": 1,
    "table": {
      "table_name": "c",
      "access_type": "fulltext",
      "possible_keys": ["PRIMARY", "name"],
      "key": "name",
      "key_length": "0",
      "used_key_parts": ["name"],
      "rows": 1,
      "filtered": 100,
      "attached_condition": "(match c.`name` against ('이름'))"
    },
    "table": {
      "table_name": "e",
      "access_type": "ALL",
      "possible_keys": ["company_with_pk"],
      "rows": 1,
      "filtered": 100,
      "attached_condition": "e.company_id = c.`id`"
    }
  }
}
```

## 자연어 검색

## Boolean 모드 검색
```mysql
select *
from company c
join employer e on c.id = e.company_id
where MATCH(c.name) AGAINST('이름1 이름2' IN BOOLEAN MODE);
```


### OR 데이터 조회
```mysql
select *
from company c
         join employer e on c.id = e.company_id
where MATCH(c.name) AGAINST('이름1 이름2' IN BOOLEAN MODE);
```
### AND 데이터 조회
```mysql
select *
from company c
         join employer e on c.id = e.company_id
where MATCH(c.name) AGAINST('이름1 +이름2' IN BOOLEAN MODE);
```
### 제외하여 조회
```mysql
select *
from company c
         join employer e on c.id = e.company_id
where MATCH(c.name) AGAINST('이름1 -이름2' IN BOOLEAN MODE);
```

### `이름%`포함 데이터 조회 
```mysql
select *
from company c
         join employer e on c.id = e.company_id
where MATCH(c.name) AGAINST('이름*' IN BOOLEAN MODE);
```
### 띄어쓰기 포함 데이터 조회
```mysql
select *
from company c
         join employer e on c.id = e.company_id
where MATCH(c.name) AGAINST('"이 름"' IN BOOLEAN MODE);
```

