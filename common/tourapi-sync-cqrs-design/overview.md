# TourAPI 관광지 동기화 개요 - CQRS 기반

## 목적
+ 외부 TourAPI로부터 제공되는 관광지 데이터를 내부 시스템에 동기화하여 저장한다
+ 안정적인 관광 정보 제공 및 검색 성능 향상 기대

## 시스템 구성 및 주요 기능
+ 외부 API 연동 - TourAPI
  + 지역코드, 관광지 목록 등을 정기적으로 호출
  + JSON 기반의 응답 파싱 후 데이터 저장

+ 동기화 처리
  + 관광지 동기화 커맨드 실행
  + Write 모델(Repository)을 통해 DB에 저장

+ 데이터 조회
  + 사용자 요청 시 Read 모델을 통해 관광지 정보 제공
  + CQRS 기반 분리된 Query 핸들러

## 아키텍처 흐름 - CQRS 기반
```text
[ TourAPI ]
     ↓
[ ExternalApiClient ]
     ↓
[ SyncCommand (ex. SavePlaceCommand) ]
     ↓
[ CommandHandler ]
     ↓
[ Write Repository ]
     ↓
  [ DB ]
     ↑
[ Read Repository ] ← [ Query Handler ] ← 사용자 요청
```

## 기술
+ Java / Spring Boot
+ CQRS 아키텍처
+ REST Client

## 책임 분리
| 계층 | 역할 |
| --- | --- |
| API Client | 외부 TourAPI 요청 및 응답 파싱 |
| Command | Write 처리, DB 저장 명령 |
| Query | 사용자 요청 처리 및 조회 |
| Repository | Read/Write 저장소 구분 |

## 추가 고려 사항
+ 동기화 상태 관리
+ 중복 데이터 필터링 또는 갱신

---

## 동기화 요청 시퀀스 다이어그램
```text
사용자            Hip's Tour             TourAPI                  TourAPI DB             Hip's Tour DB

   ───동기화 요청 클릭───▶                   
   
   ───동기화 API 호출───▶
  
                      ─────외부 API 요청────▶ 
                  
                                           ─Tour API가 TourAPI DB 조회─▶ 
                                            
                                           ◀─────────JSON 응답─────────
                  
                      ◀─────파싱 및 매핑─────
                      
                      ─────────────────────────데이터를 Hip's Tour DB에 저장────────────────────▶
                                                                                              ▼ INSERT 성공
                  
   ◀───저장 완료 응답───
```
