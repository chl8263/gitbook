# Springboot 프로젝트 bootjar layered 나누기

## 기존방식의 문제점

Spring boot를 활용하여서 도커 이미지를 만들 때 기존에는 아래와과 같은 `Dockerfile` 을 사용하여 간단하게 spring boot jar 파일을 도커 이미지로 만들 수 있다.

```bash
FROM openjdk:11-jdk as builder
ENV	APP_HOME=/usr/app/
WORKDIR $APP_HOME
COPY build/libs/*.jar application.jar
EXPOSE 8080
CMD ["java", "-jar", "application.jar"]
```

`하지만 jar 파일이 무거워 지는 경우 docker image를 만드는 것이 비효율이 발생`한다. 

도커는 레이어마다 캐쉬를 활용할 수 있고 그것을 통해서 빠른 docker 이미지를 만들 수 있는 장점이 존재한다. 이런 구조에서는 자바의 모든 구조가 jar 파일로 되는 것이기 때문에 캐쉬를 적용하기가 어렵다. 

\(소스 코드 한 줄이 바뀌더라도 캐쉬가 깨지기 때문에 다시 연산을 해야한다.\)

## 개선방식 \(Layered\)

 Spring을 이용해서 Jar 파일을 만들 때 4개의 영역으로 구분되어 지도록 만들 수 있다.

아래 방법으로 build.gradle 에 추가하여 Jar 파일을 만들고 밑에 방식을 통해서 jar 파일을 풀게 되면 4가지 폴더\(layer\)가 만들어 진다.

### 1. build gradle 에 아래 코드 추

```kotlin
tasks.getByName<BootJar>("bootJar") {
	layered {
		application {
			intoLayer("spring-boot-loader") {
				include("org/springframework/boot/loader/**")
			}
			intoLayer("application")
		}
		dependencies {
			intoLayer("snapshot-dependencies") {
				include("*:*:*SNAPSHOT")
			}
			intoLayer("dependencies")
		}
		layerOrder = listOf("dependencies", "spring-boot-loader", "snapshot-dependencies", "application")
	}
}
```

#### 자세한 Layered 설정은 아래 공식 문서를 참조하라.

{% embed url="https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/htmlsingle/\#packaging-executable.configuring.layered-archives" %}



| folder name | description\(공식문서 설명\) |
| :--- | :--- |
| application | application classes and resources |
| snapshot-dependencies | any dependency whose version contains SNAPSHOT |
| spring-boot-loader | the jar loader classes |
| dependencies | any dependency whose version does not contain SNAPSHOT |

정확하게 이렇게 4가지로 나눠지는 이유는 적혀있지 않았지만 application layer부터 밑으로 자주 바뀌지 않은 순서가 존재한다는 것이다. \(application이 제일 자주 바뀌고 dependiencies가 제일 바뀌지 않는다\)

그래서 실제로 docker image를 만들어 가는 과정에서 이 순서에 역순으로 작성을 해주어야 한다. \(자주 바뀌지 않기 때문에 캐쉬가 깨질 가능성이 적어진다.\)

### 2. Dockerfile 에 아래 코드 추

```bash
FROM openjdk:11-jdk as builder
ENV APP_HOME=/usr/app
WORKDIR $APP_HOME
COPY build/libs/demo-0.0.1-SNAPSHOT.jar application.jar
RUN java -Djarmode=layertools -jar application.jar extract

FROM openjdk:11-jdk
ENV APP_HOME=/usr/app
WORKDIR $APP_HOME
ENV spring.profiles.active local
COPY --from=builder /usr/app/dependencies/ ./
COPY --from=builder /usr/app/spring-boot-loader/ ./
COPY --from=builder /usr/app/snapshot-dependencies/ ./
COPY --from=builder /usr/app/application/ ./
EXPOSE 8080

ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

### 3. Docker build and run

```text
# docker build -t YOUR_IMAGE_NAME

# docker run -p 9445:8080 chl8263/demo-springboot
```

