---
layout: post
title: Spring - Caching.6
date: 2024-07-09 15:00 +0900
description: redisson을 이용한 jpa 2차 캐시 테스트
image:
category: ["spring-cache", "project"]
tags: ["redisson", "cache concurrency strategy", "hibernate", "redis"]
published: true
sitemap: true
author: kim-dong-jun99
---

## redisson & hibernate 2차 캐시 설정

현재 배포된 redis를 이용해 jpa 2차 캐시 동작 확인을 진행했습니다.      
공식 문서를 참고해가며 어렵지 않게 redis 2차 캐시를 구성할 수 있었습니다.

- [하이버네이트 캐싱 공식 문서](https://docs.jboss.org/hibernate/orm/6.1/userguide/html_single/Hibernate_User_Guide.html#caching)
- [redisson hibernate 공식 문서](https://github.com/redisson/redisson/tree/master/redisson-hibernate)

설정한 2차 캐시 관련 yml 내용은 다음과 같습니다.
```yaml
jpa:
    properties:
      jakarta:
        persistence:
          sharedCache:
            mode: ENABLE_SELECTIVE
      hibernate:
        dialect: org.hibernate.dialect.MySQLDialect
        hbm2ddl:
          auto: update
        format_sql: true
        show_sql: true
        cache:
          use_structured_entries: true
          use_second_level_cache: true
          use_query_cache: true
          use_minimal_puts: true
          region:
            factory_class: org.redisson.hibernate.RedissonRegionFactory
          redisson:
            config: redisson.yaml
            entity:
              eviction:
                max_entries: 3000
              expiration:
                time_to_live: 600000
              localcache: // redisson 유료 버전을 사용해야 적용가능합니다.
                time_to_live: 300000
                max_idle_time: 300000
                eviction_policy: LRU
                reconnection_strategy: CLEAR
                size: 1500
            collection:
              eviction:
                max_entries: 3000
              expiration:
                time_to_live: 600000
              localcache:
                time_to_live: 300000
                max_idle_time: 300000
                eviction_policy: LRU
                reconnection_strategy: CLEAR
                size: 1500
            query:
              eviction:
                max_entries: 3000
              expiration:
                time_to_live: 600000
              localcache:
                time_to_live: 300000
                max_idle_time: 300000
                eviction_policy: LRU
                reconnection_strategy: CLEAR
                size: 1500
            timestamps:
              eviction:
                max_entries: 3000
              expiration:
                time_to_live: 600000
              localcache:
                time_to_live: 300000
                max_idle_time: 300000
                eviction_policy: LRU
                reconnection_strategy: CLEAR
                size: 1500
          default_cache_concurrency_strategy: read-write
          generate_statistics: true
```


`src/main/resources/redisson.yaml` 파일은 다음 내용으로 구성했습니다.

```yaml
singleServerConfig:
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  password: [비밀번호]
  subscriptionsPerConnection: 5
  clientName: [계정]
  address: "redis://[배포 서버 ip]:6379"
  subscriptionConnectionMinimumIdleSize: 1
  subscriptionConnectionPoolSize: 50
  connectionMinimumIdleSize: 10
  connectionPoolSize: 64
  database: 0
  dnsMonitoringInterval: 5000
```


## jpa 2차 캐시 동작 확인 

### 간단한 조회 동작 확인

우선 먼저 jpa 2차 캐시 동작 확인을 위한 테스트용 엔티티, 서비스, 레포지토리 그리고 컨트롤러를 만들었습니다.

<img width="334" alt="Screenshot 2024-07-09 at 15 02 56" src="https://github.com/rastle-dev/rastle_backend/assets/95599193/fac400dc-7809-438e-a0ea-9b375edda155">

간단한 api를 개발해 멤버 생성, 전체 조회, id로 개별조회를 만들었습니다.

<img width="380" alt="Screenshot 2024-07-09 at 15 04 19" src="https://github.com/rastle-dev/rastle_backend/assets/95599193/03fa09fa-74e6-4a58-bdb0-eaf918618998">

`TestMember` 에는 `@Cache` 어노테이션을 추가해서 jpa 2차 캐시에 저장되도록하였습니다.

<img width="526" alt="Screenshot 2024-07-09 at 15 05 34" src="https://github.com/rastle-dev/rastle_backend/assets/95599193/4ac08df8-7552-47c0-bfc4-d6e115fa9493">

3명의 멤버를 생성한 다음, 전체 조회 api를 한번 실행하고, 개별 조회로 멤버들을 조회해봤습니다.       

<img width="1177" alt="Screenshot 2024-07-09 at 15 14 09" src="https://github.com/rastle-dev/rastle_backend/assets/95599193/e0f666ea-0f40-49f3-b167-c3d4d30b73da">

엔티티에 `@Cache` 어노테이션을 추가하였기에, 전체 조회를 할 때 멤버 엔티티가 2차 캐시에 저장된 것을 확인할 수 있습니다. 그리고 캐싱된 멤버들을 조회할 때, 데이버테이스 쿼리가 실행되지 않고 2차 캐시에서 멤버들을 조회하는 것을 확인할 수 있었습니다.

### 캐시 쓰기 작업 동작 확인

이제 멤버를 한명 생성하고, 해당 멤버 정보 업데이트를 한 후, 멤버를 다시 조회해보겠습니다.

<img width="1086" alt="Screenshot 2024-07-09 at 15 20 43" src="https://github.com/rastle-dev/rastle_backend/assets/95599193/b72196f7-eba7-41fc-86ae-2e48572f1bae">

테스트 과정에서 실행된 데이터베이스 쿼리입니다.    

<img width="175" alt="Screenshot 2024-07-09 at 15 25 31" src="https://github.com/rastle-dev/rastle_backend/assets/95599193/a9cb46ab-dc46-4d76-a85f-fe23b478f802">

멤버를 업데이트하고, 다시 개별조회할 때, 조회 쿼리가 실행되는 것을 확인할 수 있습니다. redis를 확인해보니, 멤버 업데이트 요청이 실행된 이후에 캐시에서 값이 삭제되는 것을 확인할 수 있었습니다.

jpa 2차 캐시가 위와 같은 방법으로 쓰기 작업을 처리한 것이 `CacheConcurrencyStrategy` 와 관련 있어보여 jpa 의 캐시 동시성 전략에 대해서 알아보았습니다.

## JPA CacheConcurrencyStrategy

> 참고 블로그 :          
>  [https://vladmihalcea.com/how-does-hibernate-read_only-cacheconcurrencystrategy-work/](https://vladmihalcea.com/how-does-hibernate-read_only-cacheconcurrencystrategy-work/)
> [https://vladmihalcea.com/how-does-hibernate-nonstrict_read_write-cacheconcurrencystrategy-work/](https://vladmihalcea.com/how-does-hibernate-nonstrict_read_write-cacheconcurrencystrategy-work/)
> [https://vladmihalcea.com/how-does-hibernate-read_write-cacheconcurrencystrategy-work/](https://vladmihalcea.com/how-does-hibernate-read_write-cacheconcurrencystrategy-work/)
> [https://vladmihalcea.com/how-does-hibernate-transactional-cacheconcurrencystrategy-work/](https://vladmihalcea.com/how-does-hibernate-transactional-cacheconcurrencystrategy-work/)

### READ_ONLY

불변인 객체에 사용하기 적합한 캐시 정책입니다. 캐시에서 값을 읽어올 수 만 있기 때문에, immutable한 객체를 캐시하는데 좋은 정책입니다.

### NONSTRICT_READ_WRITE 

캐시된 데이터가 변경 가능할 때, 캐시 읽기 뿐만 아니라 캐시 쓰기 전략을 고려해야할 때, 쓰기 적합한 정책입니다.        
내부 동작 방식은 다음과 같습니다.

![https://vladmihalcea.com/wp-content/uploads/2015/05/nonstrictreadwritecacheconcurrencystrategy4.png](https://vladmihalcea.com/wp-content/uploads/2015/05/nonstrictreadwritecacheconcurrencystrategy4.png)

먼저 데이터베이스 트랜잭션이 커밋되기전에, 캐시는 무효화되고 flush 되는 동안 다음 작업들이 진행됩니다.
1. 현재 하이버네이트 트랜잭션이 커밋됩니다. 
2. `DefaultFlushEventListener`가 `ActionQueue`를 실행합니다.
3. `EntityUpdateAction`이 `EntityRegionAccessStrategy`의 `update` 메소드를 호출합니다.
4. 실행된 `update` 메소드에 의해 캐시에 있는 값이 제거됩니다.

그리고 트랜잭션이 커밋된 이후에 위와 같은 과정을 한번 더 실행하여 한번 더 캐시에 있는 값을 지웁니다.

**동시성 문제**

`NONSTRICT_READ_WRITE`은 캐시 쓰기 작업이 발생했을 때, 캐시 값이 업데이트 되지 않고 무효화 처리 되기 때문에 `read through` 캐시이지 `write through` 캐시는 아닙니다. 캐시가 트랜잭션 커밋 이전, 이후 2번이나 무효화 처리되지만, 그래도 캐시와 데이터베이스 간 불일치가 존재하는 구간은 아주 짧은 시간이지만 분명하게 존재합니다.

캐시 무효화하는 과정에 lock을 획득하여 순차적으로 작업을 진행하지 않습니다. 캐시를 무효화하는 과정에서 데이터 조회 요청이 들어오게되면, 과거 버전으로 캐시에 데이터가 존재할 수 있습니다.
> 이 문제는 제가 개발한 `WriteThroughCache` 어노테이션과 같은 문제로 보입니다.

### READ_WRITE

`NONSTRICT_READ_WRITE`은 `read through` 를 읽기 전략으로 사용하고, 쓰기 작업에는 캐시 무효화를 이용해 처리했습니다. 효율적인 전략으로 보이지만, 쓰기 요청이 빈번하게 발생하는 환경에서 성능 저하를 불러올 수 있습니다.
> 쓰기 작업시 cache에서 값이 제거되기에 데이터베이스에 접근해서 값을 읽어와야합니다.

쓰기 작업이 빈번하게 일어나는 환경에서 `write through` 정책이 좀 더 적합한 쓰기 전략입니다. 

데이터 변경 작업이 일어났을 때 동작 방식은 다음과 같습니다.

![https://vladmihalcea.com/wp-content/uploads/2015/05/readwritecacheconcurrencystrategy_update4.png](https://vladmihalcea.com/wp-content/uploads/2015/05/readwritecacheconcurrencystrategy_update4.png)

1. 하이버네이트 트랜잭션 커밋이 `Session.flush`를 유발합니다.
2. `EntityUpdateAction`이 현재 캐시 엔트리를 `lock`오브젝트로 대체합니다.
3. synchronous cache concurrency strategy 인 경우 `update`가 호출됩니다.
4. 데이터베이스 트랜잭션이 커밋된 이후 `after-transaction-completed` 콜백이 호출됩니다.
5. `EntityUpdateAction`이 `afterUpdate` 메소드를 호출해 `lock` 오브젝트를 실제 아이템으로 대체합니다.

## query cache

jpa query cache를 이용하면 jpa repository를 이용해 조회한 결과를 caching 할 수 있습니다.      
jpa query cache 동작에서 확인하고 싶은 내용이 있어서 직접 동작 테스트를 해보았습니다.
> query cache로 조회한 엔티티에서 업데이트가 발생했을 때, cache에 있는 값이 올바르게 사라지는지 확인하는 테스트를 해보았습니다.

테스트해본 query cache 동작은 다음과 같습니다.

먼저 조회용 dto를 생성했습니다.           
<img width="184" alt="Screenshot 2024-07-09 at 17 08 38" src="https://github.com/rastle-dev/rastle_backend/assets/95599193/ad35fd9d-2ed6-4bc5-a1f2-8689e1423cc3">

그리고 dto로 조회하는 repository 코드를 다음과 같이 작성했습니다.           
<img width="712" alt="Screenshot 2024-07-09 at 17 09 33" src="https://github.com/rastle-dev/rastle_backend/assets/95599193/8189e6dd-6019-43c2-b07b-6c7ab4bdb3a1">

dto로 조회하는 api를 먼저 호출하고, 멤버 업데이트를 호출한 뒤 다시 dto 조회 api를 호출했을 때 실행되는 데이터베이스 쿼리입니다.        
<img width="197" alt="Screenshot 2024-07-09 at 17 10 08" src="https://github.com/rastle-dev/rastle_backend/assets/95599193/6e31ac6b-0375-43f5-8012-e64d7a1e400e">

cache에 있는 잘못된 값이 잘 invalidate 처리되는 것을 확인할 수 있었습니다.        
entity와 query 모두에 적당한 캐시 전략을 설정해두면, 문제없이 캐시를 활용할 수 있는 것이 확인되었습니다.

## querydsl + caching

하지만 querydsl의 경우, cache에서 데이터를 읽어오지 않고, 실행할 때마다 데이터베이스 쿼리가 실행되는 것을 확인할 수 있었습니다. 원인 파악을 위해 구글링 해본 결과,querydsl는 jpql 빌더이기에 영속성 컨텍스트에서 먼저 값을 찾는 것이 아니라, sql로 번역되어서 우선 실행된다는 내용을 알 수 있었습니다.

현재 전체 상품 조회, 인기 상품 조회 api는 querydsl을 이용해 동적 쿼리를 처리하도록 구현되어있습니다.      
querydsl에서 처리하는 동적 쿼리에도 caching이 적용되어야하기에,   문제 해결 과정을 이후 포스팅에서 다뤄볼 계획입니다.



