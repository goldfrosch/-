# private 생성자나 열거 타입으로 싱글턴임을 보증하라

Singleton = 인스턴스를 오직 하나만 생성할 수 있는 클래스

ex) 함수(무상태 객체), 설계상 유일해야 하는 시스템 컴포넌트

// TODO 이게 뭔 소리인지 도당체 모르겠

클래스를 싱글턴으로 만들 경우에 사용하는 클라이언트를 테스트하기 어려워질 수 있다. 타입을 인터페이스로 정의한 다음 인터페이스를 구현해 만든 싱글턴이 아닐 경우 싱글턴 인스턴스를 가짜 구현으로 대체할 수 없기 때문이다.

그럼에도 싱글턴을 사용하는 이유는 한번의 객체 생성으로 재사용이 가능하기에 메모리 낭비를 방지할 수 있고 싱글톤으로 생성된 객체는 무조건 한번 생성으로 전역성을 띄기에 다른 객체와 공유가 용이하다.

해당 방식은 Util함수에 사용하면 좋지 않을까 생각함. (개인적인 생각)

싱글턴 제작에는 2개의 방식이 존재하는데 공통적으로는 생성자는 private로 감춰두고 인스턴스에 접근 가능한 public static 멤버를 하나 마련하는 것이다.

1. public static 멤버가 final 필드인 형식

```
  public class Test {
    public static final Test TEST = new Test();

    private Test() {
      ...중략
    }
  }

  @Test
  public void singletonTest() {
    Test test1 = Test.TEST;
    Test test2 = Test.Test;

    //결과 성공
    //두 객체의 주소값이 같음
    assertSame(test1, test2);
  }
```

장점

    1-1. 해당 클래스가 싱글턴임이 API에서 명백하게 드러난다.
    1-2. public static 필드가 final이니 절대로 다른 객체를 참조할 수 없다.
    1-3. 간결하다.

예외라면 권한이 있는 클라이언트는 리플렉션 API인 AccessibleObject.setAccessible (나중 참고필요) 을 이용해서 private 생성자를 호출할 수 있는데 이러한 공격 방어를 위해 생성자를 수정해 두번째 객체가 생성되려 할 때 예외 처리를 해두면 된다한다.

2. 정적 팩토리 메소드 방식의 형식

```
  public class Test {
    private static final Test TEST = new Test();
    private Test() {
      ... 중략
    }

    public static Test getInstance() {
      return TEST;
    }
  }
```

Test.getInstance()는 항상 같은 객체의 참조를 반환하므로 제 2의 Test인스턴스는 만들어지지 않는다.

장점

    2-1. API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다. (getInstance() 호출부 수정없이 내부에서 private static 이 아닌 새 인스턴스를 생성해주면 된다)
    2-2. 원한다면 정적 팩토리 메소드를 제네릭 싱글턴 팩토리로 만들 수 있다.
    2-3. 정적 팩토리 메소드 참조를 공급자로 사용할 수 있다. Test::getInstance 를 Supplier<Test>로 사용하는 방식 (아이템 43, 44 참조 필요)

3. 원소가 하나인 열거 타입을 선언하는 방식

대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.

```
  public enum Test {
    INSTANCE;

    public String getName() {
      return "Test";
    }

    public void testtest() {
      ...중략
    }
  }

  String name = Test.INSTANCE.getName();
```

Test타입의 인스턴스는 INSTANCE하나 뿐 더이상 만들 수 없다.

복잡한 직렬화 상황이나 리플렉션 공격에도 제 2의 인스턴스가 생기는 일을 완벽하게 막아주는 최선의 방법이지만 만들려는 싱글턴이 Enum이외에 다른 상위 클래스를 상속해야할 경우에는 사용할 수 없다.
