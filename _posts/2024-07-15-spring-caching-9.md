---
layout: post
title: Spring - Caching.9
date: 2024-07-15 15:05 +0900
description: ehcache에 대하여
image:
category: ["spring-cache", "project"]
tags: ["ehcache"]
published: true
sitemap: true
author: kim-dong-jun99
---

> 공식 문서를 참고하여 학습한 내용을 정리한 글입니다.      
> [https://www.ehcache.org](https://www.ehcache.org)

## ehcache

공식 문서에서는 ehcache를 다음과 같이 소개하고 있습니다.
> Ehcache is an open source, standards-based cache that boosts performance, offloads your database, and simplifies scalability. It's the most widely-used Java-based cache because it's robust, proven, full-featured, and integrates with other popular libraries and frameworks. Ehcache scales from in-process caching, all the way to mixed in-process/out-of-process deployments with terabyte-sized caches.

### 기본 설정

ehcache는 코드 혹은 xml 파일을 이용해서 설정할 수 있습니다. 

```java
CacheManager cacheManager = CacheManagerBuilder.newCacheManagerBuilder() 
    .withCache("preConfigured",
        CacheConfigurationBuilder.newCacheConfigurationBuilder(Long.class, String.class, ResourcePoolsBuilder.heap(10))) 
    .build(); 
cacheManager.init(); 

Cache<Long, String> preConfigured =
    cacheManager.getCache("preConfigured", Long.class, String.class); 

Cache<Long, String> myCache = cacheManager.createCache("myCache", 
    CacheConfigurationBuilder.newCacheConfigurationBuilder(Long.class, String.class, ResourcePoolsBuilder.heap(10)));

myCache.put(1L, "da one!"); 
String value = myCache.get(1L); 

cacheManager.removeCache("preConfigured"); 

cacheManager.close(); 
```

```xml
<config xmlns='http://www.ehcache.org/v3'>

  <cache alias="foo"> 
    <key-type>java.lang.String</key-type> 
    <value-type>java.lang.String</value-type> 
    <resources>
      <heap unit="entries">20</heap> 
      <offheap unit="MB">10</offheap> 
    </resources>
  </cache>

  <cache-template name="myDefaults"> 
    <key-type>java.lang.Long</key-type>
    <value-type>java.lang.String</value-type>
    <heap unit="entries">200</heap>
  </cache-template>

  <cache alias="bar" uses-template="myDefaults"> 
    <key-type>java.lang.Number</key-type>
  </cache>

  <cache alias="simpleCache" uses-template="myDefaults" /> 

</config>
```

### storage tiers

Ehcache는 다양한 데이터 저장 영역을 사용할 수 있도록 구성할 수 있습니다. 캐시가 둘 이상의 저장 영역을 사용하도록 구성되면 이러한 영역은 티어로 관리됩니다. 이들은 계층적 구조로 조직되며, 가장 낮은 계층(더 멀리 있는)은 권위 계층(authority tier)이라 불리며, 다른 계층은 캐싱 계층(가까운, near cache라고도 함)의 일부입니다. 가장 빈번하게 접근되는 데이터는 캐싱 계층에 보관되며, 이는 일반적으로 적지만 더 빠릅니다. 모든 데이터는 더 느리지만 더 풍부한 권위 계층에 저장됩니다.

Ehcache가 지원하는 데이터 저장소는 다음과 같습니다:

![https://www.ehcache.org/documentation/3.10/images/diag-cc830e8ffd9359d57922580a03ac369e.png](https://www.ehcache.org/documentation/3.10/images/diag-cc830e8ffd9359d57922580a03ac369e.png)

- 온힙 스토어 (On-Heap Store)
  - 특징: Java의 온힙 RAM 메모리를 활용하여 캐시 항목을 저장합니다.
  - 장점: 매우 빠르지만 가장 제한된 저장 자원입니다.
  - 단점: JVM 가비지 컬렉터가 동일한 힙 메모리를 스캔해야 하므로, 더 많은 힙 공간을 사용할수록 애플리케이션 성능에 가비지 컬렉션 일시 중지로 영향을 미칩니다.
- 오프힙 스토어 (Off-Heap Store)
  - 특징: 사용 가능한 RAM에 의해서만 크기가 제한됩니다. Java 가비지 컬렉션(GC)의 영향을 받지 않습니다.
  - 장점: 꽤 빠르지만 온힙 스토어보다는 느립니다. 데이터가 JVM 힙으로 이동되어 재접근할 때 속도가 느려집니다.
- 디스크 스토어 (Disk Store)
  - 특징: 디스크(파일 시스템)를 활용하여 캐시 항목을 저장합니다.
  - 장점: 매우 풍부한 저장 자원이지만 RAM 기반 스토어보다는 훨씬 느립니다.
  - 권장 사항: 디스크 저장소를 사용하는 모든 애플리케이션의 경우, 처리량을 최적화하기 위해 빠르고 전용 디스크를 사용하는 것이 좋습니다.
- 클러스터드 스토어 (Clustered Store)
  - 특징: 원격 서버에 있는 캐시입니다. 원격 서버는 고가용성을 제공하는 failover 서버를 선택적으로 가질 수 있습니다.
  - 단점: 네트워크 지연 및 클라이언트/서버 일관성을 유지하는 등의 요인으로 인해 성능 페널티가 발생하므로 이 계층은 본질적으로 로컬 오프힙 저장소보다 느립니다.



### 클터스터링

terracota 서버를 사용하면 클러스터링이 가능합니다. terracota 서버를 실행하고, 캐시 매니저 설정도 다음과 같이 변경해야합니다.
```java
CacheManagerBuilder<PersistentCacheManager> clusteredCacheManagerBuilder =
    CacheManagerBuilder.newCacheManagerBuilder() 
        .with(ClusteringServiceConfigurationBuilder.cluster(URI.create("terracotta://localhost/my-application")) 
            .autoCreateOnReconnect(c -> c)); 
PersistentCacheManager cacheManager = clusteredCacheManagerBuilder.build(true); 

cacheManager.close(); 
```

분산 캐싱은 로컬 온힙(온-힙) 계층이 제공하는 낮은 지연 시간을 유지하면서도 수평 확장의 추가적인 이점을 활용할 수 있게 해줍니다.

![https://www.ehcache.org/documentation/3.10/images/diag-023dc90b3469ccf84002e716bfe382de.png](https://www.ehcache.org/documentation/3.10/images/diag-023dc90b3469ccf84002e716bfe382de.png)

- 자주 사용되는 데이터는 더 빠른 티어에 저장됩니다.
- 한 애플리케이션에 캐시된 데이터도 다른 모든 클러스터 멤버가 접근 가능합니다.

**클러스터링 개념**

![https://www.ehcache.org/documentation/3.10/images/diag-7066c19a85be7f4b557b92d7002a3c17.png](https://www.ehcache.org/documentation/3.10/images/diag-7066c19a85be7f4b557b92d7002a3c17.png)

- 서버 오프힙 리소스 (Server Off-Heap Resources)
  - 서버 오프힙 리소스는 서버에 정의된 저장 리소스입니다. 캐시는 이 서버 오프힙 리소스 내에서 클러스터 계층을 위한 저장 영역을 예약할 수 있습니다.
- 클러스터 계층 관리자 (Cluster Tier Manager)
  - Ehcache 클러스터 계층 관리자는 캐시 관리자가 클러스터링 기능을 사용할 수 있도록 하는 서버 측 구성 요소입니다. 캐시 관리자는 서버의 저장 리소스에 접근하기 위해 클러스터 계층 관리자에 연결하여, 정의된 캐시의 클러스터 계층이 이러한 리소스를 사용할 수 있도록 합니다. 서버 측의 Ehcache 클러스터 계층 관리자는 고유 식별자로 식별됩니다. 주어진 클러스터 계층 관리자의 고유 식별자를 사용하여 여러 캐시 관리자가 동일한 클러스터 계층 관리자에 연결하여 캐시 데이터를 공유할 수 있습니다. 클러스터 계층 관리자는 캐시의 클러스터 계층 저장소를 관리하며, 다음과 같은 다양한 옵션을 제공합니다.
- 전용 풀 (Dedicated Pool)
  - 전용 풀은 캐시의 클러스터 계층에 할당된 고정된 양의 저장 풀입니다. 서버 오프힙 리소스에서 이 풀로 직접 할당된 전용 저장 공간을 사용합니다. 이 저장 공간은 특정 클러스터 계층에만 독점적으로 사용됩니다.
- 공유 풀 (Shared Pool)
  - 공유 풀 역시 고정된 양의 저장 풀로, 여러 캐시의 클러스터 계층이 공유할 수 있습니다. 전용 풀의 경우와 마찬가지로, 공유 풀은 서버 오프힙 리소스에서 할당됩니다. 이 공유 풀에서 사용 가능한 저장 공간은 엄격하게 공유됩니다. 즉, 어떤 클러스터 계층도 공유 풀에서 고정된 양의 저장 공간을 요구할 수 없습니다.
- 저장 공간의 공유
  - 저장 공간을 공유한다고 해서 데이터가 공유되는 것은 아닙니다. 즉, 두 개의 캐시가 클러스터 계층으로 공유 풀을 사용하는 경우에도 각 캐시의 데이터는 여전히 격리되어 있지만, 기본 저장 공간은 공유됩니다. 따라서 리소스 용량이 한계에 도달하여 제거가 트리거되면, 제거된 매핑은 풀을 공유하는 클러스터 계층 중 어느 것에서든 나올 수 있습니다.