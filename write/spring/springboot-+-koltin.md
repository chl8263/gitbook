# Springboot + Koltin 로그설정

 `spring-boot-starter`의 dependency tree를 확인해보면 `spring-boot-starter-logging`이 포함되어 있고, 그 안에 다시 `slf4j`와 `logback-classic`이 자동으로 포함되어 있다. 따라서 별도 dependency를 추가하지 않아도 Logger를 사용할 수 있지만, Kotlin 기반의 Spring Boot 어플리케이션에서는 SLF4J를 wrapping한 `kotlin-logging`이라는 경량 프레임워크를 통해서 조금 더 쉽고 효율적으로 Logger를 사용할 수 있다.

## 1. kotlin-logging dependency 추가

}

```text
dependencies {
    implementation("org.springframework.boot:spring-boot-starter")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
    implementation("io.github.microutils:kotlin-logging:1.12.5") // Logging
​
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}
```

