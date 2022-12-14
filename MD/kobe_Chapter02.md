# 아키텍처 개요

### 네 개의 영역

표현, 응용, 도메인, 인프라 스트럭처는 아키텍처를 설계할 때 출현하는 전형적인 네가지 영역이다.

표현 : 사용자의 요청을 받아 응용 영역에 전달하고, 응욕 영억의 처리 결과를 다시 사용자에게 보여준다. 
(스프링 MVC은 표현 영역을 위한 기술, 표현 영역의 사용자는 웹 브라우저 or REST API 호출하는 시스템)

웹 어플리케이션의 “표현 영역”은 HTTP 응답, 요청을 “응용 영역”이 필요로 하는 형식으로 변환하여 전송한다.
ex) 객체를 JSON 형식으로 만들어 return 

응용: 표현 영역을 통해 요청을 전달받는 응용 영역은 시스템이 사용자에게 제공해야 할 기능을 구현
ex) “주문 등록", ”주문 취소"..

응용 영역은 기능을 구현하기 위해서 도메인 모델 영역을 사용한다. (order.cancel())

응용 서비스는 로직을 직접 수행하는 것이 아니라. 도메인 모델에 수행을 위임한다.

도메인 : 도메인 영역은 도메인 모델을 구현한다.  

Order 도메인은 “배송지 변경", “결제 완료", “주문 총액 계산"과 같은 핵심 로직을 구현한다.

인프라 스트럭처 : 인프라 스트럭처는 구현 기술을 다룬다. 
DB 접근 기술, 메시징 큐 메시지 전송, REST API 호출 → 논리적이라기 보단 실제 구현.

표현, 응용, 도메인 계층에서는 필요 코드를 직접 만들지 않고, 인프라 스트럭처에서 제공하는 기능을 사용하여 개발 → JDBC 사용하는 것도 인프라 스트럭처(?)

### 계층 구조 아키텍처

계층 구조 아키텍처의 특성 상 상위 계층에서 하위 계층으로의 의존만 존재, 하위 계층은 상위 계층에 의존하지 않는다. → 인프라 스트럭처는 어디에도 의존하지 않는다. (의존 **[≒](https://decay2u.tistory.com/29)** 존재한다)

계층 구조를 엄격히 지키기 위해서는  표→ 응 → 도 → 인 형식을 지켜야 하지만, 개발의 편의성을 위해
응 → 인, 도 → 인 같이 유연하게 적용하기도 한다.

하지만 인프라스트럭처에 의존하게 되면 테스트의 어려움과 기능 확장의 어려움이 발생하는데, DIP로 해당 문제를 해결 할 수 있다.

### DIP

고수준 모듈 : 고수준 모듈이란 의미있는 단일 기능을 제공하는 모듈이다. 고수준 모듈의 기능을 구현하기 위해서는 여러 하위 기능이 필요하다. → “가격 할인"이라는 고수준 모듈을 구현하기 위해서는, 고객 정보를 알아야하고가격 할인 룰을 적용 시켜야 하는데, 이 두가지가 저수준 모듈이다.

- 고수준 모듈이 제대로 동작하려면 저수준 모듈을 사용해야하는데, 이러한 구현을 하게 된다면 구현 변경과 테스트가 어려워진다.
- DIP는 이러한 문제를 해결하기 위해 저수준 모듈이 고수준 모듈에 의존하도록 바꾼다.→ 구현과 테스트가 용이

```java
RuleDiscounter ruleDisocunter = new DroolsRuleDiscounter();
// 저수준 객체 생성 

CalculateDiscountService disService = new CalculateDiscounterService(ruleDiscounter);
// 생성자 방식으로 주입

```

- 이러한 구현을 통해 ruleDiscounter의 구현 방식이 바뀌어도 저수준 구현 객체를 생성하는 코드만 변경하면 된다.

- DIP 역시 항상 적용하는 것이 좋은게 아닌, 구현 기술에 의존적인 코드를 사용하는 것이 효과적일 수 있다.

### 도메인 영역의 주요 구성 요소

4개의 계층 중 도메인 영역은 도메인의 핵심 모델을 구현한다. 

도메인 요소를 구성하는 주요 구성요소

- 엔티티 : 고유의 식별자를 갖는 객체로 자신의 라이프 사이클을 가진다. 
주문, 회원 상품과 같이 도메인의 고유한 개념
- 밸류 : 고유의 식별자를 가지지 않는 객체 개념적으로 하나인 값→ Money, Address
- 애그리거트 : 연관된 엔티티와 밸류 객체를 개념적으로 하나로 묶은 것.
EX) 주문과 관련된 Order엔티티, OrderLine 밸류, Orderer 밸류 객체를 “주문" 애그리거트로 묶을 수 있다.
- 리포지토리 : 도메인 모델의 영속성을 처리한다. EX) DBMS 테이블에서 엔티티 객체를 로딩하거나 저장하는 기능
- 도메인 서비스 : 특정 엔티티에 속하지 않은 도메인 로직을 제공한다. 
EX) 할인 금액 계산은 상품, 쿠폰 회원등급, 구매 금액 등 다양한 조건을 이용해서 구현하는데, 도메인 로직이 여러 엔티티와 밸류를 필요하면 도메인 서비스에서 로직을 구현.

**엔티티** 

DBMS의 엔티티 ≠ 도메인 모델 엔티티

도메인 모델 엔티티는 Order안에 있는 OrderNo와 같은 데이터도 주지만, ChangeShipping과 같은 기능도 제공해준다.

도메인 모델의 엔티티는 단순히 데이터를 담고 있는것이 아닌, 데이터를 캡슐화 하여 기능을 제공하는 객체이다. 

**밸류**

관계형 DBMS에서 표현하기 힘든 밸류를 도메인 모델에서는 보다 쉽게 구현이 가능하다.
(밸류는 불변 객체로 구현할 것을 권장, if(밸류가 바뀐다면 새로운 밸류 객체 생성)

**애그리거트**

도메인이 커질수록 엔티티와 밸류는 많아지고, 복잡해진다. 복잡한 도메인을 구현하기 위해 개발자는 전체가 아닌 하나의 엔티티와 밸류에 집중하게 되는데 → 큰 수준의 모델을 이해하지 못해 Model 관리가 어려운 상황에 빠질 수 있다.

지도의 예와 같이 상위 수준의 모델을 볼 수 있어야 전체 모델의 관계와 개별 모델을 이해하는데 도움이 된다.
이렇게 전체 구조를 이해하는데 도움이 되는 것이 애그리거트이다. 
→ 주문이라는 애그리거트 안에 ‘배송지 정보’ , ‘주문자' 등등 

애그리거트를 사용하면 개별 객체가 아닌 관련 객체를 묶어 군집 단위로 모델을 바라볼 수 있는데, 객체들 간의 관계가 아닌 애그리거트 간의 관계를 중심으로 구현하고, 큰 틀에서 도메인을 모델을 관리할 수 있다.

- 루트 엔티티 : 애그리거트에 속해 있는 엔티티와 밸류 객체를 사용해 애그리거트가 구현할 기능 제공. (Order)

애그리거트를 구현할때는 고려할 것이 많다. 어떻게 애그리거트를 구현했는지에 따라서 구현이 원할해지기도, 트랜잭션 범위도 달라진다. 

**리포지토리**

도메인 정보를 저장하기 위해서는 RDMS, NoSQL와 같은 물리적 저장소가 필요하다. 이러한 물리적 저장소를 위한 모델이 리포지토리이다. → 구현을 위한 도메인 모델

리포지토리는 애그리거트 단위로 도메인 객체를 저장하고, 조회하는 기능을 정의한다.

```java
public interface OrderRepository {

 Order findByNumber(OrderNumber number);
 void save(Order order);
 void delete(Order order);
 
 } // 대상을 찾고, 저장하는 단위가 애그리거트 루트인 Order이다.
```

```java
Public class CancelOrderService {
	
	private OrderRepository orderRepository;
	
	public void cancel(OrderNumber number) {
		Order order = orderRepository.findByNumber(number);
		if (order == null) throw new NoOrderException(number);
		order.cancel();
	
	} //주문 취소 서비스에서 Order 객체를 리포지토리에서 꺼내와 사용한다.
```

도메인 관점에서 OrderRepository는 도메인 객체를 영속화 하는데 필요한 기능을 추상화한 것으로 고수준 모듈에 속한다. (도메인 계층)

OrderRepository를 구현하기 위한 JpaOrderRepository는 저수준 인프라스트럭처 모듈에 속한다.

응용 서비스는 의존 주입과 같은 방식을 사용해서 리포지토리 구현 객체에 접근한다. (CancelOrderService)

리포지토리는 응용 서비스가 필요로하는 메소드를 제공한다. (save 메소드 , 루트 식별자로 조회하는 findById)

### 요청 처리 흐름

1. 사용자가 어플리케이션과 같은 소프트웨어를 이용하여 기능을 실행 → 표현 영역에서 요청을 받음
2. 표현 영역에서 사용자가 전송한 데이터 형식이 올바른지 검사하고, 문제가 없다면 데이터를 이용해 응용 서비스에 기능을 실행 위임 → ( 객체를 JSON으로 잘 변경하여 보냈는가)
3. 응용 서비스는 도메인 모델을 이용하여 기능을 구현 기능 구현에 필요한 도메인 객체를 리포지토리를 활용하여 저장.
4. 기능을 제공할때는 트랜잭션을 활용하여 물리 저장소에 올바르게 저장될 수 있도록 도메인의 상태를 변경해야한다.

### 인프라 스트럭처 개요

인프라 스트럭처는 표현 영역, 응용 영역, 도메인 영역을 지원한다. 

도메인 객체의 영속성 처리, 트랜잭션, SMTP 클라이언트, REST 클라이언트 등 보조 기능 지원한다.

도메인 영역과, 응용 영역에서 인프라 스트럭처의 기능을 직접 사용하기 보다는 해당 영역에서 구현한 인터페이스를 활용하여 인프라 스트럭처에서 구현하는 것이 시스템을 더 유연하고, 테스트하기 쉽게 만들어준다. (DIP)

DIP의 장점을 해치지 않는 선에서 구현의 편리함을 위해 응용, 도메인 영역에서 기술을 구현하는 것도 나쁘지 않다.

### 모듈 구성

도메인 모듈은 도메인에 속한 애그리거트를 기준으로 패키지를 구현하는데, 도메인이 크면 하위 도메인 별로 모듈을 나눈다.

EX) Order 애그리거트, OrderLine, Orderer, OrderRepository 는 order.domain 패키지에 위치 시킨다.