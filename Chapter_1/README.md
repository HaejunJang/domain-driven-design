# 1장 도메인 모델 시작하기

### 도메인이란?

소프트웨어로 해결하고자 하는 문제 영역을 도메인이라 한다.

코딩에 앞서 가장 중요한것은 요구사항을 올바르게 이해하는것이다.

어떻게 해야할까?

- 바로 개발자와 전문가가 직접 대화하는것이다.
- 개발자는 도메인 전문가 만큼은 아니겠지만 개발자도 도메인 지식을 갖춰야 한다.

### 도메인 모델

- 기본적으로 도메인 자체를 이해하기 위한 개념 모델이다.
- 특정 도메인을 개념적으로 표현한것.
- 도메인 모델을 통해 여러 관계자들이 동일한 모습으로 도메인을 이해하고 지식을 공유하는데 도움이 된다.

### 개념모델과 구현모델

- 개념 모델은 순수하게 문제를 분석한 결과물이다. 개념 모델은 데이터베이스, 트랜잭션 처리, 성능, 구현 기술과 같은 것을 고려하지 않는다. 따라서 실제 코드를 작성할 떄 개념 모델을 있는 그대로 사용할 수 없다. 이를 위해 개념 모델을 구현 가능한 형태의 모델로 전환하는 과정을 거친다.
- 처음부터 완벽한 개념 모델을 만들기보다는 전반적인 개요를 알 수 있는 수준으로 개념 모델을 작성해야 한다.
- 프로젝트 초기에는 개요 수준의 **개념 모델로 도메인에 대한 전체 윤곽**을 이해하고, 구현하는 과정에서 개념 모델을 **구현 모델**로 점진적으로 발전시켜 나간다.

### 엔티티

- 엔티티는 서로 다른 식별자를 갖는다
- 식별자의 종류
  - 특정 규칙에 따라 생성 ex) 시간 + 특정 규칙 202501130942 + 규칙
  - UUID나 Nano ID와 같은 고유 식별자 생성
  - 값을 직접 입력
  - 일련번호 사용(시퀀스나 DB의 자동 증가 컬럼)

### 밸류 타입

- 개념적으로 완전한 하나를 표현할 때 사용
- 불변으로 구현
- 식별자를 위한 밸류 타입을 사용해서 의미가 잘 드러나도록 한다.

- 예시 코드

```java
public class ShippingInfo {
	private String receiverName;
	private String receiverPhoneNumber;

	private String shippingAddress1;
	private String shippingAddress2;
	private String shippingAddress3;
}
```

다음과 같은 배송 정보에 대한 엔티티 코드가 있다 해보자

`receiverName`, `receiverPhoneNumber`는 모두 받는 사람에 대한 정보이므로 `Receiver` 라는 클래스로 묶을 수 있을 것이고, `ShippingAddress123`은 주소 `Address`라는 클래스로 묶을 수 있을 것이다.

```java
public class ShippingInfo {
	private Receiver receiverName;
	private Receiver receiverPhoneNumber;

	private Address shippingAddress1;
	private Address shippingAddress2;
	private Address shippingAddress3;
}
```

### 왜 굳이 밸류 타입을 사용해야할까?

- 밸류 타입을 사용할 경우 코드의 의미를 더 잘 이해할 수 있다.

### 도메인 모델에 set 넣지 않기

- 상태 변경을 위한 set사용시 도메인 지식이 코드에서 사라진다.
- 객체를 생성할 때 온전하지 않은 상태가 될 수 있다.

### 도메인 용어와 유비쿼터스 언어

코드를 작성할 때 도메인에서 사용하는 용어는 매우 중요하다.

상품에 대해서 결제 대기중, 상품 준비 중 과 같이 여러 단계가 있을때 각각의 상태를 만약 STEP1, STEP2, STEP3 용어를 사용한다면 코드를 분석하고 이해하는데 어려움이 있을 것이다.

- 예시 코드

```java
public enum OrderState {
	PAYMENT_WAITING, PREPARING, SHIPPED, ...
}
```

- 예시와 같이 상태를 도메인에 관련된 공통의 언어를 사용한다면 용어의 모호함을 줄이고 개발자는 도메인 코드 사이에서 **불필요한 해석 과정**을 줄일 수 있다.
- 이와 같은 언어를 **유비쿼터스 언어**라고 한다
