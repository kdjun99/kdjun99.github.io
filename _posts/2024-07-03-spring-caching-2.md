---
layout: post
title: Spring Caching.2
date: 2024-07-03 15:10 +0900
description: Cacheable 어노테이션과 Redis를 이용한 데이터 caching 처리
image:
category: ["spring-cache", "project"]
tags: ["redis"]
published: true
sitemap: true
author: kim-dong-jun99
---

[지난 포스팅](https://kim-dong-jun99.github.io/posts/2024-07-03-spring-caching-1) 에서 Cacheable 어노테이션을 이용해 redis cache를 도입해보았습니다. 

여기에 추가적인 설정을 더 해줘야 cache에서 조회한 결과와 데이터베이스에서 조회한 결과가 동일하게 유지될 수 있습니다.

## cache 쓰기 전략

지난 포스팅에서 개발한 어노테이션 만으로는 여러가지 문제 상황이 발생할 수 있습니다. 

**발생할 수 있는 문제상황**

- 사용자가 쇼핑몰 홈페이지에 접근해서 여러가지 상품들을 둘러보고있습니다.
  - 상품 조회 api로 요청이 올 것이고, 상품 서비스는 조회 요청에 대한 결과는 redis에 caching 할 것입니다.
- 쇼핑몰 관리자가 주력 상품이 맨 위에 올 수 있게 상품의 `displayOrder` 속성 값을 변경합니다.
- 관리자가 `displayOrder` 값을 변경했지만, 사용자들은 변경된 순서가 아닌 기존 순서대로 정렬된 상품들을 보게 됩니다.

이런 문제 상황이 발생하게 되는 이유는 데이터 쓰기 작업이 발생했을 때 작업 변경 사항이 cache에는 미적용 되었기 때문이다.

문제 상황을 해결하기 위해 cache 해놓은 데이터에 변경사항을 만드는 메소드가 호출됐을 때, cache 값에도 변경 사항을 적용하는 spring aop 어노테이션을 개발했습니다.

개발한 어노테이션의 동작 방식은 다음과 같습니다.
1. 메소드에 전달된 파라미터로 부터 상품 아이디 값을 확인합니다.
  - 빈 상품 아이디 값으로 전달된 경우 새로 상품이 생성된 경우로 판단합니다.
2. redis에 저장된 key 값으로부터 cache 값을 등록해놓은 메소드, 조회 조건을 추출합니다.
3. 추출한 조회 조건에 맞게 데이터베이스에서 부터 값을 조회한 후, 키값에 조회한 결과를 저장합니다.

## 개발 과정

개발한 어노테이션은 다음과 같습니다.
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface WriteThroughCache {
    String identifier() default "id"; // id 값 추출을 위한 키 값

    Class<?> paramClassType() default Object.class; // 파라미터 클래스 타입이 Object.class가 아닌 경우에는 상품이 생성된 경우입니다.

    boolean singleUpdate() default false; // 상품의 상세 이미지 같은 상세 페이지에서 조회되는 값만 변경된 경우에는 true로 넘깁니다.
}
```


cacheable 어노테이션은 다음과 같이 keyGenerator를 추가했고, WriteThroughCache 어노테이션은 아래와 같이 사용 가능합니다.
```java
    // productService 내부 메소드
    @Cacheable(cacheNames = GET_PRODUCTS, cacheManager = "cacheManager", keyGenerator = "cacheKeyGenerator")
    @Transactional(readOnly = true)
    public SimpleProductQueryResult getProductInfos(GetProductRequest getProductRequest) {

        return productQRepository.getProductInfos(getProductRequest);
    }
    
    // adminService 내부 메소드
     @WriteThroughCache
    @Transactional
    public ProductImageInfo uploadMainThumbnail(Long id, MultipartFile mainThumbnail) {
        String mainThumbnailUrl = s3Component.uploadSingleImageToS3(MAIN_THUMBNAIL, mainThumbnail);
        productBaseRepository.updateProductBaseMainThumbnail(id, mainThumbnailUrl);
        return ProductImageInfo.builder()
            .productBaseId(id)
            .imageUrls(List.of(mainThumbnailUrl))
            .build();
    }
```


기존에 사용하던 기본 keyGenerator를 사용하지 않고 새로운 keyGenerator를 생성한 이유는 기존에 사용하던 기본 keyGenerator를 이용해서 생성된 key 값을 이용해서 원본 파라미터 데이터를 추출하는데 어려움이 있었습니다.

생성한 key 값으로부터 조회 조건을 추출할 수 있는 `CacheKeyGenerator` 라는 컴포넌트를 생성해서 사용했습니다. `CacheKeyGenerator` 코드는 아래와 같습니다.

```java
@Slf4j
@Component
@RequiredArgsConstructor
public class CacheKeyGenerator implements KeyGenerator {
    private final ObjectMapper objectMapper;
    @Override
    public Object generate(Object target, Method method, Object... params) {
        switch (method.getName()) {
            case MethodConstants.GET_EVENT_PRODUCTS -> {
                // productService String visible, Pageable pageable
                String visible = (String) params[0];
                CustomPageRequest pageable = (CustomPageRequest) params[1];
                try {
                    return visible + " " + objectMapper.writeValueAsString(pageable);
                } catch (JsonProcessingException e) {
                    throw new RuntimeException(e);
                }
            }
            case MethodConstants.GET_PRODUCTS_OF_BUNDLE -> {
                // bundleService Long bundleId
                return (Long) params[0];
            }
            case MethodConstants.GET_PRODUCT_DETAIL -> {
                // productService Long id
                return (Long) params[0];
            }
            case MethodConstants.GET_PRODUCT_INFOS -> {
                // productService GetProductRequest getProductRequest
                GetProductRequest getProductRequest = (GetProductRequest) params[0];
                try {
                    return objectMapper.writeValueAsString(getProductRequest);
                } catch (JsonProcessingException e) {
                    throw new RuntimeException(e);
                }
            }default -> {
                log.warn("can't create redis cache key {} {}", target.getClass().getSimpleName(), method.getName());
                return "unknown key";
            }
        }
    }
}
```


`WriteThroughCache`어노테이션을 처리하는 코드입니다.

```java
    @Around("@annotation(rastle.dev.rastle_backend.global.common.annotation.WriteThroughCache)")
    public Object writeThroughCache(ProceedingJoinPoint joinPoint) throws Throwable {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        WriteThroughCache writeThroughCache = method.getAnnotation(WriteThroughCache.class);

        Object proceed = joinPoint.proceed(joinPoint.getArgs());

        asyncCacheComponent.writeThroughCache(writeThroughCache, joinPoint.getArgs(), signature.getParameterNames()); // cache 쓰기 작업을 진행할 컴포넌트 메소드에 어노테이션, 메소드 파라미터, 메소드 파라미터 명 전달 
        return proceed;
    }
```


실제 cache에 쓰기 작업을 하는 `asyncCacheComponent.writeThroughCache` 의 코드는 다음과 같습니다.

```java
    @Transactional
    @Async("cacheTaskExecutor")
    public void writeThroughCache(WriteThroughCache writeThroughCache, Object[] args, String[] parameterNames) {
        if (isUpdateProductRequest(writeThroughCache)) {
            Long id = getId(parameterNames, args, writeThroughCache.identifier());
            updateProductDetail(id);
        }
        updateProductByBundle();
        updateGetProducts();
        updateGetEventProducts();
    }
```

cache 업데이트 하는 메소드들 중 일부의 코드입니다.
```java
    private void updateGetProducts() {
        String cacheKey = CacheConstant.GET_PRODUCTS;
        Set<String> keys = getRedisKeySet(cacheKey);
        for (String key : keys) {
            String value = parseKey(cacheKey, key);
            try {
                GetProductRequest getProductRequest = objectMapper.readValue(value, GetProductRequest.class);
                objectRedisTemplate.save(key, productQRepository.getProductInfos(getProductRequest));
            } catch (JsonProcessingException e) {
                log.info(e.getMessage());
                log.info("error during parsing json object of GetProductRequest");
            }
        }
    }
    
    private Set<String> getRedisKeySet(String key) {
        Set<String> set = objectRedisTemplate.keys(key + "::*");
        if (set == null) {
            return Set.of();
        }
        return set;
    }

    private String parseKey(String key, String toParse) {
        return toParse.substring(key.length() + 2);
    }
```

지난 포스팅에서 개발한 caching에 적합한 caching 전략이 갖춰지지 않아 데이터 변경 이벤트가 발생했을 때, cache에 변경된 데이터를 저장하는 어노테이션을 개발해보았습니다.

개발한 어노테이션은 잘 동작하는 것으로 보였으나 여전히 몇몇 문제점을 가지고 있었습니다. 

개발한 캐싱 전략의 문제점 분석과 그에 대한 해결방안에 대해서는 [다음 포스팅](https://kim-dong-jun99.github.io/posts/spring-caching-3/)에서 확인하실 수 있습니다.

[프로젝트 전체 소스 코드](https://github.com/rastle-dev/rastle_backend)

