---
layout: post
title: mysql group replication.02
date: 2024-10-15 15:57 +0900
description: mysql group replication 실습 정리
image: https://d1.awsstatic.com/asset-repository/products/amazon-rds/1024px-MySQL.ff87215b43fd7292af172e2a5d9b844217262571.png
category: ["mysql"]
tags:
published: true
sitemap: true
author: kim-dong-jun99
---

## 도커를 이용한 group replication 실습 정리
> [공식 블로그](https://dev.mysql.com/blog-archive/setting-up-mysql-group-replication-with-mysql-docker-images/)를 참고해 진행한 group replication 구현 실습 과정을 정리한 글입니다.

### 도커 네트워크 생성
docker compose를 사용하지 않고, 개별 도커 이미지를 실행해 실습을 진행할 것이기에, 컨테이너들을 연결할 네트워크를 정의해야합니다.
```sh
docker network create cluster
```

### MySQL 도커 컨테이너 실행

공식 문서에서는 아래 명령어를 이용해 node1, node2, node3 MySQL 인스턴스를 실행합니다.
```sh
for N in 1 2 3
do docker run -d --name=node$N --net=cluster --hostname=node$N \
  -v $PWD/d$N:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=<root_password> \ # 노드 별로 디렉터리를 생성해 데이터를 저장합니다.
  mysql/mysql-server:8.0 \
  --server-id=$N \ # 노드 별로 다른 서버 아이디가 할당됩니다.
  --log-bin='mysql-bin-1.log' \
  --enforce-gtid-consistency='ON' \ # group replication을 위해 켜져야하는 설정값입니다.
  --log-replica-updates='ON' \ # slave는 deprecate되고 replica로 설정해야합니다.
  --gtid-mode='ON' \ # group repliation을 위해 켜져야하는 설정입니다.
  --transaction-write-set-extraction='XXHASH64' \
  --binlog-checksum='NONE' \
  --master-info-repository='TABLE' \
  --relay-log-info-repository='TABLE' \
  --plugin-load='group_replication.so' \ # group replication 플러그인 추가
  --relay-log-recovery='ON' \
  --group-replication-start-on-boot='OFF' \ # 반드시 OFF로 설정해야합니다.
  --group-replication-group-name='aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee' \ # UUID 값으로 SELECT UUID() 함수로 생성할 수 있습니다.
  --group-replication-local-address="node$N:33061" \ # 인스턴스의 호스트명과 포트번호 
  --group-replication-group-seeds='node1:33061,node2:33061,node3:33061' \ # 모든 인스턴의 호스트명과 포트번호가 명시될 필요는 없습니다.
  --loose-group-replication-single-primary-mode='ON' \ # 싱글 프라이머리 모드
  --loose-group-replication-enforce-update-everywhere-checks='OFF'
done
```

하지만 위 명령어를 이용해 도커 컨테이너를 실행하자 저는 플러그인 관련 오류가 발생하는 것을 확인했습니다.
MySQL 서버 초기화 설정 과정 중에 플러그인 관련 설정에서 오류가 발생했습니다, 

[플러그인 오류 관련 이슈](https://github.com/docker-library/mysql/issues/977)에서 발생한 문제에 대한 해결책을 찾을 수 있었고, 
다음 명령어를 먼저 실행하고, 공식 문서의 명령어를 실행해 성공적으로 도커 컨테이너를 실행할 수 있었습니다.

```sh
for N in 1 2 3
do docker run -d --name=node$N --net=cluster --hostname=node$N \
  -v $PWD/d$N:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=<root_password> \
  mysql/mysql-server:8.0 \
  --server-id=$N \
  --log-bin='mysql-bin-1.log'
done

echo "Wait 10s to initialize mysql data directory"

sleep 10
for N in 1 2 3
do
    docker logs node$N
    docker stop node$N
    docker remove node$N
    sudo rm $PWD/d$N/mysql.sock.lock
done
```

### Group Replication 설정

위 명령어들로 성공적으로 컨테이너를 실행했다면 Group Replication 관련 설정을 해야합니다.

```sh
docker exec -it node1 mysql -uroot -p<root_password> \
  -e "SET @@GLOBAL.group_replication_bootstrap_group=1;" \
  -e "create user 'repl'@'%';" \
  -e "GRANT REPLICATION SLAVE ON *.* TO repl@'%';" \
  -e "flush privileges;" \
  -e "change master to master_user='repl' for channel 'group_replication_recovery';" \
  -e "START GROUP_REPLICATION;" \
  -e "SET @@GLOBAL.group_replication_bootstrap_group=0;" \
  -e "SELECT * FROM performance_schema.replication_group_members;"
```

위 명령어를 통해 group replication에 사용할 user credential을 명시했고, group bootstrap을 진행했습니다.

```sh
for N in 2 3
do docker exec -it node$N mysql -uroot -p<root_passowrd> \
  -e "change master to master_user='repl' for channel 'group_replication_recovery';" \
  -e "START GROUP_REPLICATION;"
done
```

위 명령어를 통해 노드 2, 3을 그룹에 추가할 수 있습니다.

추가한 결과는 다음과 같이 확인할 수 있습니다.

```text
❯ docker exec -it node1 mysql -uroot -p<root_password> \
  -e "SELECT * FROM performance_schema.replication_group_members;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+----------------------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION | MEMBER_COMMUNICATION_STACK |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+----------------------------+
| group_replication_applier | 12173275-8b89-11ef-a6fa-0242c0a80002 | node1       |        3306 | ONLINE       | PRIMARY     | 8.0.32         | XCom                       |
| group_replication_applier | 12447f2a-8b89-11ef-963a-0242c0a80003 | node2       |        3306 | ONLINE       | SECONDARY   | 8.0.32         | XCom                       |
| group_replication_applier | 1275d0c9-8b89-11ef-aff6-0242c0a80004 | node3       |        3306 | ONLINE       | SECONDARY   | 8.0.32         | XCom                       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+----------------------------+
```

### 데이터 추가하기

다음 두 명령어를 이용해 테스트용 테이블을 생성하고, 데이터를 추가합니다.
```sh
docker exec -it node1 mysql -uroot -p<root_password> \
  -e "create database TEST; use TEST; CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY) ENGINE=InnoDB; show tables;"

for N in 1 2 3 4 5
do docker exec -it node1 mysql -uroot -p<root_password> \
  -e "INSERT INTO TEST.t1 VALUES($N);"
done
```

각 노드에 추가된 데이터를 확인할 수 있습니다.
```sh
for N in 1 2 3
do docker exec -it node$N mysql -uroot -prlaehdwns99 \
  -e "SHOW VARIABLES WHERE Variable_name = 'hostname';" \
  -e "SELECT * FROM TEST.t1;"
done
```

```text
❯ sh gr_show_data.sh
mysql: [Warning] Using a password on the command line interface can be insecure.
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| hostname      | node1 |
+---------------+-------+
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
|  4 |
|  5 |
+----+
mysql: [Warning] Using a password on the command line interface can be insecure.
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| hostname      | node2 |
+---------------+-------+
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
|  4 |
|  5 |
+----+
mysql: [Warning] Using a password on the command line interface can be insecure.
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| hostname      | node3 |
+---------------+-------+
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
|  4 |
|  5 |
+----+
```




