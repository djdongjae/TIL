# 제네릭과 컬렉션 자료구조


### 제네릭: *Generic*

---

#### 제네릭이 필요한 이유

다음과 같이 Box 클래스를 선언하고자 합니다. 박스에 넣을 내용물이 정수일지 문자열일지 모르는 상황에서 어떤 타입으로 선언을 해야 할까요?
```java
public class Box {
    public ? content;
}
```
실제 박스를 조립하기 전에(Box 클래스에 대한 객체를 생성하기 전에) 저희는 어떤 내용물을 넣을지 미리 알고 있습니다. 따라서 Box 객체를 생성하기 전에 저장할 내용물의 타입을 미리 알려주면 content에 무엇이 대입되고, 읽을 때 어떤 타입으로 제공할지를 알게 됩니다.

> ✅ 제네릭이란 결정되지 않은 타입을 파라미터로 처리하고 실제 사용할 때 파라미터를 구체적인 타입으로 대체시키는 기능입니다.

```java
public class Box<T> {
    public T content;
}
```
여기서 T는 단지 이름일 뿐입니다. T 대신에 어떠한 알파벳도 활용할 수가 있습니다. 핵심은 우리가 클래스를 선언할 때 content의 타입으로서 어떠한 형태가 올지 모를 때 이러한 기능을 활용할 수 있다는 것입니다.

```java
Box<String> box1 = new Box<String>();
box.content = "안녕하세요";
String content = box.content;

Box<Integer> box1 = new Box<Integer>();
box.content = 100;
int content = box.content;
```
> ⚠️ <>안에는 int, double과 같은 기본 타입이 아니라 오직 클래스와 인터페이스   타입만 삽입될 수 있다는 점에 주의해야 합니다. Box&lt;int&gt; 안됩니다.


#### 제네릭 기본 실습 

* Box1.java
```java
public class Box1<T> {
    public T content; // 타입 파라미터 T
}
```
* GenericExample1.java
```java
public class GenericExample1 {
    public static void main(String[] args) {
        Box1<String> box1 = new Box1<>();
        box1.content = "안녕하세요";
        String str = box1.content;
        System.out.println(str);

        Box1<Integer> box2 = new Box1<>();
        box2.content = 100;
        int value = box2.content;
        System.out.println(value);
    }
}
```


#### 제네릭 종류

* 제네릭 타입: 결정되지 않은 타입을 파라미터로 가지는 클래스와 인터페이스(위의 Box1.class)
  ```java
  public class 클래스명<A, B, ...> {...}
  public interface 인터페이스명<A, B, ...> {...}
  ```
* 제네릭 메소드: 타입 파라미터를 가지고 있는 메소드로 타입 파라미터가 메소드 선언부에 정의된다는 점에서 제네릭 타입과 차이가 있습니다. 제네릭 메소드는 리턴 타입 앞에 < > 기호를 추가하고 타입 파라미터를 정의한 뒤, 리턴 타입과 매개변수 타입에서 사용합니다.
  ```java
  public <T> Box<T> boxing(T t) {...}
  ```
* 제한된 타입 파라미터: 경우에 따라서는 타입 파라미터를 제한하는 구체적인 타입을 제한할 필요가 있습니다. 예를 들어 숫자를 연산하는 제네릭 메소드의 경우 타입 파라미터를 Number로 제한할 필요가 있습니다.
  ```java
  public <T extends Number> boolean compare(T t1, T t2) {...}
  ```


### 컬렉션: *Collections*


---


#### 컬렉션 프레임워크
자바는 널리 알려져 있는 자료구조(배열, 집합, 딕셔너리, 스택, 큐 등)를 바탕으로 객체들을 효율적으로 추가, 삭제, 검색할 수 있도록 관련된 인터페이스와 클래스들을 java.util 패키지에 포함시켜 놓았습니다. 이들을 총칭해서 컬렉션 프레임워크라고 부릅니다.
![Alt text](<image/Screenshot 2024-03-29 at 10.38.11 AM.png>)
지금 아기 사자들이 기억해두었으면 하는 자료구조로는 List, Set, Map 3가지면 충분합니다. 그중에서도 ArrayList, HashSet, HashMap 구현 클래스만 알면 됩니다.
![Alt text](<image/Screenshot 2024-03-29 at 10.38.31 AM.png>)


#### List 컬렉션
List 컬렉션은 객체를 인덱스로 관리하기 때문에 객체를 저장하면 인덱스가 부여되고 인덱스로 객체를 검색 삭제할 수 있는 기능을 제공합니다.

>✅ **ArrayList**: List의 대표적인 구현체로 객체를 추가하면 내부 배열에 객체가 저장됩니다. 일반 배열과의 차이점은 제한 없이 객체를 추가할 수 있다는 점입니다.

* Board.java
```java
public class Board {
    public String subject;
    public String content;
    public String writer;

    public Board(String subject, String content, String writer) {
        this.subject = subject;
        this.content = content;
        this.writer = writer;
    }
}
```

* ArrayListExample.java
```java
import java.util.ArrayList;
import java.util.List;

public class ArrayListExample {
    public static void main(String[] args) {
        // ArrayList 컬렉션 생성
        List<Board> list = new ArrayList<>();

        // 객체 추가
        list.add(new Board("멋사 12기", "조지미1", "정다연"));
        list.add(new Board("멋사 12기", "조지미2", "한장수"));
        list.add(new Board("멋사 12기", "조지미3", "조서윤"));
        list.add(new Board("멋사 12기", "조지미4", "오동재"));

        // 저장된 총 객체 수 얻기
        int size = list.size();
        System.out.println("총 객체 수: " + size);
        System.out.println();

        // 특정 인덱스의 객체 가져오기
        Board board = list.get(2);
        System.out.println(board.subject + " " + board.content + " " + board.writer);
        System.out.println();

        // 모든 객체를 하나씩 가져오기
        for (int i = 0; i < list.size(); i++) {
            Board b = list.get(i);
            System.out.println(b.subject + " " + b.content + " " + b.writer);
        }
        System.out.println();

        // 마음에 안드는 조지미들 삭제
        list.remove(2);
        list.remove(2);

        for (Board b : list) {
            System.out.println(b.subject + " " + b.content + " " + b.writer);
        }
    }
}
```

#### Set 컬렉션
Set 컬렉션은 List와 달리 저장 순서가 유지되지 않습니다. 또한 객체를 중복해서 저장할 수 없고, 하나의 null만 저장할 수 있습니다. Set 컬렉션은 수학의 집합에 비유될 수 있습니다.

> ✅ **HashSet**: Set 인터페이스의 구현 클래스로 동일한 객체를 중복 저장하지 않는다. 여기서 동일한 객체는 참조값이 같은 객체 뿐만 아니라 hashCode() 메소드의 리턴값이 같고, equals() 메소드가 true를 리턴하면 동일한 객체라고 판단하고 중복 저장하지 않는다.

* HashSetExample.java
```java
import java.util.*;

public class HashSetExample {
    public static void main(String[] args) {
        // HashSet 컬렉션 생성
        Set<String> set = new HashSet<String>();

        // 객체 저장
        set.add("오동재");
        set.add("주영빈");
        set.add("김신희");
        set.add("오동재"); // 중복 객체이므로 저장되지 않음
        set.add("최기웅");
        set.add("안준영");

        // 저장된 객체 수 출력
        int size = set.size();
        System.out.println("총 백엔드 운영진 수: " + size);
    }
}
```


#### Map 컬렉션
Map 컬렉션은 키와 값으로 구성된 엔트리 객체를 저장합니다. 여기서 키와 값은 모두 객체입니다. 키는 중복 저장할 수 없지만 값은 중복 저장이 가능합니다. 기존에 저장된 키와 동일한 키로 값을 저장하면 기존의 값은 없어지고 새로운 값으로 저장됩니다. 쉽게 말해 덮어쓰기의 과정을 거치는 것이라고 볼 수 있습니다.

> ✅ **HashMap**: Map 인터페이스의 대표적인 구현 클래스로 키의 경우 HashSet과 같이 hashCode() 메소드의 리턴값이 같고 equals() 메소드가 true를 리턴할 경우, 동일 키로 보고 중복 저장을 허용하지 않습니다.

* HashMapExample.java
```java
import java.util.*;

public class HashMapExample {
    public static void main(String[] args) {
        // Map 컬렉션 생성
        Map<String, Integer> map = new HashMap<>();

        map.put("오동재", 4);
        map.put("조서윤", 1);
        map.put("한장수", 1);
        map.put("정다연", 4);
        System.out.println("총 조지미 수: " + map.size());
        System.out.println();

        String key = "한장수";
        int value = map.get(key);
        System.out.println("이름:" + key + " 학년:" + value);
        System.out.println();

        Set<String> keySet = map.keySet();
        for (String name : keySet) {
            int v = map.get(name);
            System.out.println(name + " : " + v);
        }
        System.out.println();

        // 키로 엔트리 삭제
        map.remove("한장수");
        System.out.println("총 조지미 수: " + map.size());
        System.out.println();
    }
}

```