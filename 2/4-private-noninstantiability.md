# 아이템 4: private 생성자로 noninstantiability를 강제하라

static 메서드와 static 필드를 모아둔 클래스를 만든 경우, 해당 클래스를 abstract로 만들어도 인스턴스를 만드는 걸 막을 순 없다.

상속 받아서 인스턴스를 만들 수 있기 때문이다.

```java
public abstract class UtilClass {
    public static String getName() {
        return "yellme";
    }

    static class AnotherClass extends UtilClass {

    }

    public static void main(String[] args){
        AnotherClass anotherClass = new AnotherClass();
    }
}
```

&#x20;

아무런 생성자를 만들지 않은 경우 컴파일러가 기본적으로 아무 인자가 없는 pubilc 생성자를 만들어 주기 때문에 그런 경우에도 인스턴스를 만들 수 있다.

따라서, 명시적으로 private 생성자를 추가해야 한다.

```java
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
}
```

&#x20;

생성자를 제공하지만 쓸 수 없기 때문에 직관에 어긋나는 점이 있으므로 위에 코드처럼 주석을 추가하는 것이 좋다.

부가적으로 상속도 막을 수 있다. 상속한 경우에 명시적이든 암묵적이든 상위 클래스의 생성자를 호출해야 하는데, 이 클래스의 생성자가 private이라 호출이 막혔기 때문에 상속을 할 수 없다.



<details>

<summary>실습</summary>

```java
abstract class UtilClass {
    public static String getName() {
        return "pudding";
    }
}

class AnotherClass extends UtilClass {

}

// Noninstantiable utility class
class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
}

public class PrivateConstructor {
    public static void main(String[] args) {
        AnotherClass anotherClass = new AnotherClass();
        System.out.println(anotherClass.getName());
        // 실행결과
        // pudding

        /*
        * PrivateConstructor.java:25: error: UtilityClass() has private access in UtilityClass
        */
        // UtilityClass utilityClass = new UtilityClass();
    }
}
```

</details>

\
