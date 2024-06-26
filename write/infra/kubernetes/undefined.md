# 서비스계정

서비스 계정(Service Account)은 Kubernetes 클러스터 내에서 애플리케이션 또는 프로세스가 다른 리소스에 접근할 때 사용되는 인증 정보를 제공하는 Kubernetes 리소스입니다. 서비스 계정은 다음과 같은 주요 기능과 역할을 수행합니다:

1. **인증 및 권한 부여:**
   * 서비스 계정은 클러스터 내의 다른 리소스(파드, 서비스 등)에 접근할 때 사용됩니다.
   * 각 서비스 계정에는 Kubernetes API 서버에 대한 액세스 권한을 부여하는 인증 토큰이 포함되어 있습니다.
2. **리소스 접근 제어:**
   * 서비스 계정은 네임스페이스 내에서 작동하며, 특정 네임스페이스에 속한 리소스에 접근할 수 있습니다.
   * Kubernetes는 RBAC(Role-Based Access Control)을 통해 서비스 계정이 사용할 수 있는 리소스와 작업을 제한할 수 있습니다.
3. **애플리케이션과의 통합:**
   * 서비스 계정은 주로 클러스터 내에서 실행되는 애플리케이션, 컨트롤러, 데몬 등과 관련이 있습니다.
   * 예를 들어, 파드가 특정 서비스를 호출하거나 클러스터의 다른 파드와 통신할 때 사용될 수 있습니다.
4. **기본 서비스 계정:**
   * Kubernetes는 각 네임스페이스에 기본적으로 `default` 서비스 계정을 제공합니다. 이 계정은 명시적으로 생성하지 않아도 모든 네임스페이스에서 사용할 수 있습니다.
   * `default` 서비스 계정은 일반적으로 파드나 리소스가 별도의 서비스 계정을 명시적으로 지정하지 않았을 때 사용됩니다.

서비스 계정은 일반적으로 다음과 같은 방식으로 생성 및 관리됩니다:

*   **수동 생성:** 필요에 따라 `kubectl create serviceaccount` 명령어를 사용하여 서비스 계정을 생성할 수 있습니다.

    ```sh
    sh코드 복사kubectl create serviceaccount my-service-account
    ```
*   **파드와 연결:** 파드를 생성할 때 해당 파드가 사용할 서비스 계정을 지정할 수 있습니다. 이를 통해 파드는 클러스터의 다른 리소스에 접근할 때 해당 서비스 계정을 사용합니다.

    ```yaml
    yaml코드 복사apiVersion: v1
    kind: Pod
    metadata:
      name: my-pod
    spec:
      serviceAccountName: my-service-account
      containers:
      - name: my-container
        image: nginx
    ```
* **RBAC 설정:** 서비스 계정에 대한 권한을 관리하기 위해 Kubernetes의 RBAC을 사용하여 리소스 접근을 제어할 수 있습니다. 예를 들어, 특정 서비스 계정이 특정 네임스페이스 내의 리소스에 대한 읽기 또는 쓰기 권한을 부여할 수 있습니다.

서비스 계정은 Kubernetes 클러스터의 보안과 리소스 관리에 중요한 역할을 합니다. 필요에 따라 적절한 권한과 범위를 설정하여 클러스터 내에서 안전하고 효율적인 리소스 관리를 지원합니다.
