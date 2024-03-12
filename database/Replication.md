# Replication

## 0. Intro

데이터베이스 사용 및 운영 시 가장 중요한 두 가지 요소는 확장성(Scalability)과 가용성(Availability)다.

이 두 요소를 위해 가장 일반적으로 사용되는 기술이 복제(Replication)이다.

## 1. Replication

복제(Replication)은 한 서버에서 다른 서버로 데이터가 동기화되는 것이다.

- 소스(Source) 서버: 원본 데이터를 가진 서버
- 레플리카(Replica) 서버: 복제된 데이터를 가지는 서버

![](https://velog.velcdn.com/images/minide/post/11e4a8b4-9b03-4fec-9ae0-5c7e6d530bee/image.png)

소스 서버에서 데이터 및 스키마에 대한 변경이 발생하고, 레플리카 서버는 이러한 변경 내역을 소스 서버로부터 전달받아 자신이 가지고 있는 데이터에 반영함으로써 소스 서버에 저장된 데이터와 동기화시킨다.

### 1.1. Replication Purpose

1. 스케일 아웃(Scale-out)

   서비스의 트래픽이 증가해 DB 서버의 부하가 높아지면, 복제를 통해 DB 서버를 한 대 이상 사용함으로써 쿼리를 분산시켜 서비스를 보다 안정적으로 운영할 수 있다.

2. 데이터 백업

   데이터가 삭제되는 경우에 대비하기 위해 DB 서버에 저장된 데이터를 주기적으로 백업하는 것이 필수적이다. 동일한 서버 내에 백업이 실행되는 경우 DBMS에서 실행 중인 쿼리들이 영향을 받을 수 있다. 이 같은 문제를 방지하기 위해 주로 복제를 사용해 레플리카 서버를 구축하고, 데이터 백업을 실행한다.

3. 데이터 분석

   분석용 쿼리는 대량의 데이터를 조회하는 경우가 많고, 집계 연산 등 복잡하고 무거운 쿼리가 대부분 사용되기 때문에 쿼리 실행 시 서버의 리소스를 많이 사용한다. 이로 인해 서비스에 사용되는 다른 쿼리들이 영향을 받을 수 있으므로 복제를 이용해 분석용 쿼리만 전용으로 실행될 수 있는 환경을 만드는 것이 좋다.

4. 데이터의 지리적 분산

   애플리케이션 서버와 DB 서버가 서로 떨어져 있는 경우 두 서버 간 통신 시간이 거리에 비례해 늘어나 서비스 응답 속도에도 영향을 끼치므로 떨어져 있는 DB 서버의 위치를 이동시키지 못할 때 애플리케이션 서버 위치에 레플리카 서버를 구축함으로써 응답 속도를 개선할 수 있다.


## 2. Replication Architecture

- 바이너리 로그(Binary Log): MySQL 서버에서 발생하는 모든 변경사항이 순서대로 기록해둔 파일
    - 이벤트(Event): 바이너리 로그에 기록된 변경 정보들
- 릴레이 로그(Relay Log): 레플리카 서버에서 소스 서버의 바이너리 로그를 읽어 들여 따로 로컬 디스크에 저장해둔 파일

**복제 동기화 처리 과정**

![](https://velog.velcdn.com/images/minide/post/1d3051f6-cf5a-4281-b7ab-e8bfc12e6b22/image.png)

1. 소스 서버의 바이너리 로그에 이벤트가 변경됨
2. 바이너리 로그 덤프 스레드가 해당 이벤트를 읽어 레플리카 서버로 전송
3. 레플리카 서버의 I/O 스레드는 변경 이벤트를 릴레이 로그에 저장
4. 레플리카 서버의 SQL 스레드가 변경 내용을 데이터 파일에 저장 → **레플리카 서버에 데이터 변경이 반영됨**

## 3. Replication Type

![](https://velog.velcdn.com/images/minide/post/37ad40a2-eec8-46a4-9d57-b3dde7239994/image.png)

레플리카 서버가 소스 서버의 바이너리 로그의 이벤트를 식별하는 방식에는 **바이너리 로그 파일 위치 기반 복제**와 **글로벌 트랜잭션 ID(GTID) 기반 복제** 두 가지가 있다.

### 3.1. Binary Log File Location Based Replication

![](https://velog.velcdn.com/images/minide/post/c53ccdc4-e049-40ec-b36f-8d8681d74065/image.png)

- 소스 서버에서만 유효한 식별 방법
    - 소스 서버의 문제가 생겨 다른 레플리카 서버가 소스 서버로 승격된 경우, 복제에 참여하는 다른 데이터베이스 서버들은 바이너리 로그 파일의 위치를 다시 찾아야 하기 때문에 복구에 시간이 걸림

⇒ **동일한 이벤트가 레플리카 서버에 소스 서버와 동일한 파일명의 동일한 위치에 저장된다는 보장이 없다.**

### 3.2. Global Transaction ID(GTID) Based Replication

![](https://velog.velcdn.com/images/minide/post/9c3ee762-6043-446b-a55c-2a66c93105b5/image.png)

- 복제에 참여한 모든 데이터베이스 서버들이 고유한 식별값을 가지고 있어 동일한 이벤트에 대해 동일한 GTID만 읽어오면 데이터 변경을 반영할 수 있음
- **MySQL 5.6** 부터 GTID 기반 복제를 기반 복제 방식으로 사용

## 4. Replication Data Format

바이너리 로그의 로깅 타입은 변경 이벤트들이 바이너리 로그에 어떤 형태로 저장되는지를 나타낸다.

레플리카 서버가 소스 서버의 바이너리 로그 이벤트를 내부적으로 가공하지 않고 가져온 그대로 실행해 자신의 데이터에 적용하기 때문에 바이너리 로그에 이벤트가 어떤 포맷으로 기록되는지는 중요한 부분이다.

### 4.1. Statement Based Binary Log Format

이벤트를 발생시킨 SQL문을 바이너리 로그에 기록하는 방식

**장점**

- 여러 개의 데이터를 수정하는 쿼리여도 바이너리 로그에 SQL문 하나만 기록되기 때문에 **저장 공간에 대한 부담 감소** 및 **빠른 처리 가능**

**단점**

- 비확정적 쿼리(실행할 때마다 결과값이 달라지는 쿼리)의 경우 **데이터 동기화 문제 발생 가능**

  ex. DELETE, UPDATE 쿼리에서의 ORDER BY절 없이 LIMIT 사용

- Row 기반 바이너리 로그 포맷보다 락을 더 많이 검
- 하나의 트랜잭션 내에서 각 쿼리가 실행되는 시점마다 데이터 스냅샷이 달라질 수 있기 때문에 트랜잭션 격리 수준 **REPEATABLE-READ 이상만 사용 가능**

⇒ **일관되지 않은 데이터가 저장**될 위험이 있음

### 4.2. Row Based Binary Log Format

변경된 값 자체가 바이너리 로그에 기록되는 방식

- **MySQL 5.1** 부터 도입, **MySQL 5.7.7** 부터 바이너리 로그 기본 포맷으로 지정

**장점**

- 모든 트랜잭션 격리 수준에서 사용 가능

**단점**

- 데이터를 많이 변겨하는 SQL문이 실행될 경우 바이너리 로그 파일 크기가 커질 수 있음

  → MySQL에서는 바이너리 로그 Row 이미지와 바이너리 로그 트랜잭션 압축 두 가지 방식의 **용량 최적화 방식 지원**

- 어떤 쿼리들이 넘어왔고 현재 어떤 쿼리가 실행 중인지 육안으로 확인 불가능

⇒ **데이터를 일관되게 저장**하는 가장 안전한 방식

### 4.3. Mixed Format

사용자가 두 가지 바이너리 로그 포맷을 혼합해서 사용하는 방식

MySQL 서버는 Mixed 포맷 설정 시 기본적으로 Statement 포맷을 사용하며 비확정적 쿼리의 경우 Row 포맷으로 변환되어 로그에 기록된다.

## 5. Replication Synchronization

### 5.1. Asynchronous Replication

소스 서버가 레플리카 서버에서 변경 이벤트가 정상적으로 전달됐는지 확인하지 않는 방식

![](https://velog.velcdn.com/images/minide/post/61ccd1af-ca8b-4a28-9757-ea0879b0ce8d/image.png)

성능은 빠르지만 동기화를 보장하지 않음

### 5.2. Semi-synchronous Replication

소스 서버는 레플리카 서버가 소스 서버로부터 전달받은 변경 이벤트를 릴레이 로그에 기록 후 응답을 보내면 그때 트랜잭션을 완전히 커밋

- MySQL 5.5 부터 사용

![](https://velog.velcdn.com/images/minide/post/3d8ff594-c034-4ad4-9d0a-93decafc58e0/image.png)

- ACK: 이벤트가 레플리카 서버에 **전송됐음에 대한 응답** (적용됐음에 대한 응답이 아님)

## 6. Replication Topology

### 6.1. Single Replica Replication

![](https://velog.velcdn.com/images/minide/post/3cf328a2-ec5f-4050-9f26-ef00dc86a3db/image.png)

- 가장 기본적인 형태
- 레플리카 서버를 예비 서버 및 데이터 백업용으로 활용

### 6.2. Multi Replica Replication

![](https://velog.velcdn.com/images/minide/post/167987f3-1b66-470c-8ecb-34337b6b0ca3/image.png)

- 싱글 레플리카 구조 + 레플리카 서버 (레플리카 서버 2대)
    - 레플리카 1: 쿼리 부하 분산
    - 레플리카 2: 백업

### 6.3. Chain Replication

![](https://velog.velcdn.com/images/minide/post/69b31200-bc25-46f3-9526-56fac4d1c35e/image.png)

- 레플리카 서버가 너무 많아 소스 서버의 복제 부하가 커진다면 고려
- MySQL 서버 업데이트 혹은 장비 교체 시 사용

### 6.4. Dual Source Replication

![](https://velog.velcdn.com/images/minide/post/bc815e65-48ae-4aab-9526-10107679e0db/image.png)

- 두 서버 모두 쓰기 가능
- 각 서버에서 변경한 데이터는 복제를 통해 각 서버에 적용되므로 서로 동일한 데이터 가짐
- ACTIVE-PASSVIE / ACTIVE-ACTIVE 형태로 사용 가능
    - ACTIVE-PASSVIE: 하나의 서버만 쓰기 작업 수행(싱글 레플리카 복제와 동일한 방식이지만, 쓰기 작업 수행 중인 서버에 문제 발생 시 별도의 설정 변경없이 바로 예비용 서버로 쓰기 작업 전환 가능)
    - ACTIVE-ACTIVE: 두 서버 모두 쓰기 작업 수행. 주로 지리적으로 매우 떨어진 위치에서 유입되는 쓰기 요청 원활하게 처리하기 위해 사용
- 트랜잭션 충돌 시 롤백 및 복제 멈춤 현상 발생 → 잘 사용되지 않음

### 6.5. Multi Source Replication

![](https://velog.velcdn.com/images/minide/post/5a1ae422-a39a-40a2-b6e6-64277b30996f/image.png)

- 하나의 레플리카 서버가 둘 이상의 소스 서버를 가짐
- 소스 서버에 흩어진 데이터를 모아 분석을 수행할 때 사용

## 7. Crash-safe Replication

복제를 하다가 레플리카 서버에 문제가 생겨 레플리카 서버를 재구동하는 경우 릴레이 로그에 저장된 바이너리 로그 이벤트 위치와 트랜잭션 실행 정보를 기반으로 소스 서버와의 동기화를 진행하는데, 이를 크래시 세이프 복제(Crash-safe Replication)이라고 한다.

크래시 세이프 복제는 단순히 하나의 옵션을 활성화해서 적용하는 것이 아니라 여러 가지 복제 관련 옵션들을 복제 사용 형태에 따라 적절하게 설정했을 때 얻을 수 있는 효과다.

## References

[[10분 테코톡] 앤지의 DB Replication](https://www.youtube.com/watch?v=NPVJQz_YF2A&t=311s)

[Real MySQL 8.0 (2권)](https://product.kyobobook.co.kr/detail/S000001766483)
