# 컴포넌트와 리소스

<figure><img src="../../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

#### 1. **Namespace**

* **Namespace**는 쿠버네티스 클러스터에서 리소스를 격리하는 논리적인 그룹.&#x20;
* 다이어그램에서는 두 개의 네임스페이스(`api-tester-1231`, `anotherclass-123`)가 나와 있으며, 각 네임스페이스 내에 속한 리소스들은 서로 독립적으로 운영된다.

#### 2. **Deployment**

* **Deployment**는 애플리케이션의 배포 및 업데이트를 관리하는 리소스
* 이 다이어그램에서는 `api-tester-1231` 네임스페이스에서 `Deployment`가 있고, 이는 레플리카셋(ReplicaSet)을 생성하고 관리
* 레플리카 수와 배포 전략을 정의하며, 이를 통해 애플리케이션의 안정적 운영 및 롤링 업데이트 등을 수행할 수 있다.

#### 3. **ReplicaSet**

* **ReplicaSet**은 Deployment에 의해 생성되며, 특정 수의 동일한 Pod 인스턴스가 항상 실행되도록 보장
* 다이어그램에서 `api-tester-1231-xxxx` 레플리카셋이 있으며, 이 레플리카셋은 선택자(selector)를 통해 어떤 Pod를 관리할지 정의

#### 4. **Pod**

* **Pod**는 쿠버네티스에서 가장 작은 배포 단위로, 컨테이너가 실행되는 단위이다
* 이 다이어그램에서는 `api-tester-1231-xxxx-yyyy`라는 Pod가 있으며, 컨테이너 설정, 이미지, 볼륨 마운트, 리소스 제한 등을 포함하고 있다.
* 이 Pod는 레플리카셋에 의해 관리되며, 해당 레플리카셋에 의해 정의된 수만큼의 Pod 인스턴스가 유지된다

#### 5. **Service**

* **Service**는 Pod에 대한 네트워크 접근을 제공하는 추상화된 리소스
* `api-tester-1231` 서비스는 선택자(selector)를 사용하여 특정 레이블을 가진 Pod와 연결
* 이를 통해 클러스터 내 또는 외부에서 Pod에 접근할 수 있으며, 필요에 따라 로드 밸런싱을 수행

#### 6. **HPA (Horizontal Pod Autoscaler)**

* **HPA**는 애플리케이션 부하에 따라 Pod의 수를 자동으로 조정하는 리소스
* `api-tester-1231-default` HPA는 `Deployment` 또는 `ReplicaSet`과 연결되어 있으며, 정의된 메트릭(예: CPU 사용률)에 따라 최소/최대 Pod 수를 조정

#### 7. **PVC (Persistent Volume Claim)**

* **PVC**는 Pod에서 사용할 영구 스토리지를 요청하는 리소스
* `api-tester-1231-files`라는 PVC는 필요한 스토리지의 크기와 접근 모드를 정의하고, 이를 통해 PV(Persistent Volume)와 연결

#### 8. **PV (Persistent Volume)**

* **PV**는 쿠버네티스 클러스터 내에서 영구 스토리지 리소스를 나타낸다
* `api-tester-1231-files`라는 PV는 실제 물리적 스토리지를 나타내며, PVC와 연결되어 Pod에 스토리지를 제공
* 이 PV는 용량, 접근 모드, 로컬 경로 등을 정의하며, `nodeAffinity`를 사용해 특정 노드에 스토리지를 할당할 수도 있다.

#### 9. **ConfigMap**

* **ConfigMap**은 환경 설정 데이터를 Key-Value 형태로 저장하는 리소스
* `api-tester-1231-properties`라는 ConfigMap은 애플리케이션이 사용할 설정 값을 저장하며, 이 데이터는 Pod에 주입되어 컨테이너 내에서 사용될 수 있다.

#### 10. **Secret**

* **Secret**은 민감한 데이터를 저장하는 리소스
* `api-tester-1231-postgresql`이라는 Secret은 데이터베이스의 자격 증명과 같은 민감한 정보를 저장하며, Pod에서 이를 사용할 수 있다
