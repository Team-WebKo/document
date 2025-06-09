# 동기화 로직 상세 설계
---
## 스케줄링 방식
+ 시간을 기반으로 정해진 작업을 자동으로 실행한다

### Spring Boot 스케줄링
```java
@Component
@RequiredArgsConstructor
public class TourDataScheduler {

  private final TourSyncService tourSyncService;

  // 매일 새벽 3시 정각 실행
  @Scheduled(cron = "0 0 3 * * *")
  public void scheduleSync() {
    tourSyncService.syncAllAreas();
  }
}
```
+ `cron`표현식으로 지정 - [cron이란?](#cron이란?)
+ 실행 주기 : 일, 시간, 분 단뒤 등 설정 가능

### 왜 필요한가?
+ 동기화의 자동화로 추가적 작업이 줄어든다
+ 관리자의 실수 방지

### 보완
+ 정해진 규칙에 대해서 실행한다
  + 추가적인 작업이 발생 할 수 있다 - 수동 요청도 가능하도록 설계

---

## 동기화를 언제 진행하는 것이 좋은가?
+ 사용자 트래픽이 적은 시간대를 예상 및 실제 사용량에 따라 조정
+ Tour API의 변경 주기가 있다면 일정 시간 간격을 두고 작업하는것도 가능?
+ push?

---

## 트랜잭션 중단 시 처리 방법
+ 문제 예시
  + 1 ~ 100 페이지 중 50페이지 저장 중 예외 발생 > 나머지 데이터 처리 방법

### 해결 전략
| 전략 | 설명 |
| --- | --- |
| 페이지 기반 전략 | pageNo 단위로 저장<br>-> 실패한 pageNo만 재시도 |
| 중간 로그 저장 | 마지막 성공 pageNo 기록<br>-> 실패 시 이어서 진행 |
| 배치 트랜잭션 관리 | 페이지 단위로 트랜잭션 처리<br>-> 전체 실패 방지 |

#### 페이지 기반 전략
```java
@Service
@RequiredArgsConstructor
public class TourSyncService {

    private final TourApiClient tourApiClient;
    private final PlaceCommandRepository placeCommandRepository;

    private final int PAGE_SIZE = 100;

    public void syncAllAreas() {
        for (int areaCode = 1; areaCode <= 39; areaCode++) {
            int pageNo = 1;
            boolean hasNext = true;

            while (hasNext) {
                try {
                    TourApiResponse response = tourApiClient.getPlaces(areaCode, pageNo, PAGE_SIZE);
                    List<Place> places = response.toEntities();
                    placeCommandRepository.saveAll(places);

                    pageNo++;
                    hasNext = response.hasNext(); // 응답 내 totalCount 비교
                } catch (Exception e) {
                    log.error("[TourSyncService] areaCode={}, pageNo={} 실패", areaCode, pageNo, e);
                    break; // 실패 시 멈추고 기록
                }
            }
        }
    }
}
```
+ 장점 : 단순, 안전
+ 단점 : 중간 실패 시 수동 조작 필요

#### 중간 로그 저장 전략
+ Entity
```java
@Entity
public class SyncLog {
    @Id @GeneratedValue
    private Long id;

    private int areaCode;
    private int lastSuccessPage; // 마지막 성공한 pageNo
    private LocalDateTime updatedAt;
}
```
+ 서비스 수정
```java
public void syncWithLog() {
    for (int areaCode = 1; areaCode <= 39; areaCode++) {
        int pageNo = syncLogRepository.findLastPage(areaCode).orElse(1);

        while (true) {
            try {
                TourApiResponse response = tourApiClient.getPlaces(areaCode, pageNo, PAGE_SIZE);
                List<Place> places = response.toEntities();
                placeCommandRepository.saveAll(places);

                syncLogRepository.saveOrUpdate(areaCode, pageNo);
                if (!response.hasNext()) break;
                pageNo++;
            } catch (Exception e) {
                log.warn("Sync failed. areaCode={}, pageNo={}", areaCode, pageNo, e);
                break; // 실패한 page부터 재시도 가능
            }
        }
    }
}
```
+ 장점 : 이어서 재시도 가능
+ 단점 : 로그 관리 필요

#### 배치 트랜잭션 관리 전략
```java
@Service
@RequiredArgsConstructor
public class TourBatchSyncService {

    private final TourSyncHelper syncHelper;

    public void sync() {
        for (int areaCode = 1; areaCode <= 39; areaCode++) {
            int pageNo = 1;
            while (true) {
                try {
                    syncHelper.syncOnePage(areaCode, pageNo);
                    pageNo++;
                } catch (Exception e) {
                    log.warn("페이지 {} 실패, 다음 area로 진행", pageNo);
                    break;
                }
            }
        }
    }
}

@Service
public class TourSyncHelper {

    private final TourApiClient tourApiClient;
    private final PlaceCommandRepository placeCommandRepository;

    @Transactional
    public void syncOnePage(int areaCode, int pageNo) {
        TourApiResponse response = tourApiClient.getPlaces(areaCode, pageNo, 100);
        List<Place> places = response.toEntities();
        placeCommandRepository.saveAll(places);
    }
}
```
+ 장점 : rollback 최소화
+ 단점 : 분리된 트랜잭션 관리 필요

---

## 서브 프로젝트 기준 및 이점
### 서브 프로젝트란?
+ Gradle에서 하나의 프로젝트를 여러 모듈로 분리하는 것
+ 각 모듈은 `:common`, `:api-client`, `:place-service`등으로 나눌 수 있다

### 나누는 이유는?
1. 책임 분리
   + API 호출 로직은 `:api-client`
   + 비즈니스 로직은 `:place-service`
2. 재사용성
   + 다른 서비스에서 동일 API를 활용할 경우, 모듈만 재사용 하면 된다
3. 테스트 용이성
   + 기능별로 단위 테스트 작성이 쉽다
4. 배포 유연성
   + 서브 모듈만 독립 배포 가능

### 도입 기준?
+ 공통 로직인가?
+ 독립적으로 개발 및 배포가 가능한가?
+ 기능 간 의존성이 얼마나 되는가?(높지는 않은가?)

---

## 중간 로그 저장 전략 - 상세 흐름
```text
1. 지역 반복 →
   - 전체 지역 코드 (1 ~ 39 등)를 순회
   - 각 지역(areaCode)에 대해 관광지 목록을 순차적으로 동기화

2. 마지막 page 조회 →
   - SyncLog 테이블에서 해당 지역(areaCode)의 마지막 성공한 pageNo 조회
   - 없으면 pageNo = 1부터 시작
   - 있으면 pageNo = (마지막 저장 page + 1)

3. API 요청 →
   - TourAPI에 해당 지역, page 번호, row 수를 포함한 HTTP GET 요청 전송
   - 예: GET /areaBasedList1?areaCode=2&pageNo=4&numOfRows=100
   - 외부 API에서 JSON 형식으로 응답 반환

4. 응답 저장 →
   - 응답 JSON을 DTO 객체로 파싱
   - DTO → Entity 변환 (예: Place 객체 리스트)
   - placeCommandRepository.saveAll(places) 등을 통해 DB에 저장

5. 성공 로그 기록 →
   - DB 저장이 성공하면 SyncLog 테이블에
     (areaCode, pageNo, updatedAt) 저장 또는 갱신
   - 다음 실행 시 이 페이지부터 다시 시작 가능

6. 다음 page or 실패 로그 기록 →
   - TourAPI 응답에 hasNext 또는 totalCount 기반으로 다음 페이지 존재 여부 판단
     - 존재함 → pageNo++ 후 ③ API 요청으로 다시 루프
     - 존재하지 않음 → 다음 지역(areaCode++)으로 이동
   - 만약 API 호출 실패 또는 저장 중 예외 발생 시:
     - SyncFailLog 테이블에 (areaCode, pageNo, 실패사유 등) 저장
     - 해당 지역은 중단하고 다음 지역으로 이동
```

---

## 결론
+ 중간 로그 저장 전략이 우세하다 판단
  + 데이터 정합성을 위한 전략 선정과정에서 페이지 저장 전략 아웃
  + 실패 지점 저장 > 이어서 재실행 가능 부분이 높게 평가됨
  + 복잡성은 증가하나, 로그 기준으로 재처리가 가능해지며, 다소 배치 트랜잭션보다 I/O 성능 저하 문제는 중요 단점으로 보기 어려움
  + 관리자 수동 복구 가능 > 운영 편의성 증가

## 정리
+ 관리자 요청 또는 스케줄러에 의해 동기화 시작
+ TourAPI에서 JSON으로 데이터 받아온다
+ DTO로 변환 > Entity 매핑 > DB 저장
+ 사용자 요청 시 Query 모델을 통해 DB에서 조회한다 (동기화 적용 시)
  + 동기화 미 적용 시 : 데일리 여행지로 사용자별로 쿼리 발생

---

## 추가 내용
### cron이란?

| 위치 | 의미 | 예시 값 |
| --- | --- | --- |
| 1 | 초 (0~59) | 0 |
| 2 | 분 (0~59) | 0 |
| 3 | 시 (0~23) | 0 |
| 4 | 일 (1~31) | 0 |
| 5 | 월 (1~12) | 0 |
| 6 | 요일 (0~6) | 0 |

+ 예시

| cron 표현식 | 의미 |
|----------|----|
| `0 0 3 * * *` | 매일 03:00 |
| `0 */10 * * * *` | 10분마다 |
| `0 0 1 * * MON` | 매주 월요일 01:30 |
| `0 0 3 1 * *` | 매월 1일 03:00 |