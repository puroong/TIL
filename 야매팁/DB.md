# DB 야매 팁

* 조인 테이블은 왠만하면 외래키 조합을 기본키로 사용하자
    - auto incrementing 하는 정수를 기본키로 사용하다가 중복된 열이 삽입됐음에도 에러가 발생하지 않아서 몰랐음
    - 이게 REST API 설계 할 때도 일관성 있고 좋음
        * ex.
            * 외래키 조합을 기본키로 사용하는 경우
            
            | consultingId | checklistId | value |
            | ------------ | ----------- | ----- |
            | 1            |   2         | 0     |
            
            1번 컨설팅의 2번 체크리스트를 수정하고 싶은 경우 아래와 같이 설계하면 되지만
            
            `PUT /consultings/1/checklist/2`

            * id를 기본키로 사용하는 경우

            | id | consultingId | checklistId | value |
            | -- | ------------ | ----------- | ----- |
            | 1  | 1            |   2         | 0     |
            
            `PUT /checklist/1`
            
            `PUT /consultings/1/checklist/1`
            
            등 여러 경우의 수가 나올 수 있다는 문제가 있음
            
            **무엇보다도 checklist id와 조인 테이블의 id가 혼용될 수 있음**
            
            예를 들어 `PUT checklist/1`로 설계한 경우, 열을 수정하기 위해 `조인 테이블의 id`가 필요함
            
            대개 조회 API로 이 id를 받으니 checklist 조회 API 응답 id로 조인 테이블의 id를 값을 줄텐데
            
            `checklist id`로 `조인 테이블 id` 값을 주게됨