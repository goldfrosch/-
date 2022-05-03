# finalize와 cleaner 사용을 피하라

자바에는 객체 소멸자가 2개가 존재한다

finalize: 예측할 수 없고 상황에 따라 위험할 수 있어 일반적으로는 불필요하다. 나름의 쓰임새가 있으나 기본적으로 쓰면 안된다.

cleaner: finalize보다는 덜 위험하나 여전히 예측할 수 없고 느리고 일반적으로 불필요하다.

두개를 사용하면 안되는 이유는 크게 4가지가 존재한다.

1. 제때 실행되어야 하는 작업은 절대 할 수 없다.

   파일 닫기를 두개에 맡기면 중대한 오류를 일으킬 수 있다. 시스템이 finalize나 cleaner 실행을 게을리 해서 파일을 계속 열어둔다면 새로운 파일을 열지 못해 프로그램이 실패로 돌아갈 수 있다.

2. 실행 가능성을 보장받지 못한다.

   예를 들어 데이터베이스 같은 공유 자원의 영구 락 해제를 finalize나 cleaner에 맡겨 놓으면 분산 시스템 전체가 서서히 멈출 것이다.

   finalizer 동작 중 발생한 예외는 무시되며 작업을 처리할 게 남았어도 그 순간 종료되기 때문에 어떠한 상태를 영구적으로 수정하는 작업에서는 절대 의존해서는 안된다.

3. 성능 문제

   내 컴퓨터에서 간단한 AutoCloseable 객체를 생성하고 가비지 컬렉터가 수거하기 까지 12ns가 걸리는데 finalizer를 사용하면 550ns가 걸린다. finalizer가 가비지 컬렉터의 효율성을 떨어트리기 때문에 해당 문제가 발생하게 되는데, 성능 저하가 되는 순간부터 사용하기 꺼려지게 되는 것이다.

4. 보안 문제

   finalizer 공격 원리는 단순한데 생성자나 직렬화 과정에서 예외가 발생하게 될 때 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있는데 이 finalizer를 정적 필드에 자신의 참조를 할당해 가비지 컬렉터가 수집하지 못하게 막아버릴 수 있다. 해당 문제는 finalize 메소드를 만들고 final로 선언하면 누구도 하위 클래스는 만들수 없기 때문에 final로 선언해 막을 수 있다.

가장 정상적으로 자원 사용 후 반납하는 방식은 AutoCloseable을 구현해주고 클라이언트에서 사용한 후 close 메소드를 호출해 종료해주는 방식이 잇다. 다른 메소드에서는 close를 이용해 닫힌 후에 불린 것을 IllegalStateException으로 예외 처리를 해줄 수 있다.

이렇게 들으면 누가 봐도 안 좋은 것으로 인지되는데 사용하기에 적절한 방법이 2가지가 존재한다.

첫번째는 close 메소드가 호출되지 않을 경우를 대비하는 상황이다.

cleaner나 finalize가 즉시 호출되는 보장은 없으나 클라이언트에서 이루어지지 않은 자원 회수가 늦게라도 일어나는 것이 더 좋은 방법이기 때문에 사용한다.

ex) FileInputStream, FileOutputStream, ThreadPoolExecutor

두번째는 네이티브 피어와 연결된 객체에서 사용한다.

A native object is not programmed only in java, but in a platform specific language, typically c or assembler.

Memory allocated by this code cannot be disposed by the GC. Therefore you may need to clean it in a finalizer.

The native peer is the native part of a Java object.

네이티브 오브젝트는 자바에서만 프로그래밍 되는 것이 아닌 플랫폼별 언어로 프로그래밍 된 것을 의미한다고 한다.

네이티브 피어는 자바 객체가 아니라 가비지 컬렉터에서 인식하지 못한다. 그러기 때문에 네이티브 피어는 가비지 컬렉터에서 회수되지 않는데, 그 문제를 해결할 때 사용해서 청소해줄 수 있다.
