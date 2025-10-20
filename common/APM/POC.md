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
| **Spring+JPA연동 편의성** | 높음 (Agent 자동 트레이싱) | Micrometer + Actuator 설정 필요 |
| **주요 기능** | 트랜잭션 추적, SQL/에러 분석, 호출 관계 맵 | CPU/Memory, 요청 수, 응답시간, 에러율 등 메트릭 |
| **시각화** | 자체 Web UI | Grafana 대시보드 |
| **도입 난이도** | 쉬움 (Agent만 추가) | Prometheus + Grafana 세팅 필요 |
| **운영 리소스** | 중간 (Collector/Web 필요) | 높음 (Prometheus 서버 + Grafana 서버) |
| **비용** | 무료 (오픈소스) | 무료 (오픈소스) |
| **트랜잭션 단위 분석** |  가능 (API 호출 흐름 파악) | 불가 (집계 중심) |
| **커스터마이징** | 제한적 (UI 고정) |  높음 (대시보드 커스터마이징 가능) |
| **공식 근거** | [Pinpoint GitHub](https://github.com/pinpoint-apm/pinpoint)<br>[Pinpoint 문서](https://pinpoint-apm.gitbook.io/pinpoint/) | [Prometheus 공식문서](https://prometheus.io/docs/introduction/overview/)<br>[Spring Boot Metrics 가이드](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.prometheus)<br>[Grafana Docs](https://grafana.com/docs/) |



### 결론 및 의사결정

Pinpoint 도입 - Spring Boot + JPA 환경에서 트랜잭션 단위로 API 흐름을 추적 가능하며, 설정이 간단하고 네이버 오픈소스라 한국 자료가 많음
향후 계획 - Prometheus + Grafana는 필요 할 시 추후 서버 리소스 모니터링용으로 도입



