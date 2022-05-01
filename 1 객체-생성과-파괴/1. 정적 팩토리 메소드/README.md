# 생성자 대신 정적 팩토리 메소드

클라이언트에서 클래스의 인스턴스를 얻는 전통적인 수단은 public 생성자다

하지만 생성자 이외에도 클래스의 인스턴스를 반환하는 방법이 하나 있는데 그 방법이 정적 팩토리 메소드다

```
  public static Boolean valueOf(boolean value) {
    return value ? Boolean.TRUE : Boolean.false;
  }
```

해당 방식을 사용해서 클라이언트에 생성자와 정적 팩토리 메소드를 이용해서 인스턴스를 제공해 줄 수 있는데 뭐든 장단점이 존재하듯이 정적 팩토리 메소드에도 장단점이 존재한다.

- 장점

  - 이름을 가질 수 있다.

    생성자를 사용해서 인스턴스를 반환할 때는 여러가지 방식이 존재한다 하지만 일반적인 생성자를 통해서 인스턴스를 반환하게 될때는 보통 다형성을 이용해서 다양한 매개변수에 따른 생성자를 만들게 된다. 하지만 이 방식은 뭐가 뭔지를 헷갈릴 때가 종종 존재하게 되는데 정적 팩토리 메소드를 사용하게 되면 해당 메소드에 대한 이름이 존재하기 때문에 이름만 잘 짓게 된다면 어떤게 반환되는지를 명확하게 알 수 있다라는 큰 장점이 존재한다.

    ```
      public class Person {
        private String name;
        private int age;
        private String job;

        public Person (String name, int age) {
          this.name = name;
          this.age = age;
        }

        public Person (String name, String job) {
          this.name = name;
          this.job = job;
        }

        public static Person getNameWithAge(String name, int age) {
          return new Person(name, age);
        }
      }
    ```

    이런식으로 한번 감싸주면 조금더 원활하게 사용이 가능하다라는 점

  - 호출될 때 마다 인스턴스를 새로 생성하지 않아도 된다.
    정적 팩토리 메소드 덕에 인스턴스를 미리 만들어두거나 새로 생성한 인스턴스를 캐싱해서 재활용하는 방식으로 객체 생성을 피할 수 있다.

    즉 생성자를 이용해서 새로운 객체를 생생하는 것이 아닌 클래스에서 바로 선언이 가능하다라는 것

    ```
      public class Person {
        ... 중략
      }

      public static void main(String[] args) {
        System.out.println(Person.getPersonWithJob("이름","직업").getName());
      }
    ```

    해당 부분을 통해서의 가장 큰 장점은 미리 만들어두거나 새로 생성한 인스턴스를 캐싱해서 재활용하기 때문에 불필요한 객체 생성을 줄일 수 있어 매번 새로운 객체를 생성할 필요가 없어지기 때문에 최적화에 도움이 될 수 있다.

  - 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

    하위 자료형 객체를 반환하는 정적 팩토리 메소드의 특징은 상속 사용에서 확인이 가능한데 생성자의 역할을 하는 정적 팩토리 메소드가 반환값을 가지고 있기 때문에 하위 타입으로의 반환이 가능하다.

    ```
    class GradeCalculator {
      static Grade of(int score) {
        if (score >= 90) {
            return new A();
        } else if (score >= 80) {
            return new B();
        }
        return new F();
      }
    }

    interface Grade {
      String toText();
    }

    class A implements Grade {
      @Override
      public String toText() {
        return "A";
      }
    }

    class B implements Grade {
      @Override
      public String toText() {
        return "B";
      }
    }

    class F implements Grade {
      @Override
      public String toText() {
        return "F";
      }
    }

    ```

    자바 9의 List 인터페이스의 of 메소드도 인터페이스를 반환하는 정적 팩토리 메소드라 한다.

  - 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

    // TODO 뭔말인지 1도 모르겠다.

    반환 타입의 하위 타입이기만 한다면 어느 타입이든 객체를 반환해도 상관이 없다

  -

- 단점

  - 상속을 위해서는 public이나 protected 생성자가 필요한데 정적 팩토리 메소드를 사용하게 된다면 하위 클래스를 만드는게 문제가 있다.

    상속을 사용해서 정적 팩토리 메소드를 사용하게 될 경우에는 아에 사용이 불가능하다라는 단점이 존재한다. 대표적인 예시로 java.utils.Collections는 정적 팩토리 메소드를 주로 사용하기에 해당 클래스들로 만든 구현체는 상속이 불가능하다라는 것이다.

  - 정적 팩토리 메소드는 프로그래머가 찾기 어렵다
    생성자마냥 Javadoc 문서에서 따로 정리해주지 않는다고 한다. API문서를 잘 적고 메소드 이름도 알려진 규약을 따라 짓는 방식으로 해당 문제를 해결해야한다. 해당 문제는 공용으로 사용해서 배포할 때의 문제가 아닐까 싶다.
