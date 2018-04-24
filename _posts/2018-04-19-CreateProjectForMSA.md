---
title:        "IntelliJ에서 비동기 리액티브 프로젝트 생성하기 "
categories:
              - "MSA"
tags:         
              - "Web-Flux"
              - "MSA"
              - "Async"
              - "Pub/Sub"
---

### Reactive MicroService 구현
* Spring5 기반의 WebFlux 사용
* RabbitMQ와 같은 메세징 서비를 통한 비동기적 호출

### WebFlux를 이용한 간단한 예제
`Application.java`
~~~java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

@RestController
class GreetingController {

    @RequestMapping("/")
    Mono<Greet> greet(){
        return Mono.just(new Greet("Hello World!"));
    }
}

class Greet {
    private String message;

    public Greet() {
    }

    public Greet(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
~~~
* WebFlux를 시험해 보기 위해 Mono를 리턴해주는 Controller와 Greet 객체를 생성한다.

`ApplicationTest.java`
~~~java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)
public class ApplicationTests {

	WebTestClient webClient;
	@Before
	public void setup(){
		webClient = WebTestClient.bindToServer()
				.baseUrl("http://localhost:8080").build();
	}

	@Test
	public void testWebFluxEndpoint() {
		webClient.get().uri("/")
				.accept(MediaType.APPLICATION_JSON)
				.exchange()
				.expectStatus().isOk()
				.expectBody(Greet.class).returnResult()
				.getResponseBody().getMessage().equals("Hello World!");
	}
}
~~~
* WebFlux를 테스트 하기 위해서는 기존의의 RestTemplete.webclient가 아니라 Webclient, WebTestClient를 이용한다.

### 리액티브 스트림의 이해
* 리액티브 스트림은 아래의 4가지로 구성된다.
* publisher, subscriber, subscription, processor

### RabbitMQ 를 이용한 간단한 예제
`Application`
~~~java
@SpringBootApplication
public class BootmessagingApplication implements CommandLineRunner {

    @Autowired
    Sender sender;

    public static void main(String[] args) {
        SpringApplication.run(BootmessagingApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        sender.send("Hello Messaging..!!!");
    }
}

@Component
class Sender {

    @Autowired
    RabbitMessagingTemplate template;

    @Bean
    Queue queue() {
        return new Queue("TestQ", false);
    }

    public void send(String message) {
        template.convertAndSend("TestQ", message);
    }
}

@Component
class Receiver {

    @RabbitListener(queues = "TestQ")
    public void processMessage(String content) {
        System.out.print(content);
    }
}
~~~

* 리액티브 스트림의 4가지 요소를 위의 코드를 통해 이해해보자
* `Sender` Component는 Publisher에 해당하며 발행을 담당한다
* `Receiver` Component는 Subscriber에 해당하며 `TestQ`를 구독한다
* Receiver는 `@RabbitListner` annotaion을 이용해 RabbitMQ를 구독하는데 이와 같은 행위가 `Subscription`에 해당된다
* 마지막으로 `processMessage()` 메소드는 리액티브 스트림의 processor에 해당한다.

위의 코드를 빌드하게 되면 CommandLineRunner로 인해 run 메소드가 수행되고 Sender가 RabbitMQ의 `TestQ` Queue에 메세지를 보내게 된다. 이 `TestQ`를 구독하고 있던 Receiver가 processMessage 메소드를 수행시켜 콘솔에 해당 메세지가 찍힌다.

### 이 외에 MSA와 관련한 이슈들
1. MSA를 도입하게 되면 필연적으로 보안과 관련한 이슈를 처리해야한다. 서로 통신하는 MicroService가 많을 수록 중요성은 더 커진다. 중요한 점은 각 모듈별로 보안과 관련한 이슈들을 handling하기에는 오버헤드가 커진다는 단점이 발생한다. Spring Security에는 이러한 점을 보완하기 위한 다양한 기술을 제공한다.
2. 일반적으로 하나의 도메인에서 다른 도메인에 데이터 요청하는 것은 보안상의 이유로 금지되어 있지만 MSA는 모듈간의 통신으로 구성되기 때문에 이러한 제약사항을 맞이할 수 밖에 없다. 이 때는 CORS(Cross-origin Resource Sharing)를 적절히 이용하여 구현한다.
3. 설마 수 많은 API문서를 직접 정리하고 있는건 아니겠지? 이럴 땐 Swagger를 이용해보자
