# Setting GitLab CI/CD

## CI/CD란

* CI -> Continuous Integration
    - build, test를 실시하는 프로세스로 이러한 통합 프로세스를 상시로 실시해주는 것

* CD -> Continuous Delivery / Deploy
    - 소프트웨어가 항상 신뢰 가능한 수준에서 배포될 수 있도록 지속적으로 관리하는 것
    - Continuous Delivery: 적용한 변경사항이 버그테스트를 거쳐 저장소에 자동 업로드 되는 것
    - Continuous Deploy: 변경사항을 저장소에서 고객이 사용가능한 운영환경까지 자동으로 릴리즈 해주는 것

### CI/CD가 필요한 이유

* CI
    - 각 개발자가 작성한 코드를 마지막에 통합하려고 하는 것은 어려움

* CD
    - 짧은 주기로 소프트웨어를 개발하는 경우, 소프트웨어가 신뢰 가능한 수준으로 배포되는 것을 보증함

## GITLAB CI/CI

### 소개

* gitlab이 자체적으로 제공하는 CI/CD 툴

### 사용방법

* 최상위 디렉토리에 .gitlab-ci.yml란 파일에 설정을 작성할 수 있음

* GitLab Runner라는 툴이 .gitlab-ci.yml 설정 파일을 읽고 ci/cd를 수행함

* 각각의 script를 job이라고 하며 일련의 job들을 pipeline이라고 함
