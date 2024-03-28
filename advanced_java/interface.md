# 인터페이스


### 다형성

---

#### 다형성의 개념
다형성이란 사용방법은 동일하지만 실행 결과가 다양하게 나오는 성질을 말합니다. 주로 인터페이스를 통해 다형성을 실현하게 되는데, 인터페이스는 쉽게 말해 역할을 의미한다고 볼 수 있습니다. 인터페이스는 역할이기 때문에 구체적인 실체가 존재하지는 않지만 약속된 특징이 존재합니다. 해당 약속들을 지켜가며 객체를 생성하기 위해 구성한 설계도를 클래스라고 합니다. 따라서 해당 클래스는 인터페이스가 규정하는 약속만 지킨다면 같은 역할을 하는 다양한 객체를 생성할 수 있게 됩니다. 

>이를 웹 프로그래밍에 적용해 보면 클라이언트라는 객체를 변경하지 않은 상태에서, 서버라는 역할만 구현한 객체라면 기존 객체에서 새로운 객체로의 유연한 변경이 가능하게 됩니다.

![Alt text](<image/Screenshot 2024-03-28 at 2.53.18 PM.png>)

객체 A는 인터페이스의 메소드만 사용하므로 객체 B가 객체 C로 변경되는 것에는 관심이 없습니다. 즉 A의 많은 코드 변경 없이 다형성이라는 특징으로 다양한 결과를 불러올 수 있는 특징을 다형성이라고 합니다.

### 실습

---


#### 다형성 예시
인터페이스는 다형성을 구현하기 위해 메소드 오버라이딩과 자동 타입 변환 기능을 이용합니다. 인터페이스의 추상 메소드는 구현 클래스에서 재정의를 해야 하며, 재정의되는 내용은 구현 클래스마다 다릅니다. 구현 객체는 인터페이스 타입으로 자동 타입 변환이 되고, 인터페이스 메소드 호출 시 구현 객체의 재정의된 메소드가 호출되어 다양한 실행 결과를 얻게 됩니다.

> IntelliJ에서 advanced_java_project 생성 후 시작

* Tire.java
```java
package chap01;

public interface Tire {
    void roll();
}
```


* HankookTire.java
```java
package chap01;

public class HankookTire implements Tire {
    // 메소드 오버라이딩
    @Override
    public void roll() {
        System.out.println("한국 타이어가 굴러갑니다.");
    }

}
```

* KumhoTire.java
```java
package chap01;

public class KumhoTire implements Tire {
    // 메소드 오버라이딩
    @Override
    public void roll() {
        System.out.println("금호 타이어가 굴러갑니다.");
    }
}
```

* Car.java
```java
package chap01;

public class Car {
    public Tire tire;

    public void run() {
        tire.roll();
    }
}
```

* CarExample.java
```java
package chap01;

public class CarExample {
    public static void main(String[] args) {
        Car myCar = new Car();
        
        // 자동 타입 변환
        myCar.tire = new HankookTire();
        myCar.run();

        // 자동 타입 변환
        myCar.tire = new KumhoTire();
        myCar.run();
    }
}
```

#### 구성

* 인터페이스 선언
```java
interface 인터페이스명 {...} // default 접근 제한
public interface 인터페이스명 {...} // public 접근 제한
```

* 멤버 선언
```java
public interface 인터페이스명 {
    // 상수 필드 ex) [public static final] double MATH_PI = 3.141592;
    // 추상 메소드 ex) [public abstract] void run();
    // 디폴트 메소드 ex) [public] default void run() {...}
    // 정적 메소드
}
```
대괄호 안에 있는 특성은 생략이 가능합니다. 왜냐하면 인터페이스는 해당 특징을 가지는 멤버 밖에 선언하지 못하기 때문입니다.
* 상수 필드는 인터페이스에 딸린 불변의 변수라고 생각하면 됩니다.
* 추상 메소드는 리턴 타입, 메소드명, 매개변수만 기술되고 중괄호{}를 붙히지 않는 메소드를 말합니다.
* 인터페이스만의 완전한 실행 코드를 가진 메소드를 디폴트 메소드라고 한다. default 키워드가 리턴 타입 앞에 꼭 붙어야 합니다.