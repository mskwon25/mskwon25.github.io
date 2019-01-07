---
title:        "JPA-Projections"
categories:
              - "JPA"
tags:         
              - "Spring"
              - "JPA"
              - "ORM"
              - "DB"
---
### JPA Projections 학습기

---

요즘에는 개발하시는 많은 분들이 ORM을 사용한다. 나 역시 최근 JPA에 대해 조금씩 공부하고 있는데, 아직은 모르는게 많아 문제 하나를 해결하는데 많은 시간을 할애하고 있다. 정말.. 배움cd 의 끝은 없는것인가...

### 문제의 발견

잡설은 빠르게 넘어가자. 나는 최근, 업무를 진행하면서 그룹별 카운트를 구해 화면에 제공해주는 API를 만들게 되었다.

Mysql을 이용하면 쿼리가 간단하게 나오는데 JPA로 할려고 하니 뭔가 Entity를 가져와서 연산하는 로직을 만들어야 하니 허허.. JPA가 만능은 아니구나 하던 차에!! @Query` Annotation 발견!

```java
public interface PersonRepository extends JpaRepository<Person, Serializable> {

    @Query("SELECT COUNT(p.id) AS count, p.address AS address FROM Person p GROUP BY p.address")
    List<CountByAddress> getAddressAndPersonCount();
}

```

게다가 `Query`Annotation을 사용하면 내가 원하는 값을 특정 model로 한 번에 Mapping 할 수 있다.

근데 만들고 보니 `CountByAddress` 부분에 자꾸 빨간불이 들어오네? 읽어보니 결국 `Class`가 아니라 `Interface`로 구현하라는데

![what](/assets/images/jpa-projections/what.jpg)

일단 Interface로 만들라고 하니 아래와 같이 만들긴한다.

```java
public interface CountByAddress {}
```

### 1차 테스트

그리고 실제 테스트를 해보자.

```java
package com.canopus.jpa;

import com.canopus.jpa.entity.Person;
import com.canopus.jpa.model.CountByAddress;
import com.canopus.jpa.model.NameAgeOnly;
import com.canopus.jpa.repository.PersonRepository;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Transactional;

import java.util.Collection;
import java.util.List;

import static org.hamcrest.Matchers.is;
import static org.junit.Assert.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class PersonRepositoryTests {

    @Autowired
    private PersonRepository personRepository;

    @Before
    @Rollback
    public void setUp() throws Exception {
        Person person1 = Person.builder().name("고준희").age(34).phone("01012344321").address("서울시 강남구").build();
        Person person2 = Person.builder().name("조보아").age(28).phone("01011112222").address("서울시 서초구").build();
        Person person3 = Person.builder().name("한지민").age(37).phone("01023233232").address("서울시 동작구").build();
        Person person4 = Person.builder().name("서현진").age(34).phone("01087655678").address("서울시 서초구").build();

        personRepository.save(person1);
        personRepository.save(person2);
        personRepository.save(person3);
        personRepository.save(person4);
    }

    @Test
    public void annotationTest() {
        List<CountByAddress> countByAddressList = personRepository.getAddressAndPersonCount();
        assertThat(countByAddressList.size(), is(3));
        assertThat(countByAddressList.get(0).getCount(), is(1));
    }
}

```



```java
public interface CountByAddress {
    int getCount();

    String getAddress();
}
```

임시로 getter를 만들어주고 잘 가져 오는지 Test를 해보자!

![first-test](/assets/images/jpa-projections/first-test.jpg)

**엥? 이게 된다고???? 뭐야 구현체가 없는데 값을 어디서 가져오는거냐? 어떻게 이게 돼?**

### 2차 테스트

이해가 잘 안되니 비슷하게 회원의 이름이랑 나이만 가져오는 메소드를 다시 만들어보자.

```java
public interface PersonRepository extends JpaRepository<Person, Serializable> {
    @Query("SELECT COUNT(p.id) AS count, p.address AS address FROM Person p GROUP BY p.address")
    List<CountByAddress> getAddressAndPersonCount();

    Collection<NameAgeOnly> findByAgeGreaterThan(int age);
}
```

다시 테스트 해본다

```java
@Test
public void annotationTest() {
    Collection<NameAgeOnly> nameAgeOnlyList = personRepository.findByAgeGreaterThan(30);
    assertThat(nameAgeOnlyList.size(), is(3));

    Person result = personRepository.findByName("서현진");
    assertThat(result.getAge(), is(34));
}
```

![second-test](/assets/images/jpa-projections/second-test.jpg)

는 또 성공!

허허.. `Getter` 껍데기만 있는데 대체 이게 뭔일 이래?

### Debugger 출동

![debug](/assets/images/jpa-projections/debug.jpg)

자세히 보면 Proxy를 이용해 해당 data를 mapping해주는걸로 보이는데 이게 어떻게 가능한 걸까?

### 공식 문서 뒤져보기

> Spring Data query methods usually return one or multiple instances of the aggregate root managed by the repository. However, it might sometimes be desirable to create projections based on certain attributes of those types. Spring Data allows modeling dedicated return types, to more selectively retrieve partial views of the managed aggregates.

Spring JPA Document에 있는 Projections 부분을 보면 위와 같이 설명되어 있는데 간단히 말해, Entity가 아니라 사용자가 만든 특정 Model 로도 Obejct를 생성할 수 있도록 허용한다는 것이다. 그게 바로 `JPA Projections`이라는 사실.

이러한 구조가 가능할 수 있는 이유 역시 가이드에 존재하는데 내용은 아래와 같다.

> The query execution engine creates proxy instances of that interface at runtime for each element returned and forwards calls to the exposed methods to the target object.

JPA의 핵심인 query excution engine은 런타임시에 proxy instance를 생성해 target obeject로 향한 call에 대해 각각의 element를 리턴해준다!
결국 이 기능을 통해 Jpa의 Projections이라는 기능이 구현될 수 있었다.

### 오늘의 교훈

잘 모르는 부분 찾느라 시간은 많이 걸렸지만 덕분에 JPA의 새로운 기능을 하다 더 배워간다..

### Reference

- [Spring Data JPA - Reference Documentation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html)
