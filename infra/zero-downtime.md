# 무중단 배포

## 1. ProfileController

무중단 배포를 구축하기 위해서는 두 개의 포트에서 각각 스프링 부트 애플리케이션이 구동되고 있어야 합니다. 현재 서비스 중인 포트를 확인하기 위해 API를 하나 추가하도록 하겠습니다.

<br>

먼저 현재 프로젝트 코드로 돌아와서 controller 패키지에 `ProfileController.java`를 생성합니다.

<br>

```java
package LESSERAFIM.lesserafim.controller;

import org.springframework.core.env.Environment;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Arrays;
import java.util.List;

@RestController
public class ProfileController {
    
    private final Environment env;

    public ProfileController(Environment env) {
        this.env = env;
    }

    @GetMapping("/profile")
    public String profile() {
        List<String> profiles = Arrays.asList(env.getActiveProfiles());
        List<String> realProfiles = Arrays.asList("real1", "real2");
        String defaultProfile = profiles.isEmpty() ? "default" : profiles.get(0);

        return profiles.stream()
                .filter(realProfiles::contains)
                .findAny()
                .orElse(defaultProfile);
    }
}

```

만약 Spring Security를 사용중이라면 `/profile`이 인증 없이 호출될 수 있도록 제외 코드를 추가합니다.

<br>

다음으로 resources 폴더 안에 `application-real1.yml`과 `application-real2.yml` 파일을 생성하고 다음과 같이 작성합니다.

```yml
# application-real1.yml
server:
    port: 8081
```
```yml
# application-real2.yml
server:
    port: 8082
```

<br>

## 2. 배포 스크립트 작성



