
# Chapter1 디자인 패턴과 프로그래밍 패러다임

1. [디자인 패턴](#11-디자인-패턴)
2. [프로그래밍 패러다임](#12-프로그래밍-패러다임)

# 1.1 디자인 패턴

프로그램을 설계할 때 발생했던 문제점들을 객체 간의 상호 관계 등을
이용하여 해결할 수 있도록 하나의 '규약' 형태로 만들어 놓은 것

## **1.1.1 싱글톤 패턴**

- 하나의 클래스에 오직 하나의 인스턴스만 가지는 패턴
- ex) 데이터 베이스 연결 모듈에 많이 사용

- 장점: 인스턴스를 생성할 때 드는 비용이 줄어듦
- 단점 : 의존성이 높아짐

  - TDD를 할때 좋지 않음
    - 독립적인 인스턴스를 만들기 어려움 => 즉, 테스트에서 객체를 내마음대로 바꿀 수 없음
    ```java
    MyService service = new MyService(); // 이런 독립 인스턴스를 만들고 싶지만.. 싱글톤은 생성자를 private으로 막고, 하나의 고정된 인스턴스만 리턴함. => 테스트마다 새로운 인스턴스 만들기 어려움, 테스트 격리가 어려움
    ```
  - 해결책 : 의존성 주입

    - 의존성 주입을 통해 모듈 간의 결합을 조금 더 느슨하게 만들 수 있음
    - 싱글톤 패턴을 직접 사용할 때는 TDD가 어려움
    - DI 컨테이너가 관리하는 싱글톤을 사용하자

    - 의존성 주입의 장점
      - 모듈들을 쉽게 교체할 수 있는 구조
      - 테스팅 용이
      - 마이그레이션 용이
    - 의존성 주입의 단점
      - 모듈들이 더욱더 분리되어 클래스 수가 늘어나 복잡도 증가
      - 런타임 패널티 (약간)

- - 추가) 싱글톤 방식으로 DB연결하는 예시
  * 필요하녀 getConnection()으로 꺼내 쓰는 느낌

```java
public class MySqlConnectionManager {

    private static MySqlConnectionManager instance;

    private Connection connection;

    private MySqlConnectionManager() {
        try {
            String url = "jdbc:mysql://localhost:3306/mydb";
            String user = "root";
            String password = "1234";

            connection = DriverManager.getConnection(url, user, password);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public static MySqlConnectionManager getInstance() {
        if (instance == null) {
            instance = new MySqlConnectionManager();
        }
        return instance;
    }

    public Connection getConnection() {
        return this.connection;
    }
}
```

## 1.1.2 팩토리 패턴

- 객체를 사용하는 코드에서 객체 생성 부분을 떼어내 추상화한 패턴
- 상속관계에 있는 두 클래스에서 상위 클래스가 중요한 뼈대를 결정
- 하위 클래스에서 객체 생성에 관한 구체적인 내용을 결정하는 패턴

## 1.1.3 전략 패턴

- 전략패턴 = 정책패턴
- 객체의 행위를 바꾸고 싶은 경우 직접 수정하지 않고 전략이라고 부르는 캡슐화한 알고리즘을 컨텍스트 안에서 바꿔주면서 상호 교체가 가능하게 만드는 패턴

          ┌───────────────────┐
          │    Strategy       │   ← 전략(인터페이스)
          │  (interface)      │
          │  + execute()      │
          └────────┬──────────┘
                   │

  ┌───────────────┼────────────────┐
  │ │ │
  │ │ │
  ▼ ▼ ▼
  ┌───────────┐ ┌───────────┐ ┌───────────┐
  │ StrategyA │ │ StrategyB │ │ StrategyC │ ← 전략 구현체들
  │ execute() │ │ execute() │ │ execute() │
  └───────────┘ └───────────┘ └───────────┘

## 1.1.4 옵저버 패턴

- 주체가 어떤 객체의 상태 변화를 관찰하다가 상태 변화가 있을 때마다 메서드 등을 통해 옵저버 목록에 있는 옵저버들에게 변화를 알려주는 디자인 패턴
- 주로 이벤트 기반 시스템에 사용하며 mvc 패턴에도 사용한다

- - 추가) 옵저버 패턴을 구현한 간단 예시
- 스프링에서는 ApplicationEvent, ApplicationListener를 사용할 수 있다.

```java
1) 이벤트 클래스 생성
import org.springframework.context.ApplicationEvent;

public class OrderCreatedEvent extends ApplicationEvent {

    private final String orderId;

    public OrderCreatedEvent(Object source, String orderId) {
        super(source);
        this.orderId = orderId;
    }

    public String getOrderId() {
        return orderId;
    }
}

2) 발행자 (Publisher)
import lombok.RequiredArgsConstructor;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class OrderService {

    private final ApplicationEventPublisher publisher;

    public void createOrder(String orderId) {
        System.out.println("주문 생성 완료: " + orderId);

        // 이벤트 발행
        publisher.publishEvent(new OrderCreatedEvent(this, orderId));
    }
}

3) 구독자(옵저버) — 이벤트 리스너
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class NotificationListener {

    @EventListener
    public void handleOrderCreated(OrderCreatedEvent event) {
        System.out.println("알림 발송: 주문 " + event.getOrderId() + " 이(가) 생성되었습니다!");
    }
}

4) 사용
@Autowired
OrderService orderService;

orderService.createOrder("ORDER-001");
```

### 상속

- 자식 클래스가 부모 클래스의 메서드를 상속받아 사용하며 자식 클래스에서 추가 및 확장을 할 수 있는 것
- 장점 : 재사용성 향상, 중복성 최소화

### 구현

- 부모 인터페이스를 자식 클래스에서 “재정의”하여 구현하는 것
- 상속과 다르게!!!! 반드시 부모 클래스의 메서드를 재정의하여 구현해야함

### 상속과 구현의 차이

- 상속은 `일반클래스, abstract 클래스`를 기반으로 구현
- 구현은 `인터페이스`를 기반으로 구현

## 1.1.5 프록시 패턴과 프록시 서버

- 프록시 패턴
  - 1. 대상 객체에 접근하기 전 그 접근에 대한 흐름을 가로챔
  - 2. 해당 접근을 필터링하거나 수정하는 등의 역할을 하는 계층이 있는 디자인 패턴
- 프록시 서버

  - 서버와 클라이언트 사이에서 클라이언트가 자신을 통해 다른 네트워크 서비스에 간접적으로 접속할 수 있게 해줌

  - nginx
    - 비동기 이벤트 기반의 구조와 다수의 연결을 효과적으로 처리 가능한 웹서버
    -
  - CloudFlare
    - CDN 서비스
    - 웹 서버 앞단에 프록시 서버로 두어 ddos 공격 방어나 https 구축에 쓰임
    - cloudflare가 의심스러운 트래픽인지를 먼저 판단해 captcha 등을 기반으로 이를 일정부분 막아주는 역할도 수행함

- CORS 란 무엇인가?
  - 서버가 웹 브라우저에서 리소스를 로드할 때 다른 오리진을 통해 로드하지 못하는 HTTP 헤더 기반 메커니즘

## 1.1.6 이터레이터 패턴

- 이터레이터를 사용하여 컬렉션의 요소들에 접근하는 디자인 패턴
- EX) 다른 자료 구조인 SET과 MAP임에도 똑같은 for a of b라는 이터레이터 프로토콜을 통해 순회가 가능함!

## 1.1.7 노출모듈 패턴

- 즉시 실행 함수를 통해 private, public 같은 접근 제어자를 만드는 패턴

## 1.1.8 MVC 패턴

- M : model
- V : view
- C : controller

애플리케이션의 구성요소를 세가지 역할로 구분
장점 : 재사용성과 확장성 용이
단점 : 복잡해질수록 모델과 뷰의 관계가 복잡해짐

### 모델

`모델` : 애플리케이션의 데이터인 데이터베이스, 상수, 변수등을 뜻함

### 뷰

`뷰` : 사용자 인터페이스 요소를 나타냄, 변경이 일어나면 컨트롤러에 전달

### 컨트롤러

`컨트롤러` : 하나이상의 모델과 하나 이상의 뷰를 잇는 다리 역할을 하며 이벤트 등 메인 로직을 담당

### MVC 패턴의 예시 스프링

- +) 추가 스프링 MVC가 MVC 패턴을 구현하는 방법
  ✔ Model

서비스, DTO, Entity, Repository 등 “데이터와 비즈니스 로직”

컨트롤러로부터 전달받아 뷰로 보내는 데이터

✔ View

Thymeleaf, JSP, Freemarker 같은 화면 렌더링 담당

✔ Controller

요청을 받고 서비스 호출

Model에 데이터 넣어서 View에게 전달

또는 REST API라면 JSON 형태로 응답

- 스프링 내부 흐름

```java
DispatcherServlet
      ↓
HandlerMapping (URL → 컨트롤러 찾기)
      ↓
Controller
      ↓
Service
      ↓
Model 반환
      ↓
ViewResolver (HTML 템플릿 찾기)
      ↓
View 렌더링

```

## 1.1.9 MVP 패턴

- CONTROLLER가 PRESENTER로 교체된 패턴
- VIEW와 PRESENTER가 1대1 결합이라 MVC보다 결합력이 세다..

## 1.1.10 MVVM 패턴

- 컨트롤러가 뷰모델로 바뀐 패턴

```java
  Model  ←→  ViewModel  ←→  View
```

```java
       ┌───────────────┐
       │     View      │
       │ (UI 화면)     │
       └───────▲───────┘
               │
       데이터바인딩 / 옵저버
               │
       ┌───────▼────────┐
       │   ViewModel     │
       │ 로직 + 상태관리 │
       └───────▲────────┘
               │
               │ Model 변경 요청
               │
       ┌───────▼────────┐
       │     Model       │
       │ (데이터/비즈니스) │
       └─────────────────┘
```

# 1.2 프로그래밍 패러다임

프로그래머에게 프로그래밍의 관점을 갖게 해주는 역할을 하는 개발 방법론

1. 선언형
2. 명령형

이 있다!

**`선언형`**은 함수형이라는 하위 집합!

**`명령형`**은 객체지향, 절차지향 으로 나뉨

## 1.2.1 선언형과 함수형 프로그래밍

함수형 프로그래밍은 선언형 패러다임의 일종

**순수 함수** - 출력이 입력에만 의존 하는 것

**고차 함수** - 함수가 함수을 값처럼 매개변수로 받아 로직을 생성할 수 있는 것

## 1.2.2 객체지향 프로그래밍

객체들의 집합으로 프로그램의 상호 작용을 표현하며 데이터를 객체로 취급하여 객체 내부에 선언된 메서드를 활용하는 방식

⇒ 설계에 많은 시간 소요, 처리속도가 상대적으로 느림

### 특징

1. **추상화**

복잡한 시스템으로부터 핵심적인 개념 또는 기능을 간추려 내는 것

```java
//예시
abstract class Animal {
    // 핵심 기능만 정의 (구현은 하위 클래스가 담당)
    abstract void sound();

    void eat() {
        System.out.println("먹습니다.");
    }
}

class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("멍멍!");
    }
}

class Cat extends Animal {
    @Override
    void sound() {
        System.out.println("야옹!");
    }
}

public class AbstractionExample {
    public static void main(String[] args) {
        Animal dog = new Dog();
        dog.sound();
        dog.eat();
    }
}

```

2. **캡슐화**

객체의 속성과 메서드를 하나로 묶고 일부를 외부에 암추어 은닉하는 것

```java
//예시
class User {
    private String name;
    private int age;

    // Getter / Setter로만 수정 가능
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        if (age >= 0) { // 유효성 검증
            this.age = age;
        }
    }
}

public class EncapsulationExample {
    public static void main(String[] args) {
        User user = new User();
        user.setName("Jiwon");
        user.setAge(22);

        System.out.println(user.getName());
        System.out.println(user.getAge());
    }
}

```

3. **상속성**

상위 클래스의 특성을 하위 클래스가 이어 받아서 재사용, 추가, 확장

4. **다형성**

하나의 메서드나 클래스가 다양한 방법으로 동작하는 것

Ex) 오버로딩, 오버라이딩

### 설계원칙 SOLID

1. **`단일책임원칙 SRP`**

모든 클래스는 각각 하나의 책임만 가져야 한다

2. **`개방 폐쇄 원칙 OCP`**

유지보수 사항이 생긴다면 토드를 쉽게 확장할 수 있도록 하고 수정할 때는 닫혀 있어야 하는 원칙! 기존의 코드는 잘 변경하지 않으면서도 확장은 쉽게할 수 있어야 한다.

```java
// 안 좋은 예 (새 결제 수단 추가 시 코드 수정 필요)
@Service
public class PaymentService {

    public void pay(String type) {
        if (type.equals("kakao")) {
            System.out.println("카카오페이 결제!");
        } else if (type.equals("naver")) {
            System.out.println("네이버페이 결제!");
        }
    }
}

✅ OCP 적용 (전략 패턴 + 스프링 DI)

1) 공통 인터페이스

public interface Payment {
    void pay();
}


2) 구현체

@Component
public class KakaoPay implements Payment {
    @Override
    public void pay() {
        System.out.println("카카오페이 결제!");
    }
}

@Component
public class NaverPay implements Payment {
    @Override
    public void pay() {
        System.out.println("네이버페이 결제!");
    }
}


3) PaymentService는 확장에는 열려있어도 수정 필요 없음

@Service
@RequiredArgsConstructor
public class PaymentService {
    private final List<Payment> payments;

    public void payAll() {
        payments.forEach(Payment::pay);
    }
}
```

3. **`리스코프 치환 원칙 LSP`**

프로그램의 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 바꿀 수 있어야 하는 것을 의미

4. **`인터페이스 분리 원칙 ISP`**

구체적인 여러 개의 인터페이스를 만들자

5. **`의존 역전 원칙 DIP`**

자신보다 변하기 쉬운 것에 의존하던 것을 추상화된 인터페이스나 상위 클래스를 두어 변하기 쉬운 것의 변화에 영향 받지 않개 하는 원칙

> “구체 클래스가 아닌, 추상(인터페이스)에 의존하라.”

## 1.2.3 절차형 프로그래밍

### 장점

코드의 가독성 GOOD

실행 속도 빠름

### 단점

모듈화 어려움

유지 보수성이 떨어짐
