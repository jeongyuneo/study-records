# Synchronization

## 1. 동시성 문제 & 동기화

자바는 멀티 스레드로 동작한다. 즉, 한 프로세스에 여러 스레드가 동작할 수 있다.

동시성 문제는 여러 스레드가 공유 자원에 동시에 접근해 연산을 처리함으로써 원하지 않는 값이 나오는 문제다.

멀티 스레드 환경에서 공유 자원에 대한 순차적인 접근을 보장하기 위해 여러 스레드의 실행을 조정하는 프로세스를 동기화(Synchronization)라고 한다.

## 2. 동시성 문제 해결 방법

### 2.1. Synchronized

synchronized로 지정된 임계 영역 전체에 락을 걸어 다른 스레드가 접근할 수 없게 한다. (mutex 방식)

자바의 모든 객체는 락을 가지고 있다. 이를 **고유락(Intrinsic Lock)** 이라고 한다. synchronized 블록은 객체의 고유락을 이용해 Thread-Safe를 보장한다.

synchronized 블록 전체에 락이 걸리므로 성능 이슈가 있을 수 있다.

- static이 붙으면 **클래스** 단위로 락을 공유
- static이 없으면 **인스턴스** 단위로 락을 공유

### 2.2. Volatile

volatile 키워드가 붙은 변수의 값은 CPU cache가 아닌 **Main Memory에 읽고 쓴다.**

![](https://velog.velcdn.com/images/minide/post/9713e150-6913-43cd-b2c2-ca2282c1d53b/image.png)

멀티 스레드 환경에서 성능 향상을 위해 Main Memory에서 읽은 변수 값을 CPU Cache에 저장한다. 멀티 스레드 환경에서 스레드가 변수의 값을 읽어올 때 각각의 CPU Cache에 저장된 값을 조회하면 변수 값이 불일치하는 문제가 발생할 수 있다.

volatile 키워드를 사용하면 이와 같은 문제를 막을 수 있다.

하지만, Write Thread 1개가 아닌 이상 완벽하게 Thread-Safe를 보장하기 어렵다.

### 2.3. Atomic Class

Atomic Class는 non-blocking 방식으로 동기성 문제를 해결한다. 내부적으로 CAS(Compare-And-Swap) 알고리즘과 volatile을 사용한다.

- CAS(Compare-And-Swap) Algorithm
    1. 인자로 기존 값과 변경할 값을 전달한다.
    2. 기존 값이 Main Memory가 가지고 있는 값과 같으면, 변경할 값을 반영해서 true 반환
    3. 그렇지 않으면, 변경할 값을 반영하지 않고 false 반환

### 2.4. ConcurrentHashMap

HashTable은 synchronized 키워드를 이용해 스레드 간 락을 걸어 Thread-Safe를 보장하기 때문에 매우 느리다. 하지만 ConcurrentHashMap은 Entry 아이템별로 락을 걸어 HashTable보다 성능이 빠르다.

![](https://velog.velcdn.com/images/minide/post/eb251b2a-0861-4ca8-b8b6-2efa0f1dca1e/image.png)

ConcurrentHashMap의 `put()` 메소드를 보면 부분적으로 synchronized가 적용된 것을 볼 수 있다. 즉, 항상 동기화 처리가 되는 것이 아닌 특정 조건에서만 동기화 처리가 되므로 성능이 향상된다.

`put()` 메소드는 빈 bucket에 노드를 삽입하는 경우 CAS 알고리즘을 이용해 락을 사용하지 않고 새로운 노드를 삽입한다.

ConcurrentHashMap의 내부 가변 배열 table을 돌면서 해당 bucket을 가져온다. 해당 bucket이 비어있으면 `casTabAt()`을 이용해 노드를 담고 있는 volatile 변수에 접근하고 null이면 노드를 생성해서 넣는다.

이미 bucket에 노드가 존재하는 경우 synchronized을 이용해 락을 걸어 다른 스레드가 같은 hash bucket에 접근하지 못한다.

비어있지 않은 bucket에서 동일한 key이면 노드를 새로운 노드로 바꾼다. 이 때 해시 충돌이 일어나면 Seperate Chaining이나 TreeNode에 추가한다.

### 2.5. 불변 객체(Immutable Instance)

값이 변할 수 없기 때문에 여러 스레드가 동시에 접근해도 문제가 발생하지 않는다.

ex) String

## References

[[읽고서] 자바 고유락과 Synchronization](https://brunch.co.kr/@kd4/156)

[Java volatile이란?](https://nesoy.github.io/articles/2018-06/Java-volatile)

[ConcurrentHashMap에 대해 알아보자](https://parkmuhyeun.github.io/woowacourse/2023-09-09-Concurrent-Hashmap/)
