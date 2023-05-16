# String Pool

## 1. String 객체

Java에서 문자열을 표현하는 String은 불변객체다.
```java
String name = "jeongyun";
name += "eo";
System.out.println(name);  // 출력결과: jeongyuneo
```
위 코드에서 name에 문자열을 더한다고 해서 문자열의 값이 바뀌는 것이 아니라, 새로운 객체를 만들어 그 참조값을 참조하게 된다.

만약 같은 값을 갖는 문자열을 여러 번 선언한다면, Heap 영역에 매번 새로운 객체를 생성하게 될 것이다. 이는 메모리 낭비로 이어진다.

따라서 Java는 문자열 객체를 캐싱(caching)한다.

> 캐싱(caching)
> - 데이터를 미리 복사해 임시 저장해놓는 것
>- 데이터 접근 시간을 줄일 수 있음

## 2. String Pool

Java의 Heap 영역에는 String Pool이 존재하고, String Pool이 String 객체들을 관리한다.

String Pool은 HashTable 구조를 갖고 있다. 문자열 리터럴을 사용해 문자열 객체를 생성하면 String Pool에 같은 값을 가지는 String 객체가 있는지 검사하고 있다면 그 객체의 참조값을, 없다면 새로운 String 객체를 생성해 그 객체의 참조값을 반환한다.

이러한 과정을 ‘intern’이라고 한다. String 객체를 위 코드처럼 리터럴로 생성 시 내부적으로 intern()이라는 메소드를 실행해 수행한다.

따라서 같은 문자열은 myPet과 hisPet 두 객체의 참조값을 비교하는 동일성 비교(==)에도 true가 반환됨을 확인할 수 있다.

![R1280x0](https://github.com/SoftwareMaestro-Backend-Study/java-orm-jpa-study/assets/62989828/056a6a4b-dc98-4182-9448-981efc2b871f)

반대로 new 연산자를 이용해 생성된 객체들은 String Pool이 아닌 Heap 영역에 생성된다. 각각의 객체들은 서로 다른 참조값을 갖고 있기 때문에 이런 방식으로 생성된 myPet과 hisPet 두 객체의 참조값을 비교하는 동일성 비교(==)에서 false가 반환됨을 확인할 수 있다.

![R1280x0-2](https://github.com/SoftwareMaestro-Backend-Study/java-orm-jpa-study/assets/62989828/9df26b3d-30fd-4765-a2e3-803a7e23ee31)

Java 6까지 String Pool은 Heap 영역 안에서도 Perm 영역에 있었다. 이때문에 String Pool에 문자열 객체가 많이 생성되거나 다른 이유로 이 영역이 가득 차게되면, 런타임 환경에서 메모리를 동적으로 늘리지 못해 OutOfMemoryException 발생했다.

Java 7에서는 다른 일반 객체들과 마찬가지로 Perm 영역이 아닌 Heap 영역에 String Pool을 생성해서 OutOfMemoryException이 발생했을 때의 위험성을 줄였다.

Java 8부터는 고정 크기로 많은 문제를 일으켰던 Perm 영역은 Metaspace로 이름을 변경하고 동적크기 메모리 구성으로 변경되었다.