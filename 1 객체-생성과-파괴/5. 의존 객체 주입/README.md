# 자원을 직접 명시하지 말고 의존 객체 주입을 사용해라

클래스들이 자원(다른 클래스)에 의존하는 경우가 존재한다.

1. 정적 유틸리티 클래스

```
  //private 빈 생성자를 만들어서 인스턴스화를 방지한다.
  @NoArgsConstructor(access = AccessLevel.PRIVATE)
  public class SpellChecker {
    private final static Lexicon dictionary = ...;

    public static boolean isValid(String word) {
      ...중략;
    }

    public static List<String> suggestions(String typo) {
      ...중략;
    }
  }

  SpellChecker.isValid(word);
```

2. 싱글턴

```
  //private 빈 생성자를 만들어서 인스턴스화를 방지한다.
  @NoArgsConstructor(access = AccessLevel.PRIVATE)
  public class SpellChecker {
    private final Lexicon dictionary = ...;
    public static SpellChecker INSTANCE = new SpellChecker(...);

    public static boolean isVaild(String word) {...}
    public static List<String> suggestions(String typo) {...}
  }

  SpellChecker.INSTANCE.isValid(word);
```

두 방법 모두 확장에는 유연하지 않고 테스트가 어렵다

사전은 굉장히 여러 종류가 있는데(한국어 사전, 영어 사전, 특수 어휘용 사전 등...)
dictionary 하나로만 이 역할을 모두 수행하기에는 어렵고,
SpellChecker는 dictionary 하나만 사용할 수 있기 때문이다.

사용하는 자원에 따라 동작이 달라지는 클래스는 위 두 방법이 적합하지 않다.

그래서 final을 삭제해 사전을 교체하는 메소드를 작성하는 방식을 택하면

```
  public class SpellChecker {
    private static Lexicon dictionary = ...;

      ...중략;
      public static void changeDictionary(Lexicon new) {
        dictionary = new;
      }
      ...중략;
  }

  SpellChecker.changeDictionary(newDictionary);
```

무언가 어색한 느낌이 들기도 하지만 멀티 쓰레드 환경에서는 사용할 수 없다고 한다.

3. 의존 객체 주입의 형태

해당 방법은 인스턴스를 생성할 때 필요한 자원을 넘겨주는 방식이다.

```
  public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary){
    	this.dictionary = Objects.requireNotNull(dictionary);
    }

    public static boolean isVaild(String word) {...}
    public static List<String> suggestions(String typo) {...}
  }

  interface Lexicon {}

  // Lexicon을 상속 받아서 구현
  public class myDictionary implements Lexicon {
    ...
  }

  Lexicon dic = new myDictionary();
  SpellChecker chk = new SpellChecker(dic);

  chk.isVaild(word);
```

이 패턴의 응용 방법으로는 생성자에 자원 팩토리를 넘겨주는 방식이 있다. (팩토리 메소드 패턴)
ex. Supplier

이처럼 의존성 주입은 유연성과 테스트 용이성을 개선해주지만, 의존성이 너무 많아지면 코드가 장황해질 수도 있다.
