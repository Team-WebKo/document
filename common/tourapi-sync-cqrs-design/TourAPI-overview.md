# TourAPI란?
+ `지역코드`, `관광지 정보`, `문화행사`, `음식`, `숙박`, `쇼핑` 등 관광 관련 데이터 제공

## 활용매뉴얼
+ 서비스 Key - 인증 및 권한
+ HTTP, HTTPS
+ REST(GET) - 인터페이스 표준
+ XML, JSON - 교환데이터 표준
+ Request-Response - 메시지 교환 유형
+ 메시지 로깅 
  + 성공 - Header
  + 실패 - Header, Body
+ 데이터 갱신 주기 - 일 1회 (정확한 시간은?)

## 주요 엔드포인트
+ `/areaCode1` : 지역 및 시군구 코드 조회
+ `/areaBasedList1` : 지역 기반의 관광지 정보 목록 조회

## 요청 방식
+ REST 기반의 `HTTP GET`방식

### 필수 파라미터
+ `MobilesOS` : 사용 OS (ETC로 설정 가능)
+ `MobileApp` : 서비스명
+ `serviceKey` : 발급받은 인증키
+ `areaCode` : 지역 코드
+ `sigunguCode` : 시군구 코드
+ `numOfRows` : 한 페이지당 몇 개의 결과
+ `pageNo` : 페이지 번호
+ `_type` : `json` or `xml`
```text
GET https://apis.data.go.kr/B551011/KorService1/areaBasedList1
?MobileOS=ETC
&MobileApp=hipstour
&areaCode=1
&numOfRows=10
&pageNo=1
&_type=json
&serviceKey=...
```

### 응답 예시 - JSON
```json
{
  "response": {
    "body": {
      "items": {
        "item": [
          {
            "title": "서울숲",
            "contentid": "125266",
            "addr1": "서울특별시 성동구",
            "mapx": "127.0375",
            "mapy": "37.5442",
            "firstimage": "http://imageurl",
            ...
          }
        ]
      },
      "numOfRows": 10,
      "pageNo": 1,
      "totalCount": 3023
    }
  }
}
```

---

## 파라미터의 조합에 따른 표현
+ `지역별`관광정보 : 지역정보(필수) > 타입정보(선택) > 분류정보(선택) > 관광정보목록
  + `areaBasedList2`
+ `타입별`관광정보 : 타입정보(필수) > 지역정보(선택) > 분류정보(선택) > 관광정보목록
+ `분류별`관광정보 : 타입정보(필수) > 분류정보(선택) > 지역정보(선택) > 관광정보목록
+ `키워드`검색 : 지역정보(선택) > 타입정보(선택) > 분류정보(선택) > 검색된 정보 목록
  + `serarchKeyword2`
+ `내 주변`관광정보 : 타입정보(선택) > 관광정보목록
  + `locationBasedList2`
+ `날짜별`행사 & 축제 : 지역정보(선택) > 행사, 공연, 축제 목록
  + `searchFestival2`

### 공통 정보 및 상세 정보 조회
+ `detailCommon2` : 공통정보조회 (상세정보1)
+ `detailIntro2` : 소개정보조회 (상세정보2)
+ `detailInfo2` : 반복정보조회 (상세정보3)
+ `detailImage2` : 이미지정보조회 (상세정보4)

### 동기화 목록 조회
+ `areaBasedSyncList2` : 국문관광정보 동기화목록조회

[참고] https://www.data.go.kr/data/15101578/openapi.do#tab_layer_detail_function