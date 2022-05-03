# 인스턴스화를 막으려면 private 생성자를 사용하라

### private 생성자의 필요성

정적 필드와 정적 메소드로만 구성된 클래스를 만들고 싶은 경우가 있는데 객체 지향적으로 사고하지 않는 사람들이 종종 남용하는 방식이나 나름의 쓰임새가 존재한다.

객체 생성자를 명시해주지 않을 경우에는 public의 NoArgsConstructor 기본 생성자를 만들어주기 때문에 막아줄 필요는 있다.

그것을 private 객체 생성자를 통해서 막아주는 것이 필요하다

```
  public class Test {
    private Test() {
      throw new AssertionError();
    }
  }
```

AssertionError()는 굳이 할 필요는 없지만 클래스 내에서 생성자를 호출할 경우를 대비한 예외처리다
