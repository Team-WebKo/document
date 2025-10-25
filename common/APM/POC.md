# poc

---

### 문제의식


운영 중인 Spring Boot + JPA 서버에서 응답 지연 및 오류 발생 시, 원인 파악이 어렵다는 문제가 있음

- 현재는 로그 에만 의존하고 있으며, 요청 흐름이나 병목 구간을  한눈에 파악하기 어려움

- 실시간으로 응답시간, 오류율등을 관찰할 수 있는 모니터링 도구가 부재함

→ APM 도입을 하여 성능 데이터를 시각화하고 대응 시간을 단축하고자 함.

---

### 특징 및 요구사항

| 항목 | 설명 |
|------|------|
| **서비스 규모** | 비교적 가벼운 API 중심 서비스 |
| **기술 스택** | Spring Boot, Java 17, JPA |
| **인프라 환경** | Naver Cloud, Docker 일부 사용 |
| **비용 제약** | 유료 솔루션 사용 불가 (오픈소스 선호) |
| **도입 난이도** | APM 도입 경험이 없으므로 설정이 간단해야 함 |
| **핵심 요구사항** | - API별 응답시간 측정<br>- 오류율 추적 및 알림<br>- 요청 흐름(Call Flow) 시각화 |



### 어떻게 해결할까?

APM 시스템을 도입하여,
서비스의 응답시간 / 오류율 / 호출 흐름 을 실시간으로 수집, 분석, 시각화
이를 위해 두 가지 오픈소스 후보 기술을 비교 검토

#### 후보 비교

| 구분 | **Pinpoint** | **Prometheus + Grafana** |
|------|---------------|---------------------------|
| **개요** | Java 기반 APM (네이버 오픈소스) | 모니터링 및 시각화 도구 조합 |
| **핵심** | 분산 트랜잭션 추적 | 시계열 기반 모니터링 (Metrics) |
| **주요 기능** | 트랜잭션 추적, SQL/에러 분석, 호출 관계 맵 | CPU/Memory, 요청 수, 응답시간, 에러율 등 메트릭 |
| **시각화** | 자체 Web UI (고정 형태) | Grafana 대시보드 (자유로운 커스터마이징)|
| **도입 난이도** | 쉬움 (Agent만 추가) | Prometheus + Grafana 세팅 필요(난이도 낮음) |
| **데이터 단위** | 개별 트랜잭션(요청 단위) | 시계열(Time Series) 단위 |
| **목표** | 요청 흐름 및 병목 추적 | 시스템 상태 모니터링 |
| **비용** | 무료 (오픈소스) | 무료 (오픈소스) |
| **공식 근거** | [Pinpoint GitHub](https://github.com/pinpoint-apm/pinpoint)<br>[Pinpoint 문서](https://pinpoint-apm.gitbook.io/pinpoint/) | [Prometheus 공식문서](https://prometheus.io/docs/introduction/overview/)<br>[Spring Boot Metrics 가이드](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.export.prometheus)<br>[Grafana Docs](https://grafana.com/docs/) |


---


### 의사결정


#### 서비스 구조 기준 판단

| 기준 | 설명 | 추천 |
|------|------|------|
| **단일 서버 + DB 구조** | 내부 호출이 단순하므로 트랜잭션 추적 이점이 크지 않음 | Prometheus+Grafana |
| **응답시간 평균 / 에러율 중심 모니터링** | 집계 기반 메트릭이면 충분 |  Prometheus+Grafana |
| **대시보드 커스터마이징** | 시각화 레이아웃과 알림 설정을 자유롭게 구성 가능 | Prometheus+Grafana |
| **장애 원인 정확한 추적** | Controller → Service → Repository 등 호출 관계를 시각적으로 분석 가능 | Pinpoint |

#### 결론

현재 서비스는 단일 서버 기반의 비교적 단순한 구조로,  
복잡한 내부 호출 트랜잭션을 추적해야 할 필요성이 크지 않음.  

따라서 초기 도입은 Prometheus + Grafana 조합이 가장 실용적이라고 판단됨.  
- 단순한 설정으로도 API 응답시간, 오류율, 요청 수를 빠르게 관찰 가능  
- Grafana를 통해 자유로운 대시보드 구성 및 알림 설정 가능  
- 운영 리소스와 유지보수 부담이 적음

단, Pinpoint는 트랜잭션 추적형 APM으로서 장애 원인 분석 속도를 획기적으로 단축할 수 있는 장점이 있으므로,  
향후 서비스 복잡도가 증가하거나 구조가 확장될 경우 Pinpoint를 병행 도입하는 것이 좋다고 생각




