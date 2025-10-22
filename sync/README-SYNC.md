# Sync Module
## 개요
`Sync` 모듈은 Hips-Tour 서비시의 데이터베이스와 외부 API(한국관광공사 TourAPI)의 장소 데이터를 동기화 하는 역할을 담당합니다.

주기적인 데이터 동기화를 통해 Hips-Tour 사용자가 항상 최신 정보를 조회할 수 있도록 보장합니다.

---

## 주요 기능
`Sync` 모듈은 두 가지 핵심 기능을 제공합니다.

### 전체 데이터 로드 - Full Data Load
- 담당 서비스
    - `LoadService`
- 설명
    - TourAPI가 제공하는 모든 장소 데이터를 가져와 Hips-Tour 데이터베이스에 저장합니다
    - 주로 서비스 초기 구축 시점에 사용되며, 전체 데이터를 가져오기에 많은 시간이 소요될 수 있습니다.
- 실행 방법
    - 수동 실행
    - `POST /api/tour/data/load-all` API를 호출하여 실행합니다.

### 증분 동기화 - Incremental Sync
- 담당 서비스
    - `SyncService`
- 설명
    - 마지막 동기화 시점(적재 완료시 시점이 최초 동기화 시점으로 작동합니다) TourAPI에서 변경(추가 또는 수정)된 장소 데이터만 선별하여 데이터베이스에 반영합니다.
    - 이를 통해 시스템 부하를 최소화하고 효율적으로 최신 상태를 유지합니다.
- 실행 방법
    - 자동 실행
        - 스케줄러에 의해 주기적으로 자동 실행됩니다. (기본 설정 : 매일 새벽 1시)
    - 수동 실행
        - `POST /api/tour/data/sync-updated` API를 호출하여 실행 가능합니다.

---

## 설정
`sync/src/main/resources/application.yml` 파일에서 `Sync`모듈의 주요 동작을 설정합니다.

### 설명
- `tourapi`: TourAPI 연동에 필요한 기본 정보(URL, 서비스 키 등)를 설정합니다.
- `sync`: 동기화 작업에 필요한 정책을 설정합니다.
    - `schedule.cron`: 증분 동기화가 실행될 주기를 설정합니다.
    - `load.area-codes`: 전체 데이터 Load 시 대상이 될 지역 코드를 정의합니다.
    - `daily-api-call-limit`: TourAPI의 일일 호출량 제한을 초과하지 않도록 제어합니다.