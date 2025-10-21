# poc

---

### 문제의식


운영 중인 Spring Boot + JPA 서버에서 성능 저하나 오류 발생 시,원인 파악이 힘듬

로그만으로는 호출 경로와 병목 지점을 찾는 데 시간이 많이 걸렸고, 실시간 모니터링 도구가 부재함

→ APM 도입을 하여 성능 데이터를 시각화하고 대응 시간을 단축하고자 함.

---

### 특징 및 요구사항

서비스 규모 - 비교적 가벼운 API 중심 서비스

기술 스택 - Spring Boot, Java 17, JPA

인프라 - Naver Cloud, Docker

비용 - 최대한 저렴하게

난이도 - APM도입이 처음이라 어렵지 않았으면 좋겠음

요구사항 - API 응답시간, 오류, 흐름 시각화


### 어떻게 해결할까?

APM 시스템을 도입하여,
서비스의 응답시간 / 오류율 / 호출 흐름 을 실시간으로 추적하고 시각화

#### 후보 비교

| 구분 | **Pinpoint** | **Prometheus + Grafana** |
|------|---------------|---------------------------|
| **개요** | Java 기반 APM (네이버 오픈소스) | 모니터링 툴 조합 |
| **핵심** | 분산 트랜잭션 추적 (APM) | 시계열 기반 모니터링 (Metrics) |
| **주요 기능** | 트랜잭션 추적, SQL/에러 분석, 호출 관계 맵 | CPU/Memory, 요청 수, 응답시간, 에러율 등 메트릭 |
| **시각화** | 자체 Web UI | Grafana 대시보드 |
| **도입 난이도** | 쉬움 (Agent만 추가) | Prometheus + Grafana 세팅 필요(난이도 낮음) |
| **데이터 단위** | 개별 트랜잭션(요청 단위) | 집계된 지표(Time Series) |
| **목표** | 호출 흐름 및 병목 추적 | 시스템 상태 지표화 |
| **비용** | 무료 (오픈소스) | 무료 (오픈소스) |
| **커스터마이징** | 제한적 (UI 고정) |  높음 (커스터마이징 가능) |
| **공식 근거** | [Pinpoint GitHub](https://github.com/pinpoint-apm/pinpoint)<br>[Pinpoint 문서](https://pinpoint-apm.gitbook.io/pinpoint/) | [Prometheus 공식문서](https://prometheus.io/docs/introduction/overview/)<br>[Spring Boot Metrics 가이드](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.prometheus)<br>[Grafana Docs](https://grafana.com/docs/) |


---


### 의사결정


#### 서비스 구조 기준 판단

| 기준 | 설명 | 추천 |
|------|------|------|
| **단일 서버 + DB 구조** | 내부 호출이 단순하면 트랜잭션 이점 작음 | Prometheus+Grafana |
| **응답시간 평균 / 에러율 중심 모니터링** | 집계 기반 메트릭이면 충분 |  Prometheus+Grafana |
| **대시보드 커스터마이징** | Grafana가 훨씬 유연함 | Prometheus+Grafana |
| **장애 원인 정확한 추적** | “어떤 요청 → 어떤 내부 호출”로 분석 가능 | Pinpoint |

#### 결론

지금 구조에서는 Prometheus + Grafana 조합이 실용적.

Pinpoint는 단순히 트랜잭션이 많을 때만 유용한 게 아니라 장애 시 문제 원인 추적 속도를 상당히 줄이는 도구라 메리트가 없지는 않다고 생각하지만 지금처럼 소규모서비스 일때는 메리트가 적다고 생각.

따라서 현재는 Prometheus 중심으로 가되, 추후 서비스 복잡도가 증가하면 pinpoint를 병행 도입하여 확장하는게 좋다고 생각.




