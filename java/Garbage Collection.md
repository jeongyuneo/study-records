# Garbage Collection

## 1. Garbage Collection

Garbage Collection은 프로그램이 동적으로 할당했던 메모리 영역(**Heap 영역**) 중 필요 없게 된 영역(**어떤 변수도 가리키지 않는 영역**)을 알아서 해제하는 기법이다.

C나 C++의 경우 Heap 영역의 메모리를 관리하기 위해 코드 레벨에서 메모리를 할당받고 해제해야하지만, Java에서는 동적 메모리 영역 해제를 Garbage Collection이 해준다.

### 1.1. Garbage Collection 장점

- Memory Leak 발생하지 않음
- 휴먼 에러 발생 가능성 낮춤

### 1.2. Garbage Collection 단점

- 성능 저하

  어떤 메모리 영역을 해제할지 검사하고 해제하는 일은 CPU 자원과 메모리를 필요로 한다.

- 개발자는 언제 GC가 메모리를 해제하는지 모름

## 2. Garbage Collection & Reachability

Garbage Collection은 객체가 Garbage인지 판별하는 기준으로 reachability라는 개념을 사용한다. 어떤 객체에 유효한 참조가 있으면 reachable, 없으면 unreachable로 구별하고, unreachable 객체를 대상으로 Garbage Collection을 수행한다.

![](https://velog.velcdn.com/images/minide/post/fdd3d631-9e47-421e-b843-7bbb46936086/image.png)

- Heap 영역에 있는 객체에 대한 참조 종류
    1. Heap 영역 내 다른 객체에 의한 참조
    2. JVM Stack 영역, 즉 Java 메소드 실행 시 사용하는 지역 변수와 파라미터들에 의한 참조
    3. Native Method Stack 영역, 즉 JNI(Java Native Interface)에 의해 생성된 객체에 대한 참조
    4. Method Area의 정적 변수에 의한 참조

![](https://velog.velcdn.com/images/minide/post/f6cfe9d8-78d7-49df-bf1f-228df52c037c/image.png)

Heap 영역(1번)을 제외한 나머지 3개(2~4번)가 Root Space로, reachability를 판단하는 기준이 된다.

## 3. Garbage Collection Algorithm

Garbage Collection을 성공적으로 수행하는 알고리즘을 설계하기 위해서는 몇 가지 가설이 필요한데, 그 중 대표적인 것이 **Weak Generational Hypothesis**다.

이 가설은 대부분의 객체는 빠르게 unreachable한 상태로 전환된다고 보고, Heap 영역을 기준으로 Old Generation에서 Young Generation으로의 참조 방향은 적게 존재한다고 가정한다.

### 3.1. Reference Counting

![](https://velog.velcdn.com/images/minide/post/18e13093-712e-4276-95d5-2b67b267cfaa/image.png)

- 몇 가지 방법으로 해당 객체에 접근할 수 있는지를 나타내는 Reference Count를 가짐
- Reference Count가 0이 되면 GC의 대상이 됨
- **순환 참조 문제**라는 한계가 있음
    - Root Space로부터 연결이 끊긴 순환 참조되는 객체들의 Reference Count가 1로 유지되기 때문에 사용하지 않는 메모리 영역이 해제되지 못하고 Memory Leak 발생

### 3.2. Mark and Sweep

![](https://velog.velcdn.com/images/minide/post/c575ff89-678c-4ade-85fd-899f0e849226/image.png)

- Root Space로부터 해당 객체에 접근 가능한지를 해제의 기준으로 삼음
    - Mark: Root Space로부터 그래프 순회를 통해 접근 가능한 객체를 찾음
    - Sweep: 연결이 끊어진 객체들 메모리 해제

### 3.3. Mark and Compact

![](https://velog.velcdn.com/images/minide/post/dbf91dfb-967b-47e0-b097-2629052cf5f1/image.png)

- Mark and Sweep 알고리즘 이후 메모리를 정리하여, 메모리 파편화 문제를 해결(Compaction)

## 4. JVM의 Garbage Collection

![](https://velog.velcdn.com/images/minide/post/c6ab6211-1ed8-47b2-bc14-38b5de50412e/image.png)

- Minor GC: Young Generation에서 발생하는 GC
    - Eden 영역에 새롭게 생성된 객체들이 할당
    - Minor GC로부터 살아남은 객체들이 Survivor 영역으로 이동 & age-bit 증가
        - Survivor 0 또는 1 둘 중 하나는 반드시 비어 있어야 함(Minor GC로부터 살아남은 객체들은 Survivor 0과 1을 번갈아가며 이동)
    - 객체의 age-bit가 일정 수준을 넘어가면 해당 객체를 Old Generation으로 이동(Promotion)
- Major GC: Old Generation에서 발생하는 GC
    - Minor GC보다 더 오래 걸림

### 4.1. Stop The World

Stop The World는 GC가 동작할 때 GC가 동작하는 메인 스레드가 아닌 나머지 모든 스레드(애플리케이션 실행)가 일시적으로 멈추는 현상이다.

따라서 애플리케이션 성능을 최적화하기 위해서는 Stop The World 시간을 최소화해야 한다.

### 4.2. Garbage Collection 종류

- Serial GC
    - 하나의 스레드로 GC 실행
    - Stop The World 시간이 긺
    - Minor GC에는 Mark and Sweep 알고리즘, Major GC에는 Mark and Compact 알고리즘 사용
    - 싱글 스레드 환경 및 Heap 영역이 매우 작을 때 사용
- Parallel GC
    - **Java 8 Default GC**
    - 여러 개의 스레드로 GC 실행
    - Serial GC와 기본적인 알고리즘은 같지만, Minor GC를 멀티스레드로 수행(Major GC는 싱글 스레드)

      → Serial GC에 비해 Stop The World 시간이 짧음

    - 멀티 코어 환경에서 애플리케이션 처리 속도 향상을 위해 사용
- Parallel Old GC
    - Parallel GC 개선한 버전
    - Mark-Summary-Compact 알고리즘 사용
        - Minor GC 뿐만 아니라 Major GC 도 멀티스레드로 수행
- CMS GC (Concurrent Mark Sweep)
    - Stop The World 시간을 최대한 줄이기 위해 고안됨
        - GC 과정이 복잡한 여러 단계로 수행되기 때문에 다른 GC 대비 CPU 사용량 높음
    - Mark and Sweep 알고리즘 사용 -> 메모리 파편화 발생
    - **Java 9부터 deprecated** → **Java 14에서 사용 중지**
- G1 GC (Garbage Firest)
    - CMS GC를 대체하기위해 JDK 7 버전에서 최초 release
    - Heap 영역이 너무 작을 경우 미사용 권장(4GB 이상의 Heap 메모리 필요)
    - Heap 영역이 **Region** → 전체 GC 시간 단축
        - 상황에 따라 Eden, Survivor, Old 등 역할을 **동적으로 부여**
    - 고용량, 대규모에 짧은 GC시간 보장
    - **Java 9+ Default GC**
- Z GC
    - Java 15 release
    - 대용량 메모리(8MB ~ 16TB)를 low-latency로 처리하기 위해 고안됨
    - ZPage 영역 사용
        - G1 GC의 Region은 크기가 고정이지만, ZPage는 2MB 배수로 동적으로 운영됨
    - Heap 크기가 증가해도 Stop The World 시간이 **10ms**를 초과되지 않음을 보장

## References

[Java Garbage Collection](https://d2.naver.com/helloworld/1329)

[Java Reference와 GC](https://d2.naver.com/helloworld/329631)

[가비지 컬렉션 동작 원리 & GC 종류 총정리 - Inpa Dev](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98GC-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC)

[[10분 테코톡] 조엘의 GC](https://www.youtube.com/watch?v=FMUpVA0Vvjw)
