#  JPA 테스트 방법

![image](https://github.com/user-attachments/assets/ab235a3e-9e99-4a97-a077-51afa68c630f)
![image](https://github.com/user-attachments/assets/3bbc6682-1c9d-4a99-b089-cf9fe8d2f950)

위 샘플 코드의 경우, 일반적인 JPA 사용 패턴을 보여주고 있음. 이를 이용해서 JPA 관련 테스트 코드를 작성하는 가이드는 아래와 같음

![image](https://github.com/user-attachments/assets/c2554b1c-a1fe-4593-b5d9-51492cfa4669)

1. DataJpaTest : JPA 환경을 테스트하기 위하여, 필요한 빈(Repository) 및, Entity를 생성할 수 있도록 해준다.
2. @AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE) JPA 테스트 시, 별도의 Embedded Database를 사용하도록 할지 강제하는 설정. 이미, local 환경에서는 H2를 사용중이기 때문에, NONE으로 하여, 별도로 생성하지 않도록 한다.
3. @ActiveProfiles : JPA 테스트 시, 쿼리를 확인하는 등, 여러 DEBUG 요소가 추가되기에, 별도의 프로파일(test)를 사용하여 설정을 추가하였음으로, 가능하면 프로필을 항상 test로 맞추길 권장

![image](https://github.com/user-attachments/assets/978ec0e0-9aca-4e3b-a4e0-706368fa7fb9)

