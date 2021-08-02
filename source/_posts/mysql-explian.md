---
layout: post
title: Mysql Explain
catalog: true
header-img: 'https://i.imgur.com/avC1Xor.jpg'
date: 2019-10-13 13:55:28
subtitle:
tags:
 - MySQL
---


> 해당 글은 [MySQL 5.7 완벽 분석](http://www.yes24.com/Product/Goods/72270172?)을 정리한 내용입니다.

## MySQL 옵티마이저 구조

![](https://github.com/cheese10yun/TIL/raw/master/assets/mysql-sql-executor-flow.png)

MySQL 옵티마이저는 비용 기반으로 어떤 실행 계획으로 쿼리를 실행했을 때 비용이 얼마나 발생하는지를 계산해하여 비용이 가장 적은 것을 택하게 됩니다. 어디까지나 추정 값이므로 정확한 비용은 실행 전까지 정확하게는 알 수 없습니다.

## EXPLAIN
EXPLAIN은 MySQL 서버가 어떠한 쿼리를 실행할 것인가, **즉 실행 계획이 무엇인지 알고 싶을 때 사용하는 기본적인 명령어이다.** 5.6 부터는 JSON 형식의 출력도할 수 있게 되었다. MySQL Workbench와의 결합으로 시각화할 수도 있습니다. 이 부분은 아래에서 다루겠습니다.



## 테이블 구조
![](https://github.com/cheese10yun/TIL/blob/master/assets/smaple-erd.png?raw=true)

`EXPLAIN`을 살펴 보기 이전 ERD를 살펴보겠습니다. member는 회원 정보이고, orders는 주문 목록, transaction은 해당 주문의 거래 정보입니다.

## EXPLAIN 결과

EXPLAIN은 MySQL 서버가 어떠한 쿼리를 실행할 것인가, 즉 실행 계획이 무엇인지 알고 싶을 때 사용하는 기본적인 명령어이다. 5.6 부터는 JSON 형식의 출력도할 수 있게 되었다


```sql
explain
select m.*, o.*, t.* from member  m
inner join orders o on m.id = o.member_id
inner join transaction t on o.transaction_id = t.id
where m.id in (1, 2, 33)
```

| id  | select_type | table | partitions | type     | possible_keys | key          | key_len | ref                       | rows | filtered | Extra         |
| --- | ----------- | ----- | ---------- | -------- | ------------- | ------------ | ------- | ------------------------- | ---- | -------- | ------------- |
| '1' | 'SIMPLE'    | 'm'   | NULL       | 'range'  | 'PRIMARY'     | 'PRIMARY'    | '8'     | NULL                      | '3'  | '100.00' | 'Using where' |
| '1' | 'SIMPLE'    | 'o'   | NULL       | 'ref'    | 'FKpktxw...'  | 'FKpktxw...' | '8'     | 'sample.m.id'             | '90' | '100.00' | NULL          |
| '1' | 'SIMPLE'    | 't'   | NULL       | 'eq_ref' | 'PRIMARY'     | 'PRIMARY'    | '8'     | 'sample.o.transaction_id' | '1'  | '100.00' | NULL          |


### table
어떤 테이블에 대한 접근을 표시하고 있는지는 table 필드에 표시되어있다. 

### id
id는 SELECT에 붙은 번호를 말한다. MySQL은 조인을 하나의 단위로 실행하기 때문에 id는 그 쿼리에 실행 단위를 식별하는 것이다. 따라서 조인만 수행하는 쿼리에서는 id는 항상 1이 된다.

### select_type
select_type은 항상 SIMPLE 이된다. 복잡한 조인을 해도 SIMPLE이 된다. 서브쿼리나 UNION이 있으면 id와 select_type이 변한다.

### partitions
partitions는 파티셔닝이 되어 있는 경우에 사용되는 필드이다. 이 쿼리에서 사용된 테이블을 모두 파티셔닝이 되어 있지 않기 때문에 이 필드가 모두 NULL로 출력되았다. 파티셔닝 되어 있는 경우는 반드시 이 필드를 확인하자 

### type
type은 접근 방식을 표시하는 필드다. 접근 방식은 테이블에서 어떻게 행데이터를 가져올것인가를 가리킨다. 위 EXPLAIN에서는 ALL, eq_ref, ref가 있는데 ALL, eq_ref는 조인시 기본 키나 고유키를 사용하여 하나의 값으로 접근(최대 1행만을 정확하게 패치), ref는 여러 개의 행을 패치할 가능성이 있는 접근을 의미한다. **접근 방식은 대상 테이블로의 접근이 효율적일지 여부를 판단하는 데 아주 중요한 항목이다.**

이들 접근 방식 가운데도 주의가 필요한 것은 ALL, index, ref_or_null이다. **ALL, index 두 가지는 테이블 또는 특정 인덱스가 전체 행에 접근하기 때문에 테이블 크기가 크면 효율이 떨어진다. ref_or_null의 경우 NUL이 들어있는 행은 인덱스의 맨 앞에 모아서 저장하지만 그 건수가 많으면 MySQL 서버의 작업량이 방대해진다. 다시 말해서 ALL 이외의 접근 방식은 모두 인덱스를 사용한다.**  

접근 방식이 ALL 또는 index인 경우는 그 쿼리로 사용할 수 있는 적절한 인덱스가 없다는 의미일 수도 있다. 위 쿼리에서 Country 테이블에 대한 접근은 ALL이지만 이는 WHERE 구의 조건을 지정하지 않았기 때문이다. 그러한 쿼리에서 드라이빙 테이블에 접근한다면 전체 행을 스캔 할수 밖에 없다.

| 접근 방식           | 설명                                                                                                                             |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| const           | 기본 키 또는 고유키에 의한 loockup(등가비교), 조인이 아닌 가장 외부의 테이블에 접근 하는 방식, 결과는 항상 1행이다. 단 기본 키, 고유 키를 사용하고 있으므로 범위 검색으로 지정하는 경우 const가 되지 않는다 |
| system          | 테이블에 1행밖에 없는 경우의 특수한 접근 방식                                                                                                     |
| ALL             | 전체 행 스캔, 테이블의 데이터 전체에 접근한다.                                                                                                    |
| index           | 인덱스 스캔, 테이블의 특정 인덱스의 전체 엔트리에 접근한다.                                                                                             |
| eq_ref          | 조인이 내부 테이블로 접근할 때 기본키 또는 공유 키에 의한 lookup이 일어난다. const와 비슷하지만 조인의 내부 테이블에 접근한다는 점이 다르다                                          |
| ref             | 고유 키가아닌 인덱스에 대한 등가비교, 여러 개 행에 접근할 가능성이 있다.                                                                                     |
| ref_or_null     | ref와 마찬가지로 인덱스 접근 시 맨 앞에 저장되어 있는 NULL의 엔트리를 검색한다.                                                                              |
| range           | 인덱스 특정 범위의 행에 접근한다                                                                                                             |
| fulltext        | fulltext 인덱스를 사용한 검색                                                                                                           |
| index_merge     | 여러 개인스턴스를 사용해 행을 가져오고 그 결과를 통합한다.                                                                                              |
| unique_subquery | IN 서브쿼리 접근에서 기본 키 또는 고유 키를 사용한다. 이 방식은 쓸데 없는 오버헤드를 줄여 상당히 빠르다.                                                                 |
| index_subquery  | unique_sunquery와 거의 비슷하지만 고유한 인덱스를 사용하지 않는 점이 다르다. 이 접근 방식도 상당히 빠르다                                                            |

### possible_keys
possible_keys 필드는 이용 가능성있는 인덱스의 목록이다.

### key 
possible_keys 필드는 이용 가능성있는 인덱스의 목록 중에서 실제로 옵티마이저가 선택한 인덱스가 key가 된다. 위 EXPLAN 에서는 County 테이블(첫 번째 행)의 **Key는 NULL 인데 이는 행 데이터를 가져오기 위해 인덱스를 사용할 수 없다는 의미이다.**

### key_len
key_len 필드는 선택된 인덱스의 길이를 의미한다. 중요한 필드는 아니지만 인덱스가 너무 긴 것도 비효율적이므로 기억해두자.

### rows
rows는 이 접근 방식을 사용해 몇 행을 가져왔는가를 표시한다. 최초에 접근하는 테이블에 대해서 쿼리 전체에 의해 접근하는 행 수, 그 이후에 테이블에 대해서는 1행의 조인으로 평균 몇 행에 접근했는가를 표시한다. 단 어디까지나 통계 값으로 계산한 값이므로 실제 행 수와 반드시 일치하지 않는다.

### filtered
filtered는 행 데이터를 가져와 거이에서 WHERE 구의 검색 조건이 적용되면 몇행이 남는지를 표시한다. 이 값도 통계 값 바탕으로 계산한 값이므로 현실의 값과 반드시 일치하지 않는다. 

### extra
Extra 필드는 옵티마이저가 동작하는데 대해서 우리에게 알려주는 힌트다. 이 필드는 EXPLAN을 사용해 옵티마이저의 행동을 파악할때 아주 중요하다.

Extra | 설명
------|---
Using where | 접근 방식을 설명한 것으로, 테이블에서 행을 가져온 후 추가적으로 검색조건을 적용해 행의 범위를 축소한 것을 표시한다.
Using index | 테이블에는 접근하지 않고 인덱스에서만 접근해서 쿼티를 해결하는 것을 의미한다. **커버링 인덱스로 처리됨 index only scan이라고도 부른다**
Using index for group-by | Using index와 유사하지만 GROUP BY가 포함되어 있는 쿼리를 커버링 인덱스로 해결할 수 있음을 나타낸다
Using filesort | ORDER BY 인덱스로 해결하지 못하고, filesort(MySQL의 quick sort)로 행을 정렬한 것을 나타낸다.
Using temporary | 암묵적으로 임시 테이블이 생성된 것을 표시한다.
Using where with pushed | 엔진 컨디션 pushdown 최적화가 일어난 것을 표시한다. 현재는 NDB만 유효
Using index condition | 인덱스 컨디션 pushdown(ICP) 최적화가 일어났음을 표시한다. ICP는 멀티 칼럼 인덱스에서 왼쪽부터 순서대로 칼럼을 지정하지 않는 경우에도 인덱스를 이용하는 실행 계획이다.
Using MRR | 멀티 레인지 리드(MRR) 최적화가 사용되었음을 표시한다.
Using join buffer(Block Nested Loop) | 조인에 적절한 인덱스가 없어 조인 버퍼를 이용했음을 표시한다.
Using join buffer(Batched Key Access) | Batched Key Access(BKAJ) 알고리즘을 위한 조인 버퍼를 사용했음을 표시한다.

## JSON 형식 EXPLAIN
```sql
explain format = json
select m.*, o.*, t.* from member  m
inner join orders o on m.id = o.member_id
inner join transaction t on o.transaction_id = t.id
where m.id in (1, 2, 33)
```

JSON 형식의 EXPLAIN은 기존의 표 형식 보다 출력되는 정보가 많다. 5.7 부터는 비용에 관한 정보를 부여주어서 표 형식에 비해 편리하다. JOSN 형식의 EXPLAIN에는 Extra 필드에서 Using WHERE 라고만 출력되는 것이 attached_condition로 나와서 구체적으러 어떤 조건이 적용되는지 알 수 있다. 

```json
{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "449.03"
    },
    "nested_loop": [
      {
        "table": {
          "table_name": "m",
          "access_type": "range",
          "possible_keys": [
            "PRIMARY"
          ],
          "key": "PRIMARY",
          "used_key_parts": [
            "id"
          ],
          "key_length": "8",
          "rows_examined_per_scan": 3,
          "rows_produced_per_join": 3,
          "filtered": "100.00",
          "cost_info": {
            "read_cost": "3.61",
            "eval_cost": "0.60",
            "prefix_cost": "4.21",
            "data_read_per_join": "6K"
          },
          "used_columns": [
            "id",
            "email",
            "name"
          ],
          "attached_condition": "(`sample`.`m`.`id` in (1,2,33))"
        }
      },
      {
        "table": {
          "table_name": "o",
          "access_type": "ref",
          "possible_keys": [
            "FKpktxwhj3x9m4gth5ff6bkqgeb",
            "FKrylnffj7sn97iepyqadlfnsg0"
          ],
          "key": "FKpktxwhj3x9m4gth5ff6bkqgeb",
          "used_key_parts": [
            "member_id"
          ],
          "key_length": "8",
          "ref": [
            "sample.m.id"
          ],
          "rows_examined_per_scan": 90,
          "rows_produced_per_join": 272,
          "filtered": "100.00",
          "cost_info": {
            "read_cost": "63.00",
            "eval_cost": "54.55",
            "prefix_cost": "121.76",
            "data_read_per_join": "279K"
          },
          "used_columns": [
            "id",
            "order_number",
            "member_id",
            "transaction_id"
          ]
        }
      },
      {
        "table": {
          "table_name": "t",
          "access_type": "eq_ref",
          "possible_keys": [
            "PRIMARY"
          ],
          "key": "PRIMARY",
          "used_key_parts": [
            "id"
          ],
          "key_length": "8",
          "ref": [
            "sample.o.transaction_id"
          ],
          "rows_examined_per_scan": 1,
          "rows_produced_per_join": 272,
          "filtered": "100.00",
          "cost_info": {
            "read_cost": "272.73",
            "eval_cost": "54.55",
            "prefix_cost": "449.03",
            "data_read_per_join": "820K"
          },
          "used_columns": [
            "id",
            "code",
            "partner_transaction_id",
            "payment_method_type"
          ]
        }
      }
    ]
  }
}
```

## Visual Explain

![](https://github.com/cheese10yun/TIL/blob/master/assets/mysql-visual-explain.gif?raw=true)

MySQL Workbench을 사용하면 Visual Explain를 활용할 수 있습니다. 가장 직관적으로 Explain을 확인할 수 있습니다.


![](https://github.com/cheese10yun/TIL/blob/master/assets/mysql-explain.png?raw=true)
