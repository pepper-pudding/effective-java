# 아이템 3: private 생성자 또는 enum 타입을 사용해서 싱글톤으로 만들어라

## 싱글톤

오직 한 인스턴스만 만드는 클래스를 싱글톤이라 부른다.&#x20;

싱글톤으로 만드는 두 가지 방법이 있는데, 두 방법 모두 생성자를 private으로 만들고 public static 멤버를 사용해서 유일한 인스턴스를 제공한다.



### 방법 1 : final 필드

첫 번째 방법은 final 필드로 제공하는 방법이다.

* 생성자는 오직 최초 한번만 호출되고 Elvis는 싱글톤이 된다.

```
public class Elvis {
    public static final Elvis instance = new Elvis();
    private Elvis() {

    }
}

public class SingleTest {
    public static void main(String[] args){
        Elvis singleton1 = Elvis.instance;
        Elvis singleton2 = Elvis.instance;
        //처음 만들어진 인스턴스를 재사용
    }
}
```

&#x20;&#x20;

단, 이걸 뚫는 방법이 있다. 리플렉션을 사용해서 private 생성자를 호출하는 방법이다.

```
public class SingleTest {
    public static void main(String[] args) throws NoSuchMethodException {
        Constructor<Elvis> constructor = Elvis.class.getConstructor();
        constructor.setAccessible(true);
        Elvis singleton1 = constructor.newInstance(?);
    }
}
```

&#x20;

이 방법을 막고자 생성자 안에서 생성한 인스턴스 개수를 카운팅하거나 flag를 이용해서 예외를 던지게 할 수 있다.

```
public class Elvis {
    public static final Singleton1 instance = new Elvis();

    int count;

    private Elvis() {
        count++;
        if (count != 1) {
            throw new IllegalStateException("this object should be singleton");
        }
    }
}
```

&#x20;

#### 장점

* 이런 API 사용이 static 팩토리 메소드를 사용하는 방법에 비해 더 명확하고 더 간단하다.



### 방법 2 : static 팩토리 메소드

```
public class Elvis {

    private static final Elvis INSTANCE = new Elvis(); //private!!

    private Elvis() {
    }

    public static Elvis getInstance() {
        return INSTANCE; //생성한 인스턴스 리턴
    } //public!! static 팩토리 메소드 사용!!

}
```

&#x20;

#### 장점

* API를 변경하지 않고도 싱글톤으로 쓸지 안쓸지 변경할 수 있다.
* 처음엔 싱글톤으로 쓰다가 나중엔 쓰레드당 새 인스턴스를 만드는 등 클라이언트 코드를 고치지 않고도 변경할 수 있다.

```
public class Elvis {

    private static final Elvis INSTANCE = new Elvis(); //private!!

    private Elvis() {
    }

    public static Elvis getInstance() {
        return new Elvis(); //이렇게 바꾸면 더 이상 싱글톤 아님. 클라이언트 코드를 일일이 고치지 않고도 변경 가능.
    } 
}
```

&#x20;

* 필요하다면 Generic 싱글톤 팩토리 (아이템30)를 만들 수도 있다.
* static 팩토리 메소드를 Supplier에 대한 메소드 레퍼런스로 사용할 수도 있다.
* [interface supplier](https://docs.oracle.com/javase/8/docs/api/java/util/function/Supplier.html)

```
    public static Elvis getInstance() {
        return INSTANCE;
    }
    //이 메소드 자체가 interface supplier처럼 취급이 될 있는 메소드
```

```
    Supplier<Elvis> s1supplier = Elvis::getInstance;
```

***

## 직렬화 (Serialization)

* 위에서 살펴본 두 방법 모두, 직렬화에 사용한다면 역직렬화 할 때 같은 타입의 인스턴스가 여러 개 생길 수 있다.
  * 역직렬화할 때마다 new를 사용하게 되면서 새로운 객체가 나오고, 그러면 똑같은 타입의 객체가 여러 개가 되는 것.

&#x20;

### 해결책 1 : trasient, readResolve()의 사용

* 이 문제를 해결하려면 모든 인스턴스 필드에 transient를 추가 (직렬화 하지 않겠다는 뜻) 하고,

```
    private static final transient Elvis INSTANCE = new Elvis();
    //INSTANCE를 직렬화하지 않는다.
```

&#x20;

* readResolve 메소드를 다음과 같이 구현하면 된다. ([객체 직렬화 API의 비밀](http://www.oracle.com/technetwork/articles/java/javaserial-1536170.html) 참고)

```
    private Object readResolve() {
        return INSTANCE;
    }
    //역직렬화할 때마다 이 메소드가 호출된다. (이 메소드가 있으면)
    //그래서 이 함수 안에다가 이미 생성된 객체를 리턴하도록 하는 것이다.
```

&#x20;

### 해결책 2 : Enum의 사용

직렬화/역직렬화 할 때 코딩으로 문제를 해결할 필요도 없고, 리플렉션으로 호출되는 문제도 고민할 필요없는 방법이 있다.

```
public enum Elvis {
    INSTANCE;

    public String getName() {
        return "yellme";
    }
}
```

```
    String name = Elvis.INSTANCE.getName();
```

* 코드는 좀 불편하게 느껴지지만 싱글톤을 구현하는 최선의 방법이다. (이상적)
* 하지만 이 방법은 Enum 말고 다른 상위 클래스를 상속해야 한다면 사용할 수 없다. (하지만 인터페이스는 구현할 수 있다.)



<details>

<summary>실습</summary>

```
import java.lang.reflect.*;

enum Elvis {
    INSTANCE;

    public String getName() {
        return "yellme";
    }
}

class ElvisFinal {
    public static final ElvisFinal INSTANCE = new ElvisFinal();

    int count;

    private ElvisFinal() {
        count++;
        if (count != 1) {
            throw new IllegalStateException("this object should be singleton");
        }
    }
}

class ElvisStaticFactory {
    private static final ElvisStaticFactory INSTANCE = new ElvisStaticFactory();

    private ElvisStaticFactory() {
    }

    public static ElvisStaticFactory getInstance() {
        return INSTANCE;
    }

    public static ElvisStaticFactory getNewInstance() {
        return new ElvisStaticFactory();
    }
}

public class Singleton {
    public static void main(String[] args) {
        System.out.println("======== 1. Final ========");
        ElvisFinal singleton1 = ElvisFinal.INSTANCE;
        ElvisFinal singleton2 = ElvisFinal.INSTANCE;
        System.out.println(singleton1.hashCode());
        System.out.println(singleton2.hashCode());

        System.out.println("\n======== 2-1. StaticFactoryMethod : Same Object ========");
        ElvisStaticFactory singleton3 = ElvisStaticFactory.getInstance();
        ElvisStaticFactory singleton4 = ElvisStaticFactory.getInstance();
        System.out.println(singleton3.hashCode());
        System.out.println(singleton4.hashCode());

        System.out.println("\n======== 2-2. StaticFactoryMethod : New Object ========");
        System.out.println(singleton3.hashCode());
        ElvisStaticFactory singleton5 = ElvisStaticFactory.getNewInstance();
        System.out.println(singleton5.hashCode());


        System.out.println("\n======== 3. Enum ========");
        System.out.println(Elvis.INSTANCE.hashCode());
        System.out.println(Elvis.INSTANCE.getName());

        System.out.println("\n======== 4. 리플렉션으로 private 생성자 호출 시도 ========");
        try {
            Constructor<ElvisFinal>  constructor = ElvisFinal.class.getConstructor(); //흠.. 체크
            constructor.setAccessible(true);    
            ElvisFinal singleton6 = constructor.newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

/* 실행결과
======== 1. Final ========
1365202186
1365202186

======== 2-1. StaticFactoryMethod : Same Object ========
1586600255
1586600255

======== 2-2. StaticFactoryMethod : New Object ========
1586600255
474675244

======== 3. Enum ========
212628335
yellme

======== 4. 리플렉션으로 private 생성자 호출 시도 ========
java.lang.NoSuchMethodException: ElvisFinal.<init>()
        at java.base/java.lang.Class.getConstructor0(Class.java:3349)
        at java.base/java.lang.Class.getConstructor(Class.java:2151)
        at Singleton.main(Singleton.java:65)
*/
```

</details>
