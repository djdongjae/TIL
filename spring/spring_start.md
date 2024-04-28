# 학생 관리 API 설계

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

domain/Student.java
```java
package org.likelion.firstspringproject.domain;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class Student {

    private Long id;
    private Long studentId;
    private String name;
    private String major;

    @Builder
    public Student(Long id, Long studentId, String name, String major) {
        this.studentId = studentId;
        this.id = id;
        this.name = name;
        this.major = major;
    }

    /*
     * HashMap에 저장할 때 key 값으로 객체의 id값을 사용하기 때문에 id값은 객체별로 유일해야 하고
     * 변경이 불가해야 한다.
     */

    public void update(String name, String major) {
        this.name = name;
        this.major = major;
    }

}
```

domain/StudentRepository.java
```java
package org.likelion.firstspringproject.domain;

import org.springframework.stereotype.Repository;

import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;

@Repository
public class StudentRepository {
    private static final Map<Long, Student> database = new HashMap<>();

    public Student save(Student student) {
        database.put(student.getId(), student);
        return database.get(student.getId());
    }

    public Optional<Student> findById(Long id) {
        return Optional.ofNullable(database.get(id));
    }

    public List<Student> findAll() {
        return database.values().stream().toList();
    }

    public void deleteById(Long id) {
        database.remove(id);
    }

    public void clearStore() {
        database.clear();
    }
}
```
### DTOs

---

dto/StudentSaveRequestDto.java
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

dto/StudentUpdateRequestDto.java
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

dto/StudentResponseDto.java
```java
package org.likelion.firstspringproject.dto;

import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.Getter;

@Getter
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class StudentResponseDto {

    private Long id;
    private Long studentId;
    private String name;
    private String major;

    public static StudentResponseDto of(Long id, Long studentId, String name, String major) {
        return new StudentResponseDto(id, studentId, name, major);
    }

}
```

### Service Layer

---

service/SudentService.java

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

@RequiredArgsConstructor
@Service
public class StudentService {
    private final StudentRepository studentRepository;

    // Service 클래스 또한 싱글톤 객체(유일함)를 생성하므로
    // sequence 필드를 service layer에 위치해도 무관하다. (즉 sequence도 유일)
    // 따라서 도메인 객체가 생성되는 service layer에서 객체별로 unique 해야 하는 id값을 지정해준다.
    private static Long sequence = 0L;

    public Student saveStudent(StudentSaveRequestDto requestDto) {
        Student student = Student.builder()
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

        return StudentResponseDto.of(student.getId(), student.getStudentId(), student.getName(), student.getMajor());
    }

    public List<StudentResponseDto> findAllStudent() {
        return studentRepository.findAll().stream()
                .map(student -> StudentResponseDto.of(
                        student.getId(),
                        student.getStudentId(),
                        student.getName(),
                        student.getMajor()
                ))
                .collect(Collectors.toList());
    }

    /*
     * 수정을 해도 참조하는 객체를 변경하지 않는 방식으로 설계.
     */
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

### Web Layer

---

controller/StudentController.java
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
@RestController
public class StudentController {

    private final StudentService studentService;

    @PostMapping("students")
    public void save(@RequestBody StudentSaveRequestDto requestDto) {
        studentService.saveStudent(requestDto);
    }

    @GetMapping("students/{id}")
    public StudentResponseDto findStudentById(@PathVariable Long id) {
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

### Test Code

---
fixture/StudentFixture.java

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

import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.likelion.firstspringproject.domain.Student;
import org.likelion.firstspringproject.domain.StudentRepository;
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

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;
import static org.likelion.firstspringproject.fixture.StudentFixture.STUDENT_1;

@ExtendWith(SpringExtension.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
public class StudentControllerTest {

    @LocalServerPort
    private int port;

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    StudentService studentService;

    @Autowired
    StudentRepository studentRepository;

    @AfterEach
    public void afterEach() {
        studentRepository.clearStore();
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

    private String asJsonString(final Object obj) {
        try {
            return new ObjectMapper().writeValueAsString(obj);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

}
```


