# 5장 스프링 데이터 JPA를 이용한 조회 기능

### CQRS

CQRS는 명령(`command`)모델과 조회(`Query`)모델을 분리하는 패턴이다.

- 명령 모델은 상태를 변경하는 기능으로 회원 가입, 암호 변경 처럼 상태(데이터)를 변경하는 기능을 구현할 때 사용
- 조회 모델은 데이터를 조회하는 기능으로 주문 목록, 주문 상세 처럼 데이터를 보여주는 기능을 구현할 때 사용

### 스펙

조회를 위해 다양한 검색 조건을 조합해야 할 때가 있다. 이 때 사용할 수 있는 것이 스펙(`Specification`)이다.

스펙은 애그리거트가 특정 조건을 충족하는지를 검사할 때 사용하는 인터페이스이다.

```java
public interface Speciation<T> {
	public boolean isSatisfiedBy(T agg);
}
```

- `isSatisfiedBy()` 메서드의 agg파라미터는 검사 대상이 되는 객체
- `isSatisfiedBy()`메서드는 검사 대상 객체가 조건을 충족하면 true, 아니면 false를 리턴한다.
- 스펙을 DAO에 사용하면 agg는 검색 결과로 리턴할 데이터 객체가 된다.
- JPA를 사용할 경우 이렇게 스펙을 구현하지 않는다. 모든 애그리거트 객체를 메모리에 보관하기도 어렵고 보관하더라도 조회 성능이 매우 떨어지기 떄문이다.

### 스프링 데이터 JPA를 이용한 스펙 구현

```java
public interface Specification<T> {
	Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder builder);
}
```

레포지토리 인터페이스 상속

```java
public interface CustomerRepository extends CrudRepository<Customer, Long>, JpaSpecificationExecutor<Customer> {
    List<Customer> findAll(Specification<Customer> spec);
    List<Customer> findAll(Specification<Customer> spec, Sort sort);
    List<Customer> findAll(Specification<Customer> spec, Pageable pageable);
}
```

스펙 생성 기능을 별도 클래스로 구현

```java
public class CustomerSpecs {

  public static Specification<Customer> isLongTermCustomer() {
    return (root, query, builder) -> {
      LocalDate date = LocalDate.now().minusYears(2);
      return builder.lessThan(root.get(Customer_.createdAt), date);
    };
  }

  public static Specification<Customer> hasSalesOfMoreThan(MonetaryAmount value) {
    return (root, query, builder) -> {
	    ...
    };
  }
}
```

### 리포지토리/DAO에서 스펙 사용하기

```java
public interface OrderSummaryDao extends Repository<OrderSummary, String> {
	List<OrderSummary> findAll(Specification<OrderSummary> spec);
}
```

```java
//스펙 객체를 생성하고
Specification<OrderSummary> spec = new OrdererIdSpec("user1");
//findAll() 메서드를 이용해서 검색
List<OrderSummary> results = orderSummaryDao.findAll(spec);
```

### 스펙 조합

JPA가 제공하는 스펙 인터페이스는 스펙을 조합하도록 `and`와 `or`을 제공한다.

```java
Specification<OrderSummary> spec1 = OrderSummarySpecs.orderId("user1");
Specification<OrderSummary> spec2 = ...
Specification<OrderSummary> spec3 = spec1.and(spec2);
```

spec3는 spec1, 2가 모두 적용된 스펙을 가지게 된다.

null 가능성이 있는 스펙 객체의 조합은 `where`를 사용한다.

```java
Specification<OrderSummary> spec = Specification.where(createnullableSpec()).and(createOtherSpec());
```

### 페이지 정렬하기

스프링 데이터 JPA는 2가지 방법으로 정렬이 가능하다

1. 메서드 이름에 `OrderBy`를 사용해서 정렬 기준 지정
2. `Sort`를 인자로 전달

```java
public interface OrderSummaryDao extends Repository<OrderSummary, String> {
	//orderBy
	List<OrderSummary> findByOrdererIdOrderByNumberDesc(String ordererId);

	//sort
	List<OrderSummary> findByOrdererId(String ordererId, Sort sort);
}
```

### 페이징 처리하기

스프링 데이터 JPA는 `Pageable` 타입을 사용한다.

```java
public interface MemberDataDao extends Repository<MemberData, String> {
	List<MemberData> findByNameLike(String name, Pageable pageable);
	Page<MemberData> findByBlocked(boolean blocked, Pageable pageable);
}
```

```java
Sort sort = Sort.by(Sort.Direction.DESC, "name");
PageRequest pageReq = PageRequest.of(1,10,sort);
List<MemberData> user = memberDataDao.findByNameLike("이름%", pageReq);
```

### 동적 인스턴스 생성

JPQL의 new 키워드를 통해 객체를 동적으로 생성할 수 있다.

```java
public interface OrderSummaryDao extends Repository<OrderSummary, String> {
    @Query("""
            select new com.myshop.order.query.dto.OrderView(
                o.number, o.state, m.name, m.id, p.name
            )
            from Order o join o.orderLines ol, Member m, Product p
            where o.orderer.memberId.id = :ordererId
            and o.orderer.memberId.id = m.id
            and index(ol) = 0
            and ol.productId.id = p.id
            order by o.number.number desc
            """)
    List<OrderView> findOrderView(String ordererId);
}
```

- select 절에 `new`키워드를 볼 수 있다.
- `new`키워드 뒤에 생성할 인스턴스의 완전한 클래스 이름을 지정하고 괄호 안에 생성자에 인자로 전달할 값을 지정한다.
- 조회 전용 모델을 만드는 이유는 표현 영역을 통해 사용자에게 데이터를 보여주기 위함이다.

### 하이버네이트 @Subselect 사용

```java
@Entity
@Immutable
@Subselect(
        """
        select o.order_number as number,
        o.version,
        from purchase_order o inner join order_line ol
        where
        ol.line_idx = 0
        and ol.product_id = p.product_id"""
)
@Synchronize({"purchase_order", "order_line", "product"})
public class OrderSummary {
    @Id
    private String number;
    private long version;
    @Column(name = "orderer_id")
    private String ordererId;
    ...생략...
    protected OrderSummary() {
    }
}
```

하이버네이트는 JPA확장 기능으로 @Subselect를 제공한다.

- `@Subselect`를 사용하면 쿼리 실행 결과를 매핑할 테이블처럼 사용한다.
- 이때 조회된 테이블은 수정하면 안되므로 `@Immutable`을 사용한다.
- `@Immutable`은 해당 엔티티의 매핑 필드/프로퍼티가 변경되도 DB에 반영하지 않고 무시한다.

하지만 조회동시에 엔티티의 값이 수정된다면 이전 값을 조회하는 문제가 생길것이다 이를 해결하기 위해 `@Synchronize` 를 사용한다.

- `@Synchronize`를 사용하면 하이버네이트는 엔티티를 로딩하기 전에 지정한 테이블과 관련된 변경이 발생하면 플러시를 통해 변경 내역이 반영된다.
