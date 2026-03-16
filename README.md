# _DOMAIN_DRIVEN_DESIGN_

![Start Date](https://img.shields.io/badge/start%20date%20-26.01.14-green?style=flat-square&logo=start)
![GitHub last commit](https://img.shields.io/github/last-commit/HaejunJang/domain-driven-design?style=flat-square)
![Language](https://img.shields.io/badge/language-Java-orange?style=flat-square&logo=java)

## 교재
<img width="200"  height="300" alt="domain-driven" src="https://github.com/user-attachments/assets/c7ec78d3-abc9-44c2-ac07-a4c445cfef77" />

- 도메인 주도 개발 시작하기(최범균 저자) 책을 읽고 기록하는 저장소 입니다.

## 공부 목표

해커톤에서 DDD 패키지 구조를 적용하며 도메인 중심 설계를 시도했습니다.

그러나 구조를 따르는 수준에 머물렀고, 다음과 한계를 경험했습니다.

- Aggregate 경계 설정의 근거 부족과 모호함
- 도메인 로직이 application 계층에 머무는 문제
- 트랜잭션 전파 옵션에 대한 이해 부족
- 이벤트 이해에 대한 부족으로 도메인간 강결합 문제

이 저장소는 이러한 부족함을 보완하고, 도메인 설계를 다시 정리하기 위한 기록입니다.
 
## 향후 계획
### ToDoc DDD 리팩토링

- Aggregate 재설계
- 도메인 로직을 application -> Domain으로 이동
- 트랜잭션 경계 및 전파 옵션 실험
- 이벤트를 도입하여 시스템 간 강결합 문제 해결

### 오버엔지니어링에 대한 고민 정리
해커톤 당시에는 개발 기간과 팀 이해도를 고려하여 JPA Entity를 그대로 DDD Entity처럼 사용하는 방식을 선택했습니다.

이는 현실적인 선택이었지만, "왜 그렇게 선택했는지"에 대한 깊은 고민은 부족했습니다.

다음 질문에 대해 리팩토링을 진행하며 답을 찾아보고자 합니다.
- DDD Entity와 JPA Entity를 언제 분리하는가?
- 헥사고날 아키텍처는 언제 필요한가?

### CQRS 적용

- Command / Query 모델 분리
- 조회 모델 최적화 설계
- 단일 DB유지 VS Read 모델 분리
- MongoDB 등 NoSQL 도입의 필요성 검토
