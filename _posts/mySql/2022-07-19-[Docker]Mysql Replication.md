---
layout : post
title : "Docker 사용하여 Mysql Replication"
category : MySQL
---

## Replication
DBMS를 사용하는 경우, 데이터를 백업 하여 보호하기 위해서나 Write/Read 로 나눠서 부하를 분산하여 사용하기도 한다.
`Master`, `Slave` 두가지로 나뉘는데 역할은 다음과 같다.
* MySQL 버전이 다른 경우 Slave 서버가 상위 버전 이여야 한다

### `Master`
웹서버로 부터 등록/수정/삭제 요청시 `Binarylog`를 생성하여 Slave 서버로 전송한다.


### `Slave`
Master DBMS로 부터 전달받은 `Binarylog`를 데이터로 반영하게 됩니다


### `Binarylog` (아카이브 로그)
DML, DDL 등의 모든 이벤트를 저장하는 로그이다.
Slave 구축시 이 로그 파일을 읽어서 사용한다.


## Replication 설정

master 와 slave 를 docker 를 이용하여 생성하려 한다.

[docker-compose와 cnf 파일 repo](https://github.com/hanbong5938/docker-compose-ex)

`master`의 config_file.cnf
```dotenv
[mysqld]
log-bin=master
server-id=1
```

`slave`의 config_file.cnf
```dotenv
[mysqld]
server-id=2
```

`log-bin` 은 업데이트되는 모든 query들을 바이너리 파일에 로그로 남기겠다는 의미이다.
기본적으로 바이너리 파일은 MySQL의 datadir인 /var/lib/mysql/ 에 호스트명.000001, 호스트명.000002 형태로 생성된다.
위에서처럼 log-bin=... 식으로 뒤에 덧붙이면 바이너리 파일의 경로와 파일명의 접두어를 입맛에 맞게 정할 수 있다. log-bin=master 이라 설정하였으므로 master.000001, master-bin.000002 형태의 바이너리 파일이 생성될 것이다.

`server-id` 는 Replication Group에서 식별하기 위한 서버의 고유 ID 값이다. master, client 각각 다르게 해주어야 한다.
1~(2^32)-1내의 숫자로 설정하면 된다.

아래와 같이 파일에 추가 설정도 가능하다.
```dotenv
[mysqld]
server-id=2
master-host             = master
master-user             = slave_user
master-password     = test
```



config_file.cnf 파일이 적용 되었는지 확인하기 위해 docker에 접속한다.
```shell
docker exec -it mysql-master /bin/bash
mysql -u root -p  
```

서버 id 적용 확인
```mysql
SELECT @@server_id as SERVER_ID;
```

아래의 명령어로 확인할 수 있다.
```mysql
SHOW MASTER STATUS\G
```

이처럼 뜨면 적용이 완료 된것이다.
```shell
*************************** 1. row ***************************
             File: master.000001
         Position: 157
     Binlog_Do_DB:
 Binlog_Ignore_DB:
Executed_Gtid_Set:
1 row in set (0.00 sec)
```
File 은 현재 바이너리 로그 파일명이고, Position 은 현재 로그의 위치를 나타낸다.
쿼리를 실행하여 데이터 변경이 일어나면 position 이 증가된다.

만약 아래와 같이 에러가 뜬다면 읽기 전용으로 설정해주면 된다.
```shell
mysql: [Warning] World-writable config file '/etc/mysql/conf.d/my.cnf' is ignored.
```

```shell
chmod 0444 my.cnf
```

### 유저 생성

```mysql
CREATE USER 'slave_user'@'%' IDENTIFIED BY 'test';  
GRANT REPLICATION SLAVE ON *.* TO 'slave_user'@'%';  
```

### 테스트 테이블 생성

```mysql
CREATE DATABASE test;
USE test;
CREATE TABLE temp ( num INT(1), PRIMARY KEY (num) );
```

### dump

docker /bin/bash 에 접속한 상태에서 아래 명령어를 실행하여 dump 한다.
```shell
mysqldump -u root -p test > dump.sql
```

이후 docker bash 에서 나온 다음 아래 명령어를 입력해주면 현재 폴터에 dump.sql 이 생성된다.
```shell
docker cp mysql-master:dump.sql dump.sql
```

### slave 에 dump 적용

sql 파일을 mysql-slave-1 에도 복사 해준 다음
```shell
docker cp dump.sql mysql-slave-1:.
docker exec -it mysql-slave-1 /bin/bash
```

scheme를 생성한다.
```mysql
CREATE DATABASE test;
```

test 스키마에 dump.sql를 입력해준다.
```shell
mysql -u root -p test < dump.sql
```

dump 이후에 auto.cnf 라는 파일에 UUID 정보가 저장되어 있어 master DB 의 데이터를 dump 해올때 해당 파일도 함께 복사되어 아래와 같은 에러가 발생할 수 있다.
```shell
The slave I/O thread stops because master and slave have equal MySQL server UUIDs; these UUIDs must be different for replication to work.
```

미리 Master 서버의 UUID 가 들어있는 auto.cnf를 제거 후 master를 재시작해주는 것이 좋다.
master bash로 들어가 아래 명령어를 통해 제거한다.
```shell
rm -rf /var/lib/mysql/auto.cnf
```

### master - slave 연결

master 에 접속하여 아래 데이터를 확인한다.
position과 file 을 이용하여 slave를 연결 해야한다.
```mysql
SHOW MASTER STATUS\G
```

slave에 접속하여 master의 정보를 입력해준다.
```mysql
CHANGE MASTER TO MASTER_HOST='mysql-master', MASTER_USER='slave_user', MASTER_PASSWORD='test', MASTER_LOG_FILE='master.000005', MASTER_LOG_POS=157; 
```

입력 내용
- MASTER_HOST : master 서버의 호스트명
- MASTER_USER : master 서버의 mysql에서 REPLICATION SLAVE 권한을 가진 User 계정의 이름
- MASTER_PASSWORD : master 서버의 mysql에서 REPLICATION SLAVE 권한을 가진 User 계정의 비밀번호
- MASTER_LOG_FILE : master 서버의 바이너리 로그 파일명
- MASTER_LOG_POS : master 서버의 현재 로그의 위치

이후 slave 에서 아래 명령어를 실행하여 slave 연결을 확인한다.
```mysql
SHOW SLAVE STATUS\G  
```

아래의 결과로 에러가 없는 것을 확인한다.
```shell
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: mysql-master
                  Master_User: slave_user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master.000005
          Read_Master_Log_Pos: 157
               Relay_Log_File: c951b41b2647-relay-bin.000002
                Relay_Log_Pos: 323
        Relay_Master_Log_File: master.000005
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 157
              Relay_Log_Space: 540
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: 99e39ff2-0701-11ed-82b2-0242ac130002
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
       Master_public_key_path:
        Get_master_public_key: 0
            Network_Namespace:

```

만약 연결시 아래 에러가 발생하는 경우
```shell
Unable to load authentication plugin 'caching_sha2_password'
```

```mysql
alter user 'slave_user'@'%' identified with mysql_native_password by 'test';
```
