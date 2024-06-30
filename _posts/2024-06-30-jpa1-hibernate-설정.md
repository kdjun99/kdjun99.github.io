---
layout: post
title: JPA1. Hibernate 설정
date: 2024-06-30 20:21 +0900
description:
image:
category: ["jpa"]
tags: ["jpa"]
published: true
sitemap: true
author: kim-dong-jun99
---
> spring 과 함께 jpa를 사용하는 것이 아닌 순수 jpa를 hibernate를 이용해서 이용할 때 설정하는 방법입니다.

# 1. Hibernate 설정

JPA는 `persistence.xml`을 사용해서 필요한 설정 정보를 관리한다. 다음 내용은 설정 파일의 예시이다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">
  <!--영속성 유닛 설정, 연결할 데이터베이스 하나당 하나의 영속성 유닛을 사용  -->
  <persistence-unit name="jpabook">
    <properties>
      <!-- JDBC 드라이버 -->
      <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
      <!-- 데이터베이스 접속 아이디-->
      <property name="javax.persistence.jdbc.user" value="sa" />
      <!-- 데이터베이스 접속 비밀번호 -->
      <property name="javax.persistence.jdbc.password" value=""/>
      <!-- 데이터베이스 접속 url -->
      <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
      <!-- 데이터베이스 방언 설정 -->
      <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />
    </properties>
  </persistence-unit>
</persistence>

```

## 데이터베이스 방언

JPA는 데이터베이스에 종속적이지 않은 기술이다. 데이터베이스 마다 제공하는 문법이 다르다. 개발과정에서 사용하는 데이터베이스가 변경될 수도 있다. 이때 JPA는 작성한 쿼리의 내용을 변경하지 않고 데이터베이스 방언을 수정하는 것으로 데이터베이스 의존성을 낮출 수 있다.

## 애플리케이션 개발

아래는 JPA의 동작 코드의 일부분이다.
```java
import javax.persistence.*;
import java.util.*;

public class Main {

  public static void main(String[] args) {
    // Entity Manager Factory 생성
    EntityManagerFactory factory = Persistence.createEntityManagerFactory("jpabook");
    // Entity Manager 생성
    EntityManager manager = factory.createEntityManager();
    // 트랜잭션 획득
    EntityTransaction tx = manager.getTransaction();

    try {
      tx.begin();
      ~~~ // 비즈니스 로직
      tx.commit();

    } catch(Exception e) {
      tx.rollback();
    } finally {
      manager.close();
    }
    factory.close();

  }

}
```

- EntityManagerFactory
  - 파라미터로 전달 받은 이름을 `persistence.xml`에서 찾아서 엔티티 매니저 팩토리를 생성한다.
  - `persistence.xml` 설정 정보를 읽어서 JPA 기반 객체를 만들고, JPA 구현체에 따라서는 데이터베이스 커넥션 풀을 생성하기도 한다.
  - 생성 비용이 매우 크기에 **엔티티 매니저 팩토리는 애플리케이션 전체에서 딱 한번만 생성하고 공유해서 사용해야하한다.**
- EntityManager
  - JPA의 기능 대부분은 엔티티 매니저가 제공한다. 
  - 엔티티 매니저를 사용해서 엔티티를 데이터베이스에 등록, 수정, 삭제, 조회할 수 있다
  - 엔티티 매니저는 데이터베이스 커넥션을 유지하면서 데이터베이스와 통신한다.
  - **데이터베이스 커넥션과 밀접한 관계가 있으므로 스레드 간에 공유하거나 재사용해선 안된다.**

### 비즈니스 로직

엔티티 매니저를 통해 CRUD 작업을 수행하는 것을 알아보자.

```java
Member member = new Member(id, age, name);
em.persist(member); // 저장
```

`persist()` 메소드에 저장할 엔티티를 넘겨주는 것으로 엔티티를 저장할 수 있다.
JPA는 멤버 엔티티의 매핑 정보를 분석해서 `INSERT` 쿼리를 작성한 후 데이터베이스에 전달한다

```java
member.setAge(age);
```

수정은 매우 단순하다. 엔티티를 수정한 후에 `persist`를 호출해 저장해야될 것 같지만, JPA는 엔티티의 변경 내용을 추적하는 기능을 갖추고 있다.

```java
em.remove(member);
```

삭제할 엔티티를 넘기면 `DELETE` 쿼리를 생성해서 데이터베이스에 넘긴다.


```java
Member findMember = em.find(Member.class, id);
```

데이터베이스에서 멤버를 하나 찾은 뒤 반환한다. 이때 아래와 같은 SQL 쿼리가 실행된다

`SELECT * FROM Member WHERE ID='id'`

