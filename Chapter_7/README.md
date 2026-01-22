# 7장 도메인 서비스

### 여러 애그리거트가 필요한 기능

하나의 애그리거트에 넣기 애매한 도메인 기능은 억지로 특정 애그리거트에 넣으면 안된다.

억지로 구현할 경우 애그리거트는 자신의 책임 범위를 넘어서는 기능을 구현하기 때문에 코드가 길어지고 외부에 대한 의존이 높아지게 되며 코드를 복잡하게 만들어 수정이 어렵다.

### 계산 로직과 도메인 서비스

주로 계산 로직과, 외부 시스템 연동이 필요한 도메인 로직에서 사용한다.

- 계산 로직 : 여러 애그리거트가 필요한 계산 로직이나, 한 애그리거트에 넣기에는 다소 복잡한 계산 로직
- 외부 시스템 연동이 필요한 도메인 로직 : 구현하기 위해 타 시스템을 사용해야 하는 도메인 로직
- 도메인 서비스는 도메인 로직만 다루며 상태가 없다.

도메인 서비스는 도메인의 의미가 드러나는 용어를 타입과 메서드 이름으로 갖는다.

```java
public class DiscountCalculationService {
	public Money calculateDiscountAmounts(
		List<OrderLine> orderLines,
		List<Coupon> coupons,
		MemberGrade grade) {
		...
	}
}
```

이렇게 작성된 도메인 서비스는 애그리거트나 응용 서비스에서 사용될 수 있다.

```java
//애그리거트 예시
public class Order{
	public void calculateAmounts(DiscountCalculationService disCalSvc, MemberGrade grade) {
	...
	}
}
```

```java
//응용 서비스 예시
public class orderService {
	private DiscountCalculationService service;

	...
}
```

### 특정 기능이 응용 서비스인지 도메인 서비스인지 확인하는 방법

아래의 행위가 일어나면 도메인 서비스

- 애그리거트 상태를 변경
- 애그리거트의 상태 값을 계산

### 외부 시스템 연동과 도메인 서비스

시스템 간 연동은 HTTP API 호출로 이루어질 수 있지만, 도메인 입장에서는 도메인 로직으로 볼 수 있다. 도메인 관점에서 인터페이스를 작성한다.

도메인 서비스의 개수가 많거나 명시적으로 구분하고 싶다면 아래와 같이 하위 패키지로 구분한다.

- domain.model
- domain.service
- domain.repository

### 도메인 서비스의 인터페이스와 클래스

도메인 서비스의 로직이 고정되어 있지 않은 경우 도메인 서비스 자체를 인터페이스로 구현하고 이를 구현한 클래스를 둘 수 있다.
