# REST API란?
+ Representational State Transfer
+ 리소스 기반 구조를 가진 HTTP 통신 방식
+ REST는 특정 규칙을 지켜서 만든 API

## REST의 특징
| 요소        | 설명 |
|-----------| --- |
| 리소스(URL)  | 자원을 URI로 표현 |
| 메서드       | HTTP 동사로 행위를 구분<br>`GET`,`POST`, `PUT`, `DELETE` |
| Stateless | 서버는 클라이언트의 상태를 저장하지 않는다 |
| 계층 구조 | 클라이언트 ↔ 서버 사이의 다양한 계층 추가 가능 |

## REST API 사용 이유
+ 단순하고 직관적 - URL만 보고도 의미 파악 가능
+ HTTP 표준 사용 - 방화벽, 프록시, 브라우저 캐시 활용 용이
+ 유지보수 용이 - 엔트포인트 단위로 기능 분리 가능