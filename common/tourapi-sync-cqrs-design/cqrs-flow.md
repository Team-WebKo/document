# CQRS 기반 관광지 정보 처리 흐름

## CQRS란?
+ CQRS는 데이터의 변경(Command)과 조회(Query) 책임을 명확히 분리하는 아키텍처 패턴입니다
  + 변경(Command) : 관광지 정보 저장, 갱신, 삭제 등
  + 조회(Query) : 사용자에게 관광지 리스트, 상세 정보 제공

---

## Command 흐름 - Write
```declarative
[ TourAPI 호출 ]
     ↓
[ SyncPlaceCommand 생성 ]
     ↓
[ SyncPlaceCommandHandler 실행 ]
     ↓
[ PlaceWriteRepository.saveAll() ]
     ↓
[ DB 저장 완료 ]
```

## Query 흐름 - Read
```declarative
[ 사용자 요청 ]
      ↓
[ GetPlaceListQuery 생성 ]
      ↓
[ PlaceQueryHandler 실행 ]
      ↓
[ PlaceReadRepository.findAll() ]
      ↓
[ ViewModel or DTO 반환 ]
```