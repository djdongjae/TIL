# 학생 관리 API 설계

### 배경 지식

---

#### Spring Framework

#### 싱글톤 패턴

#### IoC와 DI

### 프로젝트 구조

---

### 프로젝트 설정

---

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

### Domain Layer

---

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

controller/StudentControllerTest.java

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


