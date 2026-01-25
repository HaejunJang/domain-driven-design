# 8장 애그리거트 트랜잭션 관리

### 애그리거트와 트랜잭션

- 비관락(Pessimistic Lock) = 선점락
- 낙관락(Optimistic Lock) = 비관락

### 선점 잠금

JPA EntityManager는 LockModeType을 인자로 받는 find() 메서드를 제공한다.

```java
Order order = entityManager.find(
	Order.class, orderNo, LockModeType.PESSIMISTIC_WRITE);
```

혹은

```java
public interface MemberRepository extends Repository<Member, MemberId> {
	@Lock(LockModeType.PESSIMISTIC_WRITE)
	@Query("select m from Member m where m.id = :id")
	Optional<Member> findByIdForUpdate(@Param("id") MemberId memberId);
```

### 선점 잠금과 교착 상태

1. 스레드 1: A애그리거트에 대한 선점 잠금 구함
2. 스레드 2: B애그리거트에 대한 선점 잠금 구함
3. 스레드 1: B애그리거트에 대한 선점 잠금 시도
4. 스레드 2: A애그리거트에 대한 선점 잠금 시도

위와 같은 순서에서는 스레드 1은 영원히 B 애그리거트에 대한 선점 잠금을 하지 못하며 스레드 2는 A 애그리거트에 대한 선점 잠금을 하지 못하는 **교착상태**에 빠진다

이를 해결하기 위해서는 최대 대기시간을 지정해야 한다.

최대 대기시간을 지정하는 방법은 힌트를 제공해야 한다.

```java
Map<String, Object> hints = new HashMap<>();
hints.put("javax.persistence.lock.timeout",2000);
Order order = entityManager.find(
	Order.class, orderNo, LockModeType.PESSIMISTIC_WRITE, hints);
```

```java
public interface MemberRepository extends Repository<Member, MemberId> {
@Lock(LockModeType.PESSIMISTIC_WRITE)
@QueryHints({
	@QueryHint(name = "javax.persistence.lock.timeout", value = "2000")
})
@Query("select m from Member m where m.id = :id")
Optional<Member> findByIdForUpdate(@Param("id") MemberId memberId);
```

### 비선점 잠금

- 비선점 잠금은 동시에 접근하는 것을 막는 대신 변경한 데이터를 실제 DBMS에 반영하는 시점에 변경 가능 여부를 확인하는 방식이다.
- 비선점 잠금을 구현하려면 애그리거트에 버전으로 사용할 숫자 타입 프로퍼티를 사용한다.
- 애그리거트를 수정할 때마다 버전 값이 1씩 증가한다.

```java
@Entity
@Access(AccessType.FIELD)
public class Order {
	@EmbeddedId
	private OrderNo number;

	@Version
	private long version;
	...
```

비선점 방식을 여러 트랜잭션으로 확장하려면 애그리거트 버전 정보를 응용 서비스에 전달해야 한다.

응용 서비스는 전달받은 버전 값을 이용해서 애그리거트 버전과 일치하는지 확인하고, 일치하는 경우에만 기능을 수행한다.

```java
public class StartShippingRequest {
	private String orderNumber;
	private long version;
	...
}
```

응용서비스 로직

```java
public StartShippingService {
	@PreAuthorize("hasRole('ADMIN')")
	@Transactional
	public void startShipping(StartShippingRequest req) {
		Order order = orderRepository.findById(new OrderNo(req.getOrderNumber()));
		//version 확인
		if (!order.matchVersion(req.getVersion())) {
			throw new VersionConflictException();
		}
		...
	}
}
```

### 강제 버전 증가

- 애그리거트에 애그리거트 루트 외에 다른 엔티티값만 변경되면 버전값이 변경되지 않는다. 따라서 애그리거트 내에 어떤 구성 요소의 상태가 바뀌면 루트 애그리거트의 버전 값이 증가해야 비선점 잠금이 올바르게 동작한다.
- JPA 이런 문제를 해결하도록 EntityManager#find() 메서드로 엔티티를 구할 때 강제로 버전 값을 증가시키는 잠금 모드를 지원한다.

```java
public interface MemberRepsitory extends Repository<Order, OrderId> {
	@Lock(LockModType.OPTIMISTIC_FORCE_INCREMENT)
	Optional<Order> findById(OrderId orderId);
}
```
