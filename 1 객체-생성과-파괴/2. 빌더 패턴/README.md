## 생성자에 매개변수가 많으면 빌드 패턴을 고려해라

정적 팩토리와 생성자의 같은 제약 중 하나는 내부의 매개변수가 많을 때 대응하는게 힘들다라는 것이다.

기본적으로 코드를 짠 사람은 알아보지만 외부 사람이 코드를 볼 때 가장 헷갈리는 것 중 하나는 이 생성자 관련이라고 생각된다.

- 대표적인 예시

```
  public class Person {
    private String name;
    private int height;
    private double weight;
    private int age;

    public Person(String name, int height, double weight, int age) {
      this.name = name;
      this.height = height;
      this.weight = weight;
      this.age = age;
    }
  }


  // 생성자 사용
  Person person = new Person("이름", 180, 72.5, 25);
```

해당 코드를 보았을 때 물론 Person이라는 간단한 객체를 만들어서 알아볼 수 있지만, 실제로 일을 하거나 외부의 클래스를 사용할 때 저 매개변수가 뭘 의미하는지 모를 때가 많아 저 클래스를 직접 찾아보는 번거로움이 생기게 된다.

혹은 여기서 매개변수가 추가될 경우는 더욱 혐오스럽기 그지없다.

```
  public class Person {
    private String name;
    private int height;
    private double weight;
    private int age;
    private int onlyAge; // 만 나이!

    public Person(String name, int height, double weight, int onlyAge, int age) {
      this.name = name;
      this.height = height;
      this.weight = weight;
      this.onlyAge = onlyAge;
      this.age = age;
    }
  }


  // 생성자 사용
  // 23과 25는 도대체 뭐시기여?
  Person person = new Person("이름", 180, 72.5, 23, 25);
```

위의 내용은 굉장히 극단적인 내용이지만 위의 처럼 매개변수의 위치가 변경된다던가, 새로운 매개변수가 추가될 경우에 얘는 도대체 뭐인지를 또 클래스를 직접 찾아가 뭔지 확인 해야하는 번거로움은 또 생기게 된다. 즉 확장이 매우 어렵다.

물론 갓 인텔리제이는 해당 매개변수의 변수명을 가끔 띄워주기는 하는데, 그게 있다고 코드를 인텔리제이에 의존할 수도 없는 노릇이다.

해당 문제에 대해 좋은 방법이 하나 있다면 엔티티를 만들고 setter를 이용해 객체를 생성해 넣어주는 방법이다.

```
  @Setter //각각마다 setter 메소드를 만들어주면 좋지만 귀찮으니... 알아서 Lombok보자!
  @NoArgsConstructor
  public class Person {
    ... 중략
  }


  Person person = new Person();

  person.setName("하이");
  person.setHeight(180);
  person.setWeight(72.5);
  person.setOnlyAge(23);
  person.setAge(25);
```

이런식으로 객체를 만들어주면 코드는 길어졌지만 전보다는 확실히 각각 어떤 값에 뭐가 들어가는지 잘 보이게 된다.

이러한 방식을 '자바 빈즈' 패턴이라고 부르는데 해당 패턴에도 심각한 문제가 존재한다고 한다.

해당 패턴에서는 각각의 매개변수를 설정하기 위해 여러번의 메소드를 호출해야한다는 것, 객체가 완전히 생성되기 전까지는 (TODO: 부터 테스트해서 적어보자!)
