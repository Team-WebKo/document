# TourAPI란?
+ `지역코드`, `관광지 정보`, `문화행사`, `음식`, `숙박`, `쇼핑` 등 관광 관련 데이터 제공

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
[참고] https://www.data.go.kr/data/15101578/openapi.do#tab_layer_detail_function