---
layout: post
title: Spring Caching.1
date: 2024-07-03 13:01 +0900
description: Cacheable 어노테이션과 Redis를 이용한 데이터 caching 처리
image:
category:["spring", "redis"]
tags:["caching"]
published: true
sitemap: true
author: kim-dong-jun99
---

> 조회 요청 응답 속도를 향상시키기 위해 Redis caching 을 적용하며 발생한 시행착오들을 정리한 글입니다.

# 도입 배경

쇼핑몰 프로젝트를 진행하면서 상품 조회 요청에 대한 응답 속도를 높이고 디비 부하를 줄이기 위한 목적으로 caching 을 도입했습니다.

cache 데이터 저장소로는 스프링 내부에 데이터를 저장할 수 있는 ehcache와 redis를 고민하다 redis로 결정했습니다. redis를 cache 저장소로 결정한 이유는 ehcache는 스프링 로컬 cache인데, 프로젝트에서는 3개의 스프링 서버를 이용하고 있습니다. 1개의 서버에서 데이터를 수정하면 다른 2 서버 cache에 데이터 변경사항을 전달하기 위해서는 kafka 같은 메세지 큐를 사용해야합니다. caching 도입을 시작한 시점에서 쇼핑몰 오픈이 얼마 남지 않은 시점이었기에 상대적으로 구성이 편한 redis를 이용하는 것으로 결정했습니다. 

> 프로젝트 마무리 이후 ehcache와 kafka를 이용한 cache 처리도 구현해볼 계획입니다.

# 도입 과정

> Spring boot 3.0.2
>
> aws ubuntu 22.04

aws ubuntu ec2 인스턴스를 생성한 후 docker 를 이용해서 redis 서버를 실행합니다.

```docker
docker run -v ./redis:/usr/local/etc/redis \
	-p 6379:6379 \
	-m="700m" --memory-swap="1g" \
	--name redis -d redis:latest \
  redis-server /usr/local/etc/redis/redis.conf
```

redis 설정 파일은 다음과 같습니다.
```
bind 0.0.0.0 # 외부 아이피 접근 허용을 위한 설정입니다

port 6379

protected-mode yes # 외부 접근시 비밀번호를 요구하는 설정입니다. 
requirepass [레디스 비밀번호] # 기본 비밀번호 설정입니다.
user [생성할 레디스유저] on >[비밀번호] ~* resetchannels +@all # spring에서 redis 접근시 사용할 유저를 생성하며 권한을 부여합니다.
```

build.gradle에 다음 의존성을 추가합니다.
```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
implementation 'org.springframework.boot:spring-boot-starter-cache'
```

redis template, 그리고 redis config는 다음과 같이 작성했습니다.
```java
import org.springframework.data.redis.core.RedisTemplate;

public class StringRedisTemplate extends RedisTemplate<String, String> {

}
```

```java
import org.springframework.data.redis.core.RedisTemplate;
import rastle.dev.rastle_backend.global.common.constants.CacheConstant;

import java.util.concurrent.TimeUnit;

public class ObjectRedisTemplate extends RedisTemplate<String, Object> {

    public void save(String key, Object object) {
        super.opsForValue().set(key, object, CacheConstant.EXPIRE_TIME, TimeUnit.MINUTES);
    }
}
```

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cache.CacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.RedisStandaloneConfiguration;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import rastle.dev.rastle_backend.global.cache.ObjectRedisTemplate;
import rastle.dev.rastle_backend.global.cache.StringRedisTemplate;
import rastle.dev.rastle_backend.global.common.constants.CacheConstant;

import java.time.Duration;

@Configuration
public class RedisConfig {
    @Value("${spring.data.redis.host}")
    private String host;

    @Value("${spring.data.redis.port}")
    private int port;

    @Value("${spring.data.redis.username}")
    private String username;

    @Value("${spring.data.redis.password}")
    private String password;

    @Value("${cache_host}")
    private String cacheHost;

    @Value("${cache_user}")
    private String cacheUser;

    @Value("${cache_password}")
    private String cachePassword;


    @Bean(name = "redisConnectionFactory") // jwt refresh 그리고 외부 api 엑세스 토큰을 저장하기 위한 레디스 커넥션 팩토리
    public RedisConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName(host);
        redisStandaloneConfiguration.setPort(port);
        redisStandaloneConfiguration.setUsername(username);
        redisStandaloneConfiguration.setPassword(password);
        return new LettuceConnectionFactory(redisStandaloneConfiguration);
    }

    @Bean(name = "cacheRedisConnectionFactory") // caching을 위해 사용할 레디스 커넥션 팩토리
    public RedisConnectionFactory cacheRedisConnectionFactory() {
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName(cacheHost);
        redisStandaloneConfiguration.setPort(port);
        redisStandaloneConfiguration.setUsername(cacheUser);
        redisStandaloneConfiguration.setPassword(cachePassword);
        return new LettuceConnectionFactory(redisStandaloneConfiguration);
    }

    @Bean
    public StringRedisTemplate redisTemplate() {
        StringRedisTemplate stringRedisTemplate = new StringRedisTemplate();
        stringRedisTemplate.setConnectionFactory(redisConnectionFactory());
        stringRedisTemplate.setDefaultSerializer(new StringRedisSerializer()); // key, value 모두 string을 사용할 것이기에 string serializer를 사용했습니다.
        return stringRedisTemplate;
    }

    @Bean
    public ObjectRedisTemplate cacheRedisTemplate() {
        ObjectRedisTemplate objectRedisTemplate = new ObjectRedisTemplate();
        objectRedisTemplate.setConnectionFactory(cacheRedisConnectionFactory());
        objectRedisTemplate.setKeySerializer(new StringRedisSerializer()); // string key를 사용할 것이기에 string serializer 지정
        objectRedisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer()); // value 로는 오브젝트를 저장할 것이기에 objectserializer 지정
        return objectRedisTemplate;
    }

    @Bean
    public CacheManager cacheManager() {
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
            .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))
            .entryTtl(Duration.ofMinutes(CacheConstant.EXPIRE_TIME)); // 캐쉬 저장 시간 15분 설정

        return RedisCacheManager
            .RedisCacheManagerBuilder
            .fromConnectionFactory(cacheRedisConnectionFactory()) // 위에서 생성한 cache connection factory 사용
            .cacheDefaults(redisCacheConfiguration) // cache serializer, 지속 시간 설정
            .build();
    }
}
```

위와 같이 cache 설정을 한 후에 다음과 같이 서비스 메소드에서 사용 가능합니다.
```java
@Service
@RequiredArgsConstructor
public class ProductService {
    private final ProductBaseRepository productBaseRepository;
    private final BundleProductRepository bundleProductRepository;
    private final EventProductRepository eventProductRepository;
    private final EventRepository eventRepository;
    private final ProductQRepository productQRepository;

    @Cacheable(cacheNames = GET_PRODUCTS, cacheManager = "cacheManager")
    @Transactional(readOnly = true)
    public SimpleProductQueryResult getProductInfos(GetProductRequest getProductRequest) {

        return productQRepository.getProductInfos(getProductRequest);
    }


    @Cacheable(cacheNames = GET_PRODUCT_DETAIL, cacheManager = "cacheManager")
    @Transactional(readOnly = true)
    public Object getProductDetail(Long id) throws JsonProcessingException {
        ProductInfo productInfo = productBaseRepository.getProductDetailInfoById(id).orElseThrow(NotFoundByIdException::new);
        if (productInfo.getEventId() != null) {
            Event event = eventRepository.findById(productInfo.getEventId()).orElseThrow(() -> new RuntimeException("이벤트 아이디로 존재하는 이벤트가 없음"));
            return EventProductDetailInfo.fromProductInfo(productInfo, event);
        }
        return productInfo;
    }

    @Cacheable(cacheNames = GET_BUNDLE_PRODUCTS, cacheManager = "cacheManager")
    @Transactional(readOnly = true)
    public List<BundleProductInfo> getBundleProducts(String visible, Long lowerBound, Long upperBound) {
        if (visible.equals(ALL)) {
            return bundleProductRepository.getBundleProducts(lowerBound, upperBound);
        } else if (visible.equals(TRUE)) {
            return bundleProductRepository.getBundleProductsByVisibility(true, lowerBound, upperBound);
        } else {
            return bundleProductRepository.getBundleProductsByVisibility(false, lowerBound, upperBound);
        }
    }

    @Cacheable(cacheNames = GET_EVENT_PRODUCTS, cacheManager = "cacheManager")
    @Transactional(readOnly = true)
    public EventProductQueryResult getEventProducts(String visible, Pageable pageable) {
        if (visible.equals(ALL)) {
            return new EventProductQueryResult(eventProductRepository.getEventProducts(pageable).stream().map(EventProductInfo::toOutInfo).toList());
        } else if (visible.equals(TRUE)) {
            return new EventProductQueryResult(eventProductRepository.getEventProductByVisibility(true, pageable).stream().map(EventProductInfo::toOutInfo).toList());
        } else {
            return new EventProductQueryResult(eventProductRepository.getEventProductByVisibility(false, pageable).stream().map(EventProductInfo::toOutInfo).toList());
        }

    }

    @Transactional(readOnly = true)
    public Page<SimpleProductInfo> getPopularProducts(String visible, Pageable pageable) {
        if (visible.equals(ALL)) {
            return productBaseRepository.getPopularProductInfos(pageable);
        } else if (visible.equals(TRUE)) {
            return productBaseRepository.getPopularProductInfosByVisibility(true, pageable);
        } else {
            return productBaseRepository.getPopularProductInfosByVisibility(false, pageable);

        }
    }
}
```

caching을 적용할 메소드에 @Cacheable 어노테이션을 작성하는 것으로 메소드 리턴 결과를 caching 할 수 있습니다.

서비스 메소드 리턴 결과를 redis에 저장할 때 주의 사항이 있습니다.

## LocalDateTime 자료형 저장 불가

LocalDateTime 자료형을 redis에 저장이 불가능하다는 오류를 맞이하였습니다. 
```java
@Bean(name = "redisObjectMapper")
public ObjectMapper redisObjectMapper() {

  ObjectMapper objectMapper = new ObjectMapper();
  objectMapper.registerModule(new JavaTimeModule());
  return objectMapper;
}
```
objectMapper에 LocalDateTime을 serialize, deserialize 할 수 있게 해주는 모듈을 추가해주는 것으로 해결 가능했지만, 위 모듈을 추가해주면, LocalDateTime이 기존에 출력되던 String 형태가 아니라 String[] 형태로 리턴됩니다. 

LocalDateTime 자료형을 계속 String 형태로 리턴해주기 위해서 LocalDateTime을 String으로 저장하는 클래스를 만들어서 redis에 저장하는 방식으로 문제를 해결했습니다.

## Page 저장 불가

페이징 처리된 조회 결과 역시 redis에 저장이 불가했습니다. 이 문제도 페이지 내부에 있는 dto 리스트와 totalElement를 저장하는 클래스를 하나 만들고 해당 클래스로 redis에 저장, controller 단에서는 service 단에서 리턴한 클래스로 부터 페이지 결과를 반환하는 방식으로 문제를 해결했습니다.

전체 프로젝트 코드는 아래 깃허브 링크에서 확인 가능합니다.

[https://github.com/rastle-dev/rastle_backend](https://github.com/rastle-dev/rastle_backend)
