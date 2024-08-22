# ComfigMap vs Secret

`ConfigMap`과 `Secret`은 애플리케이션의 설정 데이터와 보안 정보를 관리하는 데 사용된다.

둘 사이의 주요 차이점은 데이터의 보안과 민감성에 있다.

## ConfigMap

`ConfigMap`은 애플리케이션의 비밀이 아닌 구성 데이터를 저장하고 관리하는 데 사용된다. 일반적으로 애플리케이션의 설정 값이나 환경 변수를 포함한다.

**주요 특징**

* **목적**: 애플리케이션 설정이나 비밀이 아닌 데이터를 저장
* **데이터 형식**: 일반 텍스트로 저장되며 데이터를 인코딩하거나 암호화하지 않는다
* **사용 예**: 데이터베이스 URL, API 엔드포인트, 환경 변수, 애플리케이션 설정 파일 등.
* **보안**: 비밀 데이터가 아니라는 점에서 보안 처리가 필요하지 않는다. 데이터는 평문으로 저장됨

**예제**

```yaml
yamlCopy codeapiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database.url: "jdbc:mysql://localhost:3306/mydb"
  log.level: "INFO"
```

## Secret

`Secret`은 민감한 정보를 저장하고 관리하는 데 사용된다. 비밀번호, API 키, 인증서 등의 중요한 데이터를 안전하게 처리할 수 있다.

**주요 특징**

* **목적**: 민감한 정보를 저장하고 보호한다
* **데이터 형식**: Base64로 인코딩된 텍스트로 저장됩니다. 이는 데이터를 암호화하는 것이 아니라 인코딩하는 것으로, 기본적인 보안을 제공하지만 여전히 보안에는 취약하다
* **사용 예**: 데이터베이스 비밀번호, API 키, 인증서, TLS/SSL 키 등.
* **보안**: 데이터는 Base64로 인코딩되지만, 실제 암호화는 하지 않는다. Secret은 Kubernetes의 API 서버에 의해 기본적으로 보호됨

**예제**

```yaml
yamlCopy codeapiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=          # "admin"의 Base64 인코딩 값
  password: cGFzc3dvcmQ=      # "password"의 Base64 인코딩 값
```

### Secret의 type

* **Opaque**
  * **설명**: 가장 기본적인 타입으로, 데이터를 Base64로 인코딩하여 저장한다. 일반적으로 텍스트 기반의 비밀 데이터를 저장하는 데 사용된다.
  *   **예제**:

      ```yaml
      yamlCopy codeapiVersion: v1
      kind: Secret
      metadata:
        name: my-secret
      type: Opaque
      data:
        username: YWRtaW4=   # "admin"의 Base64 인코딩 값
        password: cGFzc3dvcmQ=  # "password"의 Base64 인코딩 값
      ```
* **docker-registry**
  * **설명**: Docker 이미지 레지스트리에 대한 인증 정보를 저장하는 데 사용. Docker 레지스트리에 접근하기 위한 자격 증명 정보
  * **필드**:
    * `username`: 레지스트리 사용자 이름
    * `password`: 레지스트리 비밀번호
    * `email`: (선택적) 이메일 주소
    * `server`: 레지스트리 서버 주소
  *   **예제**:

      ```yaml
      yamlCopy codeapiVersion: v1
      kind: Secret
      metadata:
        name: reg-secret
      type: kubernetes.io/dockerconfigjson
      data:
        .dockerconfigjson: <Base64 인코딩된 JSON 데이터>
      ```
* **service-account-token**
  * **설명**: 서비스 계정의 API 토큰을 저장, 이 토큰은 클러스터 내에서 서비스 계정으로 API를 호출할 때 사용됨
  * **필드**:
    * `token`: 서비스 계정의 API 토큰
    * `ca.crt`: (선택적) CA 인증서
  * **예제**: 이 `type`은 자동으로 생성되며, 수동으로 설정할 필요는 없다
* **ssh-auth**
  * **설명**: SSH 인증 정보를 저장, 이 `type`은 Kubernetes에서 SSH 키를 사용하여 인증하는 데 사용된다.
  * **필드**:
    * `ssh-privatekey`: SSH 개인 키
  *   **예제**:

      ```yaml
      yamlCopy codeapiVersion: v1
      kind: Secret
      metadata:
        name: ssh-secret
      type: kubernetes.io/ssh-auth
      data:
        ssh-privatekey: <Base64 인코딩된 SSH 개인 키>
      ```
* **basic-auth**
  * **설명**: HTTP Basic Authentication 자격 증명을 저장한다. 사용자 이름과 비밀번호를 사용하여 HTTP 기본 인증을 처리할 때 사용된다
  * **필드**:
    * `username`: 사용자 이름
    * `password`: 비밀번호
  *   **예제**:

      ```yaml
      yamlCopy codeapiVersion: v1
      kind: Secret
      metadata:
        name: basic-auth-secret
      type: kubernetes.io/basic-auth
      data:
        username: dXNlcg==   # "user"의 Base64 인코딩 값
        password: cGFzc3dvcmQ=  # "password"의 Base64 인코딩 값
      ```
* **htpasswd**
  * **설명**: HTTP 기본 인증을 위한 암호화된 비밀번호를 저장. `htpasswd` 파일 형식의 비밀번호 해시를 사용하여 HTTP 인증을 처리한다.
  * **필드**:
    * `auth`: 암호화된 비밀번호 해시
  *   **예제**:

      ```yaml
      yamlCopy codeapiVersion: v1
      kind: Secret
      metadata:
        name: htpasswd-secret
      type: kubernetes.io/htpasswd
      data:
        auth: <Base64 인코딩된 htpasswd 파일 내용>
      ```
