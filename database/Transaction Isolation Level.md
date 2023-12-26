# Transaction Isolation Level

## Isolation Level

### READ UNCOMMITED

커밋되지 않은 트랜잭션의 데이터 변경사항을 다른 트랜잭션에서 조회하는 것을 허용

- 성능이 가장 빠름
- 데이터 부정합 문제 발생 확률 높음

데이터 어림잡아 집계하는 등의 연산에 사용하면 좋음

### READ COMMITED

커밋이 완료된 트랜잭션의 변경사항만 다른 트랜잭션에서 조회하는 것을 허용

### REPEATABLE READ

특정 행 조회 시 항상 같은 데이터를 응답하는 것을 보장

(**MySQL의 InnoDB 엔진 기본 격리 수준**)

### SERIALIZABLE

특정 트랜잭션이 사용 중인 테이블의 모든 행을 다른 트랜잭션이 접근할 수 없도록 함

- 가장 높은 데이터 정합성
- 성능이 가장 느림

## Problems

### Dirty Read

하나의 트랜잭션에서 다른 트랜잭션이 커밋하지 않은 데이터를 읽는 현상

### Non-Repeatable Read

하나의 트랜잭션에서 같은 데이터를 여러 번 읽을 때 결과가 다른 현상

### Phantom Read

하나의 트랜잭션에서 같은 조건으로 여러 번 읽을 때 **없던 데이터가 생기는 현상**

## Summary

|                 | Dirty Read | Non-Repeatable Read | Phantom Read |
|-----------------|------------|---------------------|--------------|
| READ UNCOMMITED | O          | O                   | O            |
| READ COMMITED   | X          | O                   | O            |
| REPEATABLE READ | X          | X                   | O            |
| SERIALIZABLE    | X          | X                   | X            |

## References

[데이터베이스 트랜잭션 격리 수준과 격리 수준에 따른 문제점](https://hudi.blog/transaction-isolation-level/)