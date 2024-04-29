# 학생 관리 API 설계

### Spring Framework

---

#### 정의

Spring Framework는 다형성을, 즉 동일한 코드로 같은 결과를 내는 객체 지향 프로그래밍의 가장 큰 특징을, 극대화하여 구성한 Java 기반의 프레임워크이다. Spring이라는 단어는 문맥에 따라 Spring Framework만을 지칭하기도 하고 이에 활용되는 IoC, DI 컨테이너 기술, 또는 다양한 Projects를 지칭하는 스프링 생태계 자체를 의미하기도 한다.

![alt text](<./image/Screenshot 2024-04-29 at 9.46.10 PM.png>)

#### IoC와 DI 컨테이너

IoC와 DI는 Spring Framework에서 가장 핵심적인 기술로, Java의 인터페이스만으로는 다형성을 완벽하게 구현하지 못하는 한계를 극복하게 해주는 기술이다. 인터페이스를 구현하는 클래스의 객체를 생성하는 과정에서 클래스의 종류를 개발자가 직접 선택해야 한다. Spring의 IoC와 DI 컨테이너 기술을 사용하게되면 Bean이라는 객체가 Spring Container에 등록되어 프레임워크 차원에서 자동으로 생성되고 관리된다. `@Component`, `@Service`, `@Repository`, `@Controller`와 같은 어노테이션이 붙은 클래스는 자동으로 Spring Bean으로 등록된다. 이처럼 객체의 생성과 관리를 프레임워크에 위임하는 것을 코드의 제어가 역전되었다하여 **Inversion of Control(IoC), 제어의 역전**이라고 한다.

![alt text](<./image/Screenshot 2024-04-29 at 9.47.16 PM.png>)

또한 등록된 객체들은 서로에게 필요한 관계에 놓여있는 경우 생성자에 `@Autowired` 키워드만 붙어 있다면, 필요한 객체를 해당 Container에서 꺼내어 사용하게끔 해준다. 이처럼 필요한 의존관계를 외부에서 찾아 넣어주는 것을 **Dependency Injection, 의존성 주입**이라고 한다.

> IoC와 DI에 대해 구체적으로 실제 코드를 통해 이해하고 싶다면 인프런 김영한님의 스프링 핵심 원리 기본편을 수강할 것을 추천 - [스프링핵심원리-기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)

![alt text](<./image/Screenshot 2024-04-29 at 9.47.23 PM.png>)

### Spring Web 계층

---

![alt text](<./image/Screenshot 2024-04-29 at 4.32.48 PM.png>)

* **Web Layer** -  외부 요청과 응답에 대한 처리를 진행하는 역할을 한다.
* **Service Layer** - 비즈니스 로직이 수행되는 영역이다.
* **Repository Layer** - 데이터 저장소와 직접 연결되는 영역이다.
* **DTOs** - 계층 간 데이터 이동에 사용되는 운반 역할을 한다.
* **Domain Model** - 개발 대상을 표현하는 모델이다.  

### 프로젝트 설정

---

![alt text](<./image/Screenshot 2024-04-29 at 5.01.53 PM.png>)

![alt text](<./image/Screenshot 2024-04-29 at 5.02.17 PM.png>)
  
* Spring Boot 3.2.5
* Spring Web
* Lombok

build.gradle

```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.5'
    id 'io.spring.dependency-management' version '1.1.4'
}

group = 'org.likelion'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

### Domain Model & Repository Layer

---

**Domain Model**이란 개발 대상을 모든 사람이 동일한 관점에서 이해하고 공유할 수 있도록 단순화 시킨 것이다. 예를 들어 수강 신청 서비스에서 학생, 학과, 강좌, 수강 등과 같은 것이 도메인이 될 수 있으며 주로 데이터베이스와 연동될 시 테이블과 매칭되어 Entity 모델이라고 하기도 한다.

**Repository Layer**란 데이터 저장소에 접근하는 영역을 의미한다. Spring 계층에서 데이터베이스와 가장 근접하게 연관되어 있는 영역이다.

**Student.java**
```java
package org.likelion.firstspringproject.domain;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter // 1
@NoArgsConstructor // 2
public class Student { // entity 클래스라고 하기도 한다.

    private Long id; // id 값은 객체별로 유일해야하고 변경이 불가해야 한다.
    private Long studentId;
    private String name;
    private String major

    @Builder // 3
    public Student(Long id, Long studentId, String name, String major) {
        this.studentId = studentId;
        this.id = id;
        this.name = name;
        this.major = major;
    }

    /*
    * entity 클래스에서는 setter 대신에 명확히 그 목적과 의도를 나타낼 수 
    * 있는 메소드를 추가한다.
    */
    public void update(String name, String major) {
        this.name = name;
        this.major = major;
    }

}
```
1. `@Getter`
   * 클래스 내 모든 필드의 Getter 메소드를 생성한다.
2. `@NoArgsConstructor`
   * 기본 생성자 자동 추가
   * `public Student() {}`와 같은 효과
3. `@Builder`
   * 해당 클래스의 빌더 패턴 클래스를 생성
   * 생성자 상단에 선언 시 생성자에 포함된 필드만 빌더에 포함
   * 생성자와 같은 역할을 하지만 어느 필드에 어떤 값을 채워야 할 지 명확하게 인지할 수 있다.



**StudentRepository.java**

해당 프로젝트에서는 실제 데이터베이스와 연동하는 대신 인메모리 형태의 간단한 HashMap 데이터베이스를 생성하여 작업을 수행한다.

```java
package org.likelion.firstspringproject.domain;

import org.springframework.stereotype.Repository;

import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;

@Repository // 1
public class StudentRepository {
    private static final Map<Long, Student> database = new HashMap<>();

    public Student save(Student student) {
        database.put(student.getId(), student);
        return database.get(student.getId());
    }

    // 2
    public Optional<Student> findById(Long id) {
        return Optional.ofNullable(database.get(id));
    }

    public List<Student> findAll() {
        return database.values().stream().toList();
    }

    public void deleteById(Long id) {
        database.remove(id);
    }

    public void clear() {
        database.clear();
    }
}
```
1. `@Repository`
   * 데이터베이스와 가장 밀접하게 연관된 클래스로 수행될 쿼리를 지정한다. 여기서는 실제 DB와 연동하지 않았기 때문에 쿼리가 수행되지는 않는다.
   * 해당 어노테이션이 붙으면 Spring Bean으로 자동 등록 된다.
2. `Optional`
    *  java는 기본적으로 런타임 환경에서 null 값을 참조할 수도 있는 위험성이 있는데, null 값 참조에 대한 안정성을 컴파일 단계에서 보장해 주는 클래스가 Optional 클래스이다. (Null Safety)
    *  `.isPresent()` : 현재 값이 null이 아닐 경우 true를 반환
    *  `.orElseThrow()` : null값일 경우 처리할 예외를 설정. 추후 자주 등장 예정.
  

### DTOs

---
***DTO*란 Data Transfer Object의 약자로 계층간 데이터 교환을 위한 클래스이다.** Web Layer에서 Service Layer로 데이터를 넘겨줄 때와 데이터베이스에서 조회한 데이터를 Web Layer로 넘겨줄 때 주로 사용한다. **Entity 클래스는 데이터베이스와 맞닿은 핵심 클래스이기 때문에 절대 해당 클래스를 요청과 응답용으로 사용해서는 안된다.** 예를 들어 Entity 클래스를 기준으로 테이블과 스키마가 생성되는데 화면 변경과 같은 사소한 기능 변경을 위해 Entity 클래스를 변경하는 것은 너무 큰 변경이다. 

**StudentSaveRequestDto.java**
```java
package org.likelion.firstspringproject.dto;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class StudentSaveRequestDto {

    private Long studentId;
    private String name;
    private String major;

    @Builder
    public StudentSaveRequestDto(Long studentId, String name, String major) {
        this.studentId = studentId;
        this.name = name;
        this.major = major;
    }

}
```

**StudentUpdateRequestDto.java**

```java
package org.likelion.firstspringproject.dto;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class StudentUpdateRequestDto {
    private String name;
    private String major;

    @Builder
    public StudentUpdateRequestDto(String name, String major) {
        this.name = name;
        this.major = major;
    }
}
```

**StudentResponseDto.java**
```java
package org.likelion.firstspringproject.dto;

import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import org.likelion.firstspringproject.domain.Student;

@Getter
@AllArgsConstructor(access = AccessLevel.PRIVATE) // 1
@NoArgsConstructor
public class StudentResponseDto {

    private Long id;
    private Long studentId;
    private String name;
    private String major;

    // 2
    public static StudentResponseDto from(Student student) {
        return new StudentResponseDto(student.getId(), student.getStudentId(), student.getName(), student.getMajor());
    }
}
```
1. `@AllArgsConstructor`
   * 모든 필드를 포함하는 생성자를 만들어준다.
   * 접근제한자를 private으로 설정한다.
2. 정적 팩토리 메소드 패턴
   * 객체를 생성할 때 new 연산자로 직접 객체를 생성하는 대신에 메소드를 이용하여 한 단계 더 거쳐 객체를 생성하는 패턴이다. 해당 패턴을 사용하면 더 가독성 좋은 코드를 작성하고 객체 지향적으로 프로그래밍을 할 수 있다.
  > 정적 팩토리 메소드 패턴 참고 자료 
  https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EC%A0%95%EC%A0%81-%ED%8C%A9%ED%86%A0%EB%A6%AC-%EB%A9%94%EC%84%9C%EB%93%9C-%EC%83%9D%EC%84%B1%EC%9E%90-%EB%8C%80%EC%8B%A0-%EC%82%AC%EC%9A%A9%ED%95%98%EC%9E%90

### Service Layer

---

**SudentService.java**

```java
package org.likelion.firstspringproject.service;

import lombok.RequiredArgsConstructor;
import org.likelion.firstspringproject.domain.Student;
import org.likelion.firstspringproject.domain.StudentRepository;
import org.likelion.firstspringproject.dto.StudentResponseDto;
import org.likelion.firstspringproject.dto.StudentSaveRequestDto;
import org.likelion.firstspringproject.dto.StudentUpdateRequestDto;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@RequiredArgsConstructor // 1
@Service // 2
public class StudentService {

    private final StudentRepository studentRepository; // 의존성 주입

    private static Long sequence = 0L; // 3

    public Student saveStudent(StudentSaveRequestDto requestDto) {
        Student student = Student.builder() //4
                .id(++sequence)
                .studentId(requestDto.getStudentId())
                .name(requestDto.getName())
                .major(requestDto.getMajor())
                .build();

        return studentRepository.save(student);
    }

    public StudentResponseDto findStudentById(Long id) {
        Student student = studentRepository.findById(id)
                .orElseThrow(()-> new IllegalArgumentException("해당하는 학생이 없습니다. id = " + id));

        return StudentResponseDto.from(student);
    }

    public List<StudentResponseDto> findAllStudent() {
        return studentRepository.findAll().stream()
                .map(StudentResponseDto::from) // 5
                .collect(Collectors.toList());
    }

    public Student updateStudentById(Long id, StudentUpdateRequestDto requestDto) {
        // 해당 id를 가지는 객체를 반환
        Student student = studentRepository.findById(id)
                .orElseThrow(()-> new IllegalArgumentException("해당하는 학생이 없습니다. id = " + id));

        // 해당 객체의 내용을 수정
        student.update(requestDto.getName(), requestDto.getMajor());

        // 변경된 내용을 DB에 반영
        return studentRepository.save(student);
    }

    public void deleteStudentById(Long id) {
        studentRepository.deleteById(id);
    }
}
```

1. `@RequiredArgsConstructor`
   * `final` 키워드가 붙은 필드를 포함하는 생성자를 롬복이 대신 만들어준다.
   * Spring에서 Bean을 주입받을 때(DI) 가장 권장하는 방식이 생성자로 주입받는 방식이다. 객체가 생성되는 시점에 의존관계를 불변하게 설정하는 것이다. `@Autowired`는 필드 주입으로 이를 사용하면 실행 도중 의존 관계가 변할 가능성이 있기 때문에 사용이 권장되지 않는다.
   * 해당 클래스의 의존관계가 변경될 때마다 생성자 코드를 수정하지 않아도 된다.
2. `@Service`
   * 해당 어노테이션을 사용하면 Spring Bean으로 자동 등록된다.
   * 비즈니스 로직이 구현되는 곳이다.
   * `@Transactional`로 데이터베이스의 트랜잭션을 보장한다.
3. `sequence`
   * 객체 생성과 동시에 값이 하나씩 증가하는 정적 필드이다.
   * 도메인 객체가 생성되는 service layer에서 객체별로 unique 해야 하는 id값을 지정해준다.
4. `@Builder`
   * 기본 생성자와 다르게 빌더를 사용하게 되면 다음과 같이 어느 필드에 어떤 값을 채워야할 지 명확하게 인지할 수 있다. `Example.builder().a(a).b(b).build();`
5. `.map(StudentResponseDto::from)`
   * 스트림의 요소가 매개변수의 전달 역할만 할 때 메소드 참조형식으로 작성할 수 있다.

### Web Layer

---

Controller 클래스가 위치하는 영역으로 필터(`@Filter`), 인터셉터, `@ControllerAdvice` 등이 있으며, 외부 요청과 응답에 대한 처리를 진행하는 역할을 한다.

**StudentController.java**

```java
package org.likelion.firstspringproject.controller;

import lombok.RequiredArgsConstructor;
import org.likelion.firstspringproject.dto.StudentResponseDto;
import org.likelion.firstspringproject.dto.StudentSaveRequestDto;
import org.likelion.firstspringproject.dto.StudentUpdateRequestDto;
import org.likelion.firstspringproject.service.StudentService;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RequiredArgsConstructor
@RestController // 1
public class StudentController {

    private final StudentService studentService;

    @PostMapping("students") 
    public void save(@RequestBody StudentSaveRequestDto requestDto) { // 2
        studentService.saveStudent(requestDto);
    }

    @GetMapping("students/{id}")
    public StudentResponseDto findStudentById(@PathVariable Long id) { // 3
        return studentService.findStudentById(id);
    }

    @GetMapping("students")
    public List<StudentResponseDto> findAllStudent() {
        return studentService.findAllStudent();
    }

    @PatchMapping("students/{id}")
    public void updateStudentById(@PathVariable Long id, @RequestBody StudentUpdateRequestDto requestDto) {
        studentService.updateStudentById(id, requestDto);
    }

    @DeleteMapping("students/{id}")
    public void deleteStudentById(@PathVariable Long id) {
        studentService.deleteStudentById(id);
    }

}
```
`@GetMapping`, `@PostMapping`, `@PatchMapping`, `@DeleteMapping` 모두 HTTP 메서드 GET, POST, PATCH, DELETE에 해당하는 요청을 받을 수 있는 API를 만들어준다.

1. `@RestController`
   * 컨트롤러를 JSON을 반환하는 컨트롤러로 만들어 준다.
2. `@RequestBody`
   * 요청 메시지의 message body를 자바 객체로 변환하여 준다.
3. `@PathVariable`
   * http://localhost:8080/students/1 과 같이 요청이 들어오면 경로에 존재하는 값을 변수로 사용하여 처리할 수 있다.
 
### Test Code

---

**StudentFixture.java**

```java
package org.likelion.firstspringproject.fixture;

import org.likelion.firstspringproject.domain.Student;

public class StudentFixture {

    public static final Student STUDENT_1 = new Student(
            1L,
            201914089L,
            "오동재",
            "소프트웨어공학전공"
    );

    public static final Student STUDENT_2 = new Student(
            2L,
            202012345L,
            "안준영",
            "정보통신공학전공"
    );


    public static final Student STUDENT_3 = new Student(
            3L,
            202100000L,
            "주영빈",
            "컴퓨터공학전공"
    );
}
```

**StudentControllerTest.java**

```java
package org.likelion.firstspringproject.controller;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.likelion.firstspringproject.domain.Student;
import org.likelion.firstspringproject.domain.StudentRepository;
import org.likelion.firstspringproject.dto.StudentResponseDto;
import org.likelion.firstspringproject.dto.StudentSaveRequestDto;
import org.likelion.firstspringproject.dto.StudentUpdateRequestDto;
import org.likelion.firstspringproject.service.StudentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.server.LocalServerPort;
import org.springframework.http.HttpStatus;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.filter.CharacterEncodingFilter;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;
import static org.likelion.firstspringproject.fixture.StudentFixture.*;

@ExtendWith(SpringExtension.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
public class StudentControllerTest {

    @LocalServerPort
    private int port;

    @Autowired
    private WebApplicationContext context;

    @Autowired
    private MockMvc mockMvc;

    @BeforeEach
    public void setup() {
        mockMvc = MockMvcBuilders.webAppContextSetup(context)
                .addFilter(new CharacterEncodingFilter("UTF-8", true))  // 응답 인코딩을 UTF-8로 강제합니다.
                .build();
    }

    @Autowired
    ObjectMapper objectMapper;

    @Autowired
    StudentService studentService;

    @Autowired
    StudentRepository studentRepository;

    @AfterEach
    public void afterEach() {
        studentRepository.clear();
    }

    @Test
    @DisplayName("학생을 정상 저장한다")
    public void saveStudent() throws Exception {
        // given
        final StudentSaveRequestDto requestDto = StudentSaveRequestDto.builder()
                .studentId(STUDENT_1.getStudentId())
                .name(STUDENT_1.getName())
                .major(STUDENT_1.getMajor())
                .build();

        String url = "http://localhost:"+ port + "/students";

        // when
        MvcResult mvcResult = mockMvc.perform(MockMvcRequestBuilders.post(url)
                        .content(asJsonString(requestDto))
                        .contentType("application/json"))
                .andReturn();

        // then
        assertThat(mvcResult.getResponse().getStatus()).isEqualTo(HttpStatus.OK.value());

        List<Student> all = studentRepository.findAll();
        assertThat(all.get(0).getStudentId()).isEqualTo(STUDENT_1.getStudentId());
        assertThat(all.get(0).getName()).isEqualTo(STUDENT_1.getName());
        assertThat(all.get(0).getMajor()).isEqualTo(STUDENT_1.getMajor());
    }

    @Test
    @DisplayName("학생을 정상 조회한다")
    public void getStudent() throws Exception {
        // given
        studentRepository.save(STUDENT_2);

        String url = "http://localhost:" + port + "/students/" + STUDENT_2.getId();

        // when
        MvcResult mvcResult = mockMvc.perform(MockMvcRequestBuilders.get(url)).andReturn();

        // then
        final StudentResponseDto studentResponse = objectMapper.readValue(
                mvcResult.getResponse().getContentAsString(),
                new TypeReference<>() {
                }
        );

        assertThat(mvcResult.getResponse().getStatus()).isEqualTo(HttpStatus.OK.value());

        assertThat(studentResponse.getStudentId()).isEqualTo(STUDENT_2.getStudentId());
        assertThat(studentResponse.getName()).isEqualTo(STUDENT_2.getName());
        assertThat(studentResponse.getMajor()).isEqualTo(STUDENT_2.getMajor());
    }

    @Test
    @DisplayName("학생 전체를 정상 조회한다")
    public void getAllStudent() throws Exception {
        // given
        studentRepository.save(STUDENT_1);
        studentRepository.save(STUDENT_2);
        studentRepository.save(STUDENT_3);

        String url = "http://localhost:" + port + "/students";

        // when
        MvcResult mvcResult = mockMvc.perform(MockMvcRequestBuilders.get(url)).andReturn();

        // then
        final List<StudentResponseDto> actualResponses = objectMapper.readValue(
                mvcResult.getResponse().getContentAsString(),
                new TypeReference<>() {
                }
        );

        List<StudentResponseDto> expectedResponses = studentService.findAllStudent();
        assertThat(actualResponses).usingRecursiveComparison().isEqualTo(expectedResponses);
    }

    @Test
    @DisplayName("학생을 정상 수정한다")
    public void updateStudent() throws Exception {
        // given
        final Student savedStudent = studentRepository.save(STUDENT_1);

        Long updateId = savedStudent.getId();
        String newName = "주영빈";
        String newMajor = "컴퓨터공학전공";

        StudentUpdateRequestDto requestDto = StudentUpdateRequestDto.builder()
                .name(newName)
                .major(newMajor)
                .build();

        String url = "http://localhost:" + port + "/students/" + updateId;

        // when
        MvcResult mvcResult = mockMvc.perform(MockMvcRequestBuilders.patch(url)
                        .content(asJsonString(requestDto))
                        .contentType("application/json"))
                .andReturn();

        // then
        assertThat(mvcResult.getResponse().getStatus()).isEqualTo(HttpStatus.OK.value());

        List<Student> all = studentRepository.findAll();
        assertThat(all.get(0).getName()).isEqualTo(newName);
        assertThat(all.get(0).getMajor()).isEqualTo(newMajor);
    }

    @Test
    @DisplayName("학생을 정상 삭제한다")
    public void deleteStudent() throws Exception {
        // given
        final Student savedStudent = studentRepository.save(STUDENT_1);

        String url = "http://localhost:" + port + "/students/" + savedStudent.getId();

        // when
        MvcResult mvcResult = mockMvc.perform(MockMvcRequestBuilders.delete(url)).andReturn();

        // then
        assertThat(mvcResult.getResponse().getStatus()).isEqualTo(HttpStatus.OK.value());

        List<Student> all = studentRepository.findAll();
        assertThat(all.size()).isEqualTo(0);
    }

    private String asJsonString(final Object obj) {
        try {
            return new ObjectMapper().writeValueAsString(obj);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

}
```


