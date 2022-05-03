# 불필요한 객체 생성을 피하라

객체의 재사용

같은 기능을 가진 객체를 새로 생성하는 것이 아닌 재사용하는 것이 나을 때가 있다.

```
  // bad code
  String s = new String("TEST");

  // good code
  String s = "TEST";
```

비싼 객체의 재사용

```
  static boolean isRomanNumeralSlow(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
  }
```

해당 코드는 matches() 메소드 안에서 Pattern은 한번 사용되고 버려지기에 재사용이 불가능하다.

차라리 불변 Pattern 인스턴스를 직접 생성한 후에 캐싱해두고 재사용 하는게 효율이 좋다. 불변으로 사용할 경우에는 안전하게 재사용이 가능하기 때문이다.

```
  private static final Pattern ROMAN = Pattern.compile(
    "^(?=.)M*(C[MD]|D?C{0,3})"
      + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

  static boolean isRomanNumeralFast(String s) {
    return ROMAN.matcher(s).matches();
  }
```

### 불필요한 객체 만들지 않기

오토박싱

참조 타입과 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술

기본 타입: 정수, 실수, 논리, 문자같은 기본 타입

참조 타입: 배열, 열거, 클래스, 인터페이스

```
  private static long sum() {
    // Long이라는 참조 타입이 아니라 long으로 하면 속도가 감소된다.
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i;
    return sum;
  }
```

그래서 최대한 참조 타입보다는 기본 타입을 사용하는게 좋다
