---
layout: post
title: jpa transaction manager
date: 2024-12-03 16:26 +0900
description: jpa transaction manager 동작 방식 정리
image: https://velog.velcdn.com/images/seung7152/post/ed954a47-3150-445b-9312-e3ecb6eb4443/image.png
category: ["jpa"]
tags:
published: true
sitemap: true
author: kim-dong-jun99
---

jpa 동작 방식 중 transaction manager 동작 방식에 관해서 정리한 글입니다.

우선 작성한 controller, service 코드는 다음과 같습니다.
```kotlin
    // controller
    @PostMapping("/signup")
    fun signUp(
        @Valid @RequestBody request: AuthDto.SignUpRequest,
    ): ResponseEntity<ApiResponse<*>> {
        log.info("sign up api")
        return ResponseEntity.status(HttpStatus.CREATED).body(ApiResponse.success(authService.signUp(request)))
    }

    // service
    @Transactional
    fun signUp(request: AuthDto.SignUpRequest): AuthDto.SignUpResponse {
        log.info("sign up service")
        val member = userRepository.save(mapper.signUpRequestToMember(request, passwordEncoder))
        return mapper.memberToSignUpResponse(member)
    }
```

동작 방식을 명확히 알기 위해 로그 수준을 debug으로 설정하고 api 실행시 발생한 로그 파일을 확인했습니다.

```text
2024-12-03 15:00:26.315KST [http-nio-8080-exec-1] DEBUG org.springframework.security.web.FilterChainProxy - Securing POST /api/auth/signup
2024-12-03 15:00:26.324KST [http-nio-8080-exec-1] DEBUG org.springframework.security.web.authentication.AnonymousAuthenticationFilter - Set SecurityContextHolder to anonymous SecurityContext
2024-12-03 15:00:26.327KST [http-nio-8080-exec-1] DEBUG org.springframework.security.web.FilterChainProxy - Secured POST /api/auth/signup
2024-12-03 15:00:26.328KST [http-nio-8080-exec-1] DEBUG org.springframework.web.servlet.DispatcherServlet - POST "/api/auth/signup", parameters={}
2024-12-03 15:00:26.329KST [http-nio-8080-exec-1] DEBUG org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping - Mapped to org.wpp.auth.AuthApi#signUp(SignUpRequest)
2024-12-03 15:00:26.330KST [http-nio-8080-exec-1] DEBUG org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor - Opening JPA EntityManager in OpenEntityManagerInViewInterceptor
2024-12-03 15:00:26.366KST [http-nio-8080-exec-1] DEBUG org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor - Read "application/json;charset=UTF-8" to [SignUpRequest(email=test@email.com, password=123445678!a, nickname=anonymousUser)]
2024-12-03 15:00:26.391KST [http-nio-8080-exec-1] INFO  org.wpp.auth.AuthApi - sign up api
2024-12-03 15:00:26.397KST [http-nio-8080-exec-1] DEBUG org.springframework.orm.jpa.JpaTransactionManager - Found thread-bound EntityManager [SessionImpl(130132814<open>)] for JPA transaction
2024-12-03 15:00:26.397KST [http-nio-8080-exec-1] DEBUG org.springframework.orm.jpa.JpaTransactionManager - Creating new transaction with name [org.wpp.auth.AuthService.signUp]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
2024-12-03 15:00:26.398KST [http-nio-8080-exec-1] DEBUG org.hibernate.engine.transaction.internal.TransactionImpl - On TransactionImpl creation, JpaCompliance#isJpaTransactionComplianceEnabled == false
2024-12-03 15:00:26.398KST [http-nio-8080-exec-1] DEBUG org.hibernate.engine.transaction.internal.TransactionImpl - begin
2024-12-03 15:00:26.409KST [http-nio-8080-exec-1] DEBUG org.springframework.orm.jpa.JpaTransactionManager - Exposing JPA transaction as JDBC [org.springframework.orm.jpa.vendor.HibernateJpaDialect$HibernateConnectionHandle@b29e8ec]
2024-12-03 15:00:26.410KST [http-nio-8080-exec-1] INFO  org.wpp.auth.AuthService - sign up service
2024-12-03 15:00:26.509KST [http-nio-8080-exec-1] DEBUG org.springframework.orm.jpa.JpaTransactionManager - Found thread-bound EntityManager [SessionImpl(130132814<open>)] for JPA transaction
2024-12-03 15:00:26.509KST [http-nio-8080-exec-1] DEBUG org.springframework.orm.jpa.JpaTransactionManager - Participating in existing transaction
2024-12-03 15:00:26.526KST [http-nio-8080-exec-1] DEBUG org.springframework.orm.jpa.JpaTransactionManager - Initiating transaction commit
2024-12-03 15:00:26.526KST [http-nio-8080-exec-1] DEBUG org.springframework.orm.jpa.JpaTransactionManager - Committing JPA transaction on EntityManager [SessionImpl(130132814<open>)]
2024-12-03 15:00:26.527KST [http-nio-8080-exec-1] DEBUG org.hibernate.engine.transaction.internal.TransactionImpl - committing
2024-12-03 15:00:26.527KST [http-nio-8080-exec-1] DEBUG org.hibernate.event.internal.AbstractFlushingEventListener - Processing flush-time cascades
2024-12-03 15:00:26.527KST [http-nio-8080-exec-1] DEBUG org.hibernate.event.internal.AbstractFlushingEventListener - Dirty checking collections
2024-12-03 15:00:26.530KST [http-nio-8080-exec-1] DEBUG org.hibernate.engine.internal.Collections - Collection found: [org.wpp.user.User.mutableCommentList#01938b1b-0e89-2f6a-05e7-bf73c82036ca], was: [<unreferenced>] (initialized)
2024-12-03 15:00:26.530KST [http-nio-8080-exec-1] DEBUG org.hibernate.engine.internal.Collections - Collection found: [org.wpp.user.User.mutableLikedItemsList#01938b1b-0e89-2f6a-05e7-bf73c82036ca], was: [<unreferenced>] (initialized)
2024-12-03 15:00:26.530KST [http-nio-8080-exec-1] DEBUG org.hibernate.engine.internal.Collections - Collection found: [org.wpp.user.User.mutableReviewList#01938b1b-0e89-2f6a-05e7-bf73c82036ca], was: [<unreferenced>] (initialized)
2024-12-03 15:00:26.531KST [http-nio-8080-exec-1] DEBUG org.hibernate.event.internal.AbstractFlushingEventListener - Flushed: 1 insertions, 0 updates, 0 deletions to 1 objects
2024-12-03 15:00:26.531KST [http-nio-8080-exec-1] DEBUG org.hibernate.event.internal.AbstractFlushingEventListener - Flushed: 3 (re)creations, 0 updates, 0 removals to 3 collections
2024-12-03 15:00:26.532KST [http-nio-8080-exec-1] DEBUG org.hibernate.internal.util.EntityPrinter - Listing entities:
2024-12-03 15:00:26.532KST [http-nio-8080-exec-1] DEBUG org.hibernate.internal.util.EntityPrinter - org.wpp.user.User{mutableCommentList=[], createdAt=2024-12-03T15:00:26.503705, password=$2a$10$ThCg5zWj6m2rBM6jRjP5QeXVdTtAlYNYvKcU4EKyGPTqP/xqw3F.G, authority=ROLE_USER, nickname=anonymousUser, mutableLikedItemsList=[], survey=component[accommodation,activeHours,food,planType,typeStyle]{planType=NOT_ANSWERED, activeHours=NOT_ANSWERED, accommodation=NOT_ANSWERED, typeStyle=NOT_ANSWERED, food=NOT_ANSWERED}, id=01938b1b-0e89-2f6a-05e7-bf73c82036ca, mutableReviewList=[], email=test@email.com}
2024-12-03 15:00:26.539KST [http-nio-8080-exec-1] DEBUG org.hibernate.SQL - 
    insert 
    into
        user
        (authority, created_at, email, nickname, password, accommodation, active_hours, food, plan_type, type_style, id) 
    values
        (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
Hibernate: 
    insert 
    into
        user
        (authority, created_at, email, nickname, password, accommodation, active_hours, food, plan_type, type_style, id) 
    values
        (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
2024-12-03 15:00:27.239KST [http-nio-8080-exec-1] DEBUG org.springframework.orm.jpa.JpaTransactionManager - Not closing pre-bound JPA EntityManager after transaction
2024-12-03 15:00:27.249KST [http-nio-8080-exec-1] DEBUG org.springframework.web.servlet.mvc.method.annotation.HttpEntityMethodProcessor - Using 'application/json', given [*/*] and supported [application/json, application/*+json]
2024-12-03 15:00:27.249KST [http-nio-8080-exec-1] DEBUG org.springframework.web.servlet.mvc.method.annotation.HttpEntityMethodProcessor - Writing [org.wpp.common.response.ApiResponse@79fe48e1]
2024-12-03 15:00:27.261KST [http-nio-8080-exec-1] DEBUG org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor - Closing JPA EntityManager in OpenEntityManagerInViewInterceptor
2024-12-03 15:00:27.262KST [http-nio-8080-exec-1] DEBUG org.springframework.web.servlet.DispatcherServlet - Completed 201 CREATED
```

발생한 로그 파일을 통해 동작 방식을 확인해보겠습니다.

```text
2024-12-03 15:00:26.315KST [http-nio-8080-exec-1] DEBUG org.springframework.security.web.FilterChainProxy - Securing POST /api/auth/signup
2024-12-03 15:00:26.324KST [http-nio-8080-exec-1] DEBUG org.springframework.security.web.authentication.AnonymousAuthenticationFilter - Set SecurityContextHolder to anonymous SecurityContext
2024-12-03 15:00:26.327KST [http-nio-8080-exec-1] DEBUG org.springframework.security.web.FilterChainProxy - Secured POST /api/auth/signup
2024-12-03 15:00:26.328KST [http-nio-8080-exec-1] DEBUG org.springframework.web.servlet.DispatcherServlet - POST "/api/auth/signup", parameters={}
2024-12-03 15:00:26.329KST [http-nio-8080-exec-1] DEBUG org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping - Mapped to org.wpp.auth.AuthApi#signUp(SignUpRequest)
```

가장 처음으로 security filter를 통과해 dispatcherServlet으로 요청이 도달한 것을 확인할 수 있습니다.<br/>
이후 `OpenEntityManagerInViewInterCeptor` 코드가 실행됩니다.
```text
2024-12-03 15:00:26.330KST [http-nio-8080-exec-1] DEBUG org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor - Opening JPA EntityManager in OpenEntityManagerInViewInterceptor

```
`OpenEntityManagerInViewInterceptor` 코드를 확인해보면, prehandle 코드가 실행되는 것을 확인할 수 있습니다.
```java
    public void preHandle(WebRequest request) throws DataAccessException {
        String key = this.getParticipateAttributeName();
        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
        if (!asyncManager.hasConcurrentResult() || !this.applyEntityManagerBindingInterceptor(asyncManager, key)) {
            EntityManagerFactory emf = this.obtainEntityManagerFactory();
            if (TransactionSynchronizationManager.hasResource(emf)) {
                Integer count = (Integer)request.getAttribute(key, 0);
                int newCount = count != null ? count + 1 : 1;
                request.setAttribute(this.getParticipateAttributeName(), newCount, 0);
            } else {
                this.logger.debug("Opening JPA EntityManager in OpenEntityManagerInViewInterceptor");

                try {
                    EntityManager em = this.createEntityManager();
                    EntityManagerHolder emHolder = new EntityManagerHolder(em);
                    TransactionSynchronizationManager.bindResource(emf, emHolder);
                    AsyncRequestInterceptor interceptor = new AsyncRequestInterceptor(emf, emHolder);
                    asyncManager.registerCallableInterceptor(key, interceptor);
                    asyncManager.registerDeferredResultInterceptor(key, interceptor);
                } catch (PersistenceException ex) {
                    throw new DataAccessResourceFailureException("Could not create JPA EntityManager", ex);
                }
            }

        }
    }
```

`preHandle` 코드를 살펴보면, `WebAsyncUtils`에서 `WebAsyncManager`를 현재 요청에대한 `AsyncManager`를 생성합니다.

```java
    public static WebAsyncManager getAsyncManager(WebRequest webRequest) {
        int scope = 0;
        WebAsyncManager asyncManager = null;
        Object asyncManagerAttr = webRequest.getAttribute(WEB_ASYNC_MANAGER_ATTRIBUTE, scope);
        if (asyncManagerAttr instanceof WebAsyncManager wam) {
            asyncManager = wam;
        }

        if (asyncManager == null) {
            asyncManager = new WebAsyncManager();
            webRequest.setAttribute(WEB_ASYNC_MANAGER_ATTRIBUTE, asyncManager, scope);
        }

        return asyncManager;
    }
```

`asyncManager`가 존재하면 존재하는 매니저를 리턴, 없다면 새로 생성해서 리턴하는 것을 확인할 수 있습니다.
그리고 `!asyncManager.hasConcurrentResult() || !this.applyEntityManagerBindingInterceptor(asyncManager, key)` 이 조건은 request에 대한 리턴 값이 비동기 값인지 확인하는 코드입니다.

리턴 값이 비동기 값이 아닌 경우 요청에 대한 EntityManager가 생성되는 것을 확인할 수 있습니다.

```java

                this.logger.debug("Opening JPA EntityManager in OpenEntityManagerInViewInterceptor");

                try {
                    EntityManager em = this.createEntityManager();
                    EntityManagerHolder emHolder = new EntityManagerHolder(em);
                    TransactionSynchronizationManager.bindResource(emf, emHolder);
                    AsyncRequestInterceptor interceptor = new AsyncRequestInterceptor(emf, emHolder);
                    asyncManager.registerCallableInterceptor(key, interceptor);
                    asyncManager.registerDeferredResultInterceptor(key, interceptor);
                } catch (PersistenceException ex) {
                    throw new DataAccessResourceFailureException("Could not create JPA EntityManager", ex);
                }
```

위 코드가 실행되어 엔티티 매니저가 생성되고, `TransactionSynchronizationManager`에 생성한 엔티티 매니저를 등록합니다.

이후 `@Transactional` 어노테이션이 부착된 서비스 메소드를 실행합니다.<br/>
`@Transactional` 이 부착된 코드는 내부적으로 다음 코드로 감싸진 후 실행됩니다.

```java
public Object invoke(MethodInvoation invoation) throws Throwable {
	TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
	
	try {
		Object ret = invoation.proceed();
		this.transactionManager.commit(status);
		return ret;
	} catch (Exception e) {
		this.transactionManager.rollback(status);
		throw e;
	}
}
```
`transactionManager.getTransaction`을 호출해 `transaction`을 받아오고, `invocation.proceed()`로 비즈니스 로직을 처리한 후, 문제가 없으면 커밋, 문제가 발생하면 롤백하게 됩니다.<br/>
spring은 트랜잭션 매니저로 `PlatformTransactionManger` 인터페이스를 사용하고, jpa를 사용할 경우, `JpaTransactionManager`를 구현체로 사용하게 됩니다.

`JpaTransactionManager`의 `getTransaction()` 코드를 호출하면 상위 추상 클래스인 `AbstractPlatformManager`의 `getTransaction` 메소드가 실행됩니다.

```java

    public final TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException {
        TransactionDefinition def = definition != null ? definition : TransactionDefinition.withDefaults();
        Object transaction = this.doGetTransaction();
        boolean debugEnabled = this.logger.isDebugEnabled();
        if (this.isExistingTransaction(transaction)) {
            return this.handleExistingTransaction(def, transaction, debugEnabled);
        } else if (def.getTimeout() < -1) {
            throw new InvalidTimeoutException("Invalid transaction timeout", def.getTimeout());
        } else if (def.getPropagationBehavior() == 2) {
            throw new IllegalTransactionStateException("No existing transaction found for transaction marked with propagation 'mandatory'");
        } else if (def.getPropagationBehavior() != 0 && def.getPropagationBehavior() != 3 && def.getPropagationBehavior() != 6) {
            if (def.getIsolationLevel() != -1 && this.logger.isWarnEnabled()) {
                this.logger.warn("Custom isolation level specified but no actual transaction initiated; isolation level will effectively be ignored: " + def);
            }

            boolean newSynchronization = this.getTransactionSynchronization() == 0;
            return this.prepareTransactionStatus(def, (Object)null, true, newSynchronization, debugEnabled, (Object)null);
        } else {
            SuspendedResourcesHolder suspendedResources = this.suspend((Object)null);
            if (debugEnabled) {
                Log var10000 = this.logger;
                String var10001 = def.getName();
                var10000.debug("Creating new transaction with name [" + var10001 + "]: " + def);
            }

            try {
                return this.startTransaction(def, transaction, false, debugEnabled, suspendedResources);
            } catch (Error | RuntimeException ex) {
                this.resume((Object)null, suspendedResources);
                throw ex;
            }
        }
    }
```

코드를 확인해보면, 먼저 `doGetTransaction`을 호출해 트랜잭션 오브젝트를 획득합니다.<br/>
`doGetTransaction`은 `JpaTransactionManager`의 구현 메소드로 실행됩니다.

```java
    protected Object doGetTransaction() {
        JpaTransactionObject txObject = new JpaTransactionObject();
        txObject.setSavepointAllowed(this.isNestedTransactionAllowed());
        EntityManagerHolder emHolder = (EntityManagerHolder)TransactionSynchronizationManager.getResource(this.obtainEntityManagerFactory());
        if (emHolder != null) {
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Found thread-bound EntityManager [" + emHolder.getEntityManager() + "] for JPA transaction");
            }

            txObject.setEntityManagerHolder(emHolder, false);
        }

        if (this.getDataSource() != null) {
            ConnectionHolder conHolder = (ConnectionHolder)TransactionSynchronizationManager.getResource(this.getDataSource());
            txObject.setConnectionHolder(conHolder);
        }

        return txObject;
    }
```

`TransactionSynchronizationManager.getResource`에서는 앞서 생성한 엔티티 매니저가 리턴되고, 트랜잭션 오브젝트에 해당 엔티티 매니저를 세팅한 다음 트랜잭션 오브젝트를 리턴합니다.

생성된 트랜잭션 오브젝트는 `JpaTransactionManager.isExistingTransaction()` 메소드로 새로 생성된 트랜잭션 인지 아닌지 확인합니다. `isExistingTransaction()` 메소드는 `JpaTransactionObject.hasTransaction()` 메소드의 호출 결과를 리턴합니다.

```java

        public boolean hasTransaction() {
            return this.entityManagerHolder != null && this.entityManagerHolder.isTransactionActive();
        }
```

트랜잭션 오브젝트를 생성할 때 세팅한 엔티티 매니저의 `isTransactionActive()` 값으로 새로운 트랜잭션인지 확인하는데, 이 값은 기본적으로 false로 생성되어 있어, `AbstractPlatformTransacionManager.startTransaction()`를 실행합니다.

```java


    private TransactionStatus startTransaction(TransactionDefinition definition, Object transaction, boolean nested, boolean debugEnabled, @Nullable SuspendedResourcesHolder suspendedResources) {
        boolean newSynchronization = this.getTransactionSynchronization() != 2;
        DefaultTransactionStatus status = this.newTransactionStatus(definition, transaction, true, newSynchronization, nested, debugEnabled, suspendedResources);
        this.transactionExecutionListeners.forEach((listener) -> listener.beforeBegin(status));

        try {
            this.doBegin(transaction, definition);
        } catch (Error | RuntimeException ex) {
            this.transactionExecutionListeners.forEach((listener) -> listener.afterBegin(status, ex));
            throw ex;
        }

        this.prepareSynchronization(status, definition);
        this.transactionExecutionListeners.forEach((listener) -> listener.afterBegin(status, (Throwable)null));
        return status;
    }
```

이후 `JpaTransactionManager.doBegin()`이 호출돼 새로운 트랜잭션을 시작합니다.<br/>
트랜잭션이 생성된 이후, `userRepository.save()`를 호출해 트랜잭션을 받아갈 때, 기존에 생성된 트랜잭션에 참여해서 실행되는 것을 확인할 수 있습니다.
```text
2024-12-03 17:38:41.694KST [http-nio-8080-exec-2] INFO  org.wpp.auth.AuthService - sign up service
2024-12-03 17:38:41.786KST [http-nio-8080-exec-2] DEBUG org.springframework.orm.jpa.JpaTransactionManager - Found thread-bound EntityManager [SessionImpl(449934990<open>)] for JPA transaction
2024-12-03 17:38:41.786KST [http-nio-8080-exec-2] DEBUG org.springframework.orm.jpa.JpaTransactionManager - Participating in existing transaction
```

