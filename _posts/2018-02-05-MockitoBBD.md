---
title:        "BBD스타일을 위한 MockitoBBD"
categories:
              - "TDD"
tags:         
              - "JUnit"
---

### BBD(Behavior-driven development)란 무엇인가?

>In software engineering, behavior-driven development BDD is a software development process that emerged from test-driven development TDD. Behavior-driven development combines the general techniques and principles of TDD with ideas from domain-driven design and object-oriented analysis and design to provide software development and management teams with shared tools and a shared process to collaborate on software development.
>

Wiki를 요약해보면 결국 BDD에 대한 정의는 TDD와 큰 차이는 없다. 그래서 BDD에 대한 글을 찾아 보다가 `Dan North`의 `INTRODUCING BDD` 라는 유명한 글을 읽게 되었다. 이 글에 따르면 BDD의 가장 큰 특징은 비즈니스 요구사항에 집중하여 TestCase를 만드는 것이다. 이러한 틍징에 따라서 BDD는 TDD에 비해 보다 자연어에 가까운 테스트 구조가 가능하도록 다양한 기능을 지원한다. 그 중 이번에 소개할 기능은 바로 `Aliases for behavior driven development`이다

### Aliases for behavior driven development 사용해보기

`Behavior Driven Development` 스타일의 테스트 작성방법은 테스트 method에 기본으로 `//given` `//when` `//then` 이라고 주석을 달아두는 것이다.

문제는 when을 이용한 stubbing이 `//given` `//when` `//then` 주석에 잘 맞아떨어지지 않는다는 것이다. 왜냐하면 stubbing이 when이 아닌 given에 속하기 때문이다.

따라서 BDDMockito 클래스는 stub을 할 때 사용할 메소드로 BDDMockito.given(Object)를 제공한다

아래의 테스트를 통해 이해해보자
~~~java
Seller seller = mock(Seller.class);
Shop shop = new Shop(seller);

public void TestShouldBuyBread() throws Exception {
  //given
  when(seller.askForBread()).thenReturn(new Bread());

  //when
  Goods goods = shop.buyBread();

  //then
  assertThat(goods, containBread());
}
~~~

위의 예제에서는 기능상의 문제는 없지만 `//given` 주석임에도 불구하고 `when` 메소드를 사용했기 때문에 Test 시나리오의 진행이 자연스럽지 못하다.

~~~java
import static org.mockito.BDDMockito.*;

Seller seller = mock(Seller.class);
Shop shop = new Shop(seller);

public void shouldBuyBread() throws Exception {
  //given  
  given(seller.askForBread()).willReturn(new Bread());

  //when
  Goods goods = shop.buyBread();

  //then
  assertThat(goods, containBread());
}
~~~

따라서 위와 같이 BDDMockito가 지원하는 given 메소드를 사용하게되면 보다 자연스러운 TestCase를 구현할 수 있다.
