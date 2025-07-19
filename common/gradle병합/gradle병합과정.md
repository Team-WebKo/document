# Gradle 멀티모듈 프로젝트 병합

## 멀티모듈 공통 의존성 관리 필요

### 왜?

중복 제거: 대부분의 모듈에서 동일한 의존성(spring-boot-starter, lombok, junit 등)을 매번 반복 작성

중앙 집중 관리: 버전 충돌이나 의존성 누락 문제를 예방하기 위해 라이브러리를 루트에서 한 번에 관리하고 싶음

모듈별 유연함 : util 모듈처럼 의존성이 거의 필요 없는 모듈도 있어, 공통 설정을 강제하기보단 선택적으로 적용하고 싶음

Gradle 병합 필요성: 다양한 모듈들이 공통된 설정과 dependency 버전을 공유하지 않으면 관리가 어려워지고, 추후 모듈이 많아질수록 확장성과 유지보수성이 떨어지기 때문


---

## 시도 - 루트 build.gradle.kts에 dependencies사용

```kotlin
// 루트 build.gradle.kts
...

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web") //에러 발생
}
```


### 문제점


루트 프로젝트 build.gradle.kts에서는 `dependencies {}` 블록이 허용되지 않음
루트 프로젝트는 빌드 로직을 직접 실행하지 않기 때문에 dependencies {} 블록을 넣으면 동작하지 않음


(참고자료 - https://docs.gradle.org/current/userguide/multi_project_builds.html)


### 알게 된 점


 루트 build.gradle.kts는 공통 설정 위치이긴 하지만, 실제로 의존성을 선언할 수는 없다.


---


## 시도 - gradle/config.gradle.kts 사용

gradle/config.gradle.kts 라는 별도 파일을 만들어서, 각 모듈에서 apply(from = "../gradle/config.gradle.kts") 로 불러오는 방식


### 문제점:


DSL 혼용 문제 : `build.gradle.kts` (Kotlin DSL)와 config.gradle.kts, apply(from = ...)간의 충돌문제 발생


### 알게 된 점:


Kotlin DSL을 쓰는 프로젝트에서는 DSL 혼용을 지양해야 하며, apply(from = ...) 방식은 유지보수에 적합하지 않다.

---

## 해결 - buildSrc + 커스텀Plugin 도입


Gradle이 자동으로 인식하는 buildSrc 디렉토리를 활용하여 공통 Plugin을 만들고, 이를 각 모듈에서 plugins { id("...") } 로 불러오는 구조로 변경
공식적으로 Gradle에서 권장

(참고자료 - https://docs.gradle.org/current/userguide/sharing_build_logic_between_subprojects.html)


### 알게 된 점

공통 설정을 plugin으로 만들면, 모듈 간 정책 일관성을 유지하면서도 선택적으로 적용 가능하고, 새로 buildSrc라는 방식에 대해 알게 되었다.


### 추루 계획

지금은 빠르게 Convention을 만들기 위해 하드코딩했지만, 추후에는 `libs.versions.toml`을 활용해 dependency 버전 중앙 관리 할 계획이다.

---





### 참고자료 정리
https://docs.gradle.org/current/userguide/organizing_gradle_projects.html
https://docs.gradle.org/current/userguide/multi_project_builds.html
https://docs.gradle.org/current/userguide/sharing_build_logic_between_subprojects.html

