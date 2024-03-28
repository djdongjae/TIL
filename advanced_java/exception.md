# 예외 처리


### 자바 오류

---

#### 에러 *Error*

컴퓨터 하드웨어의 고장으로 인해 응용프로그램 실행 오류가 발생하는 것을 자바에서는 에러라고 합니다. 프로그램을 아무리 견고하게 만들어도 개발자는 이런 오류를 대처할 방법이 없습니다.


#### 예외 *Exception*

예외란 잘못된 사용 또는 잘못된 코딩으로 인해 발생하는 오류를 의미합니다. 예외가 발생하면 프로그램은 곧바로 종료된다는 점에서 에러와 동일하지만, 예외 처리를 통해 계속 실행 상태를 유지할 수 있습니다.

자바는 예외가 발생하면 예외 클래스로부터 객체를 생성합니다. 그리고 이 객체는 예외 처리 시에 사용됩니다. 자바의 모든 에러와 예외 클래스는 Throwable을 상속받아 만들어지고, 추가적으로 예외 클래스는 java.lang.Exception 클래스를 상속받게 됩니다.

![Alt text](<image/Screenshot 2024-03-28 at 8.16.02 PM.png>)

* **일반 예외(Exception)**: .java 파일이 .class 파일로 컴파일될 때 예외 여부 검사 과정에서 발생하는 예외를 의미힙니다.
* **실행 예외(Runtime Exception)**: 컴파일러가 예외 처리 코드를 검사하지 않는 예외를 말합니다. 즉 프로그램 실행 중에 발생하는 예외를 의미합니다.
> ✅ 참고로 스프링에서 예외가 발생해도 프로그램이 종료되지 않는 이유는 다음과 같습니다.![Alt text](<image/Screenshot 2024-03-29 at 12.11.45 AM.png>)

### try-catch-finally

---

#### 예외 처리 코드
예외가 발생했을 때 프로그램의 갑작스러운 종료를 막고 정상 실행을 유지할 수 있도록 처리하는 코드를 예외 처리 코드라고 합니다.

#### 구성
try 블록에서 작성한 코드가 예외 없이 정상 실행되면 catch 블록은 실행되지 않고 finally 블록이 실행됩니다. 하지만 try 블록에서 예외가 발생하면 catch 블록이 실행되고 연이어 finally 블록이 실행됩니다.

예외 발생 여부와 상관없이 finally 블록은 항상 실행됩니다. 심지어 try 블록과 catch 블록에서 return 문(메소드 종료)을 사용하더라도 finally 블록은 항상 실행됩니다. finally 블록은 옵션으로 생략 가능합니다.

![Alt text](<image/Screenshot 2024-03-29 at 12.22.42 AM.png>)

#### 실습

* ExceptionHandlingExample1.java
```java
public class ExceptionHandlingExample1 {
    public static void printLength(String data) {
        try {
            int result = data.length();
            System.out.println("문자 수: " + result);
        } catch (NullPointerException e) {
            System.out.println(e.getMessage()); // 1번
            // System.out.println(e.toString()); // 2번
            // e.printStackTrace(); // 3번
        } finally {
            System.out.println("[마무리 실행]\n");
        }
    }

    public static void main(String[] args) {
        System.out.println("[프로그램 시작]\n");
        printLength("멋쟁이사자처럼12기");
        printLength(null);
        System.out.println("[프로그램 종료]");
    }
}
```
1번의 e.getMessage( ) 출력 방식은 예외가 발생한 이유만 반환하고, 2번의 e.toString() 경우 예외의 종류도 반환하여 줍니다. 3번의 e.printStackTrace( )의 경우 예외가 어디서 발생했는지 추적한 내용까지 출력하여 줍니다.

### 상속 관계에서의 예외 처리

---

#### 설명
처리해야 할 예외 클래스들이 상속 관계에 있을 때는 하위 클래스 catch 블록을 먼저 작성하고 상위 클래스 catch 블록을 나중에 작성해야 합니다. 예외가 발생하면 catch 블록은 위에서부터 차례대로 검사 대상이 되므로 하위 예외도 상위 클래스 타입이므로 상위 클래스 catch 블록이 먼저 검사 대상이 되면 안됩니다.

#### 실습
* ExceptionHandling2.java
```java
public class ExceptionHandlingExample2 {
    public static void main(String[] args) {
        String[] array = {"100", "1oo"};

        for (int i = 0; i < 3; i++) {
            try {
                int value = Integer.parseInt(array[i]);
                System.out.println("array[" + i + "]: " + value);
            } catch (ArrayIndexOutOfBoundsException e) {
                System.out.println("배열 인덱스가 초과됨: " + e.getMessage());
            } catch (Exception e) {
                System.out.println("실행에 문제가 있습니다.");
            }
        }
    }
}
```
Integer.parseInt( array[ i ] )에 대한 예외가 2번째 catch 블록에서 먼저 처리되고, 이후에 배열 인덱스에 대한 예외가 1번째 catch 블록에서 처리 됩니다.


### throws

---

#### 설명

메소드 내부에서 예외가 발생했을 때 try-catch 블록으로 예외를 처리하는 것이 기본이지만, 메소드를 호출한 곳으로 예외를 처리하게끔 throw 할 수 있습니다. 일종의 책임 전가라고 볼 수 있지요. 이때 사용하는 키워드가 throws이며 메소드 선언부 끝에 떠넘길 예외 클래스들과 함께 작성해줍니다.

```java
리턴타입 메소드명(매개변수, ...) throws 예외클래스1, 예외클래스2, ... {
}
```

#### 실습
``` java
public class ThrowsExample {
    public static void main(String[] args) throws Exception{
        findClass();
    }

    public static void findClass() throws ClassNotFoundException {
        Class.forName("멋쟁이사자처럼");
    }
}
```

위 코드를 실행하면 findClass 메소드는 ClassNotFoundException 예외를 발생시키고 이를 main 메소드에 책임 전가합니다. main 메소드는 발생한 예외를 최종적으로 JVM에게 책임 전가하고 JVM은 예외의 내용을 콘솔에 출력하는 것으로 예외를 처리합니다.