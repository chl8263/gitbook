---
description: >-
  클러스터 내의 서비스에 대한 외부 접근을 관리하는 API 오브젝트이며, 일반적으로 HTTP를 관리함. 인그레스는 부하 분산, SSL 종료,
  명칭 기반의 가상 호스팅을 제공할 수 있다
---

# Ingress란?

<figure><img src="../../../.gitbook/assets/Screenshot 2024-08-29 at 9.34.42 AM.png" alt=""><figcaption></figcaption></figure>

### 1. **Kubernetes Cluster Components (쿠버네티스 클러스터 구성 요소)**

* **Master Node**:
  * Kubernetes의 Control Plane을 운영하는 노드이다. `kube-apiserver`, `kube-scheduler`, `kube-controller-manager` 등의 핵심 서비스들이 실행되고 있다.
  * **Ingress Controller**는 Master Node에 배포되어 있고, 이 컨트롤러는 Ingress 리소스를 모니터링하고 필요한 설정을 자동으로 적용한다.
* **Worker Node**:
  * 애플리케이션이 실제로 배포되고 실행되는 노드이다. 이 도식도에서는 두 개의 Worker Node가 존재한다.
  * 각 노드에는 Pod가 배포되어 있으며, 이 Pod들은 네트워크를 통해 서로 통신한다.

### 2. **Namespace (네임스페이스)**

* **nginx-ingress** 네임스페이스:
  * 이 네임스페이스 내에 Ingress Controller와 관련된 리소스들이 존재한다.
  * `nginx-ingress-controller`라는 이름의 Service가 LoadBalancer 타입으로 구성되어 있어 외부 트래픽을 받을 수 있다.
* **api-tester-322** 네임스페이스:
  * 이 네임스페이스 내에는 다양한 Service와 Pod들이 배포되어 있다.
  * `portal`, `core`라는 Service가 있으며, 이들은 `ClusterIP` 타입으로 설정되어 클러스터 내부에서만 접근이 가능합니다.
  * service 가 nodeport 로 되어있지 않아 외부 트래픽을 서비스로 바로 받는 것이 아니라 위 구조 에서는 ingress controller 로 인해 트래픽을 수신받는다.

### 3. **Service와 Ingress**

* **Service**:
  * 도식도에서 `ClusterIP`로 설정된 `portal`, `core` 서비스는 클러스터 내부에서만 접근할 수 있다.
  * `ingress-nginx`라는 또다른 Service는 `LoadBalancer` 타입으로 외부 트래픽을 받을 수 있으며, `nginx2`라는 Pod와 연결된다.
    * 이 구성은 서비스별로 로드밸런서를 분리해서 부하를 줄이기 위한 예시이다.
* **Ingress**:
  * Ingress 리소스는 외부에서 특정 도메인으로 들어오는 트래픽을 특정 서비스로 라우팅하는 규칙을 정의합니다.
  * 도식도에는 세 가지 Ingress가 정의되어 있으며, 각 Ingress는 `host`, `path`, `service`와 같은 속성으로 구성되어 있다.
  * `class: nginx`로 설정된 Ingress는 `portal.com`과 `k8s.core` 도메인을 각각 `portal`과 `core` 서비스로 라우팅한다.
  * `class: nginx2`로 설정된 Ingress는 `portal2.com` 도메인을 `portal2` 서비스로 라우팅한다.
  * Ingress 는 당연히 ingres controller 없이 동작할 수 없다

{% hint style="info" %}
<mark style="color:red;">**Ingress 를 보고 path 에 따라 서비스로 리다이렉팅 시키는데, 이 때 각 서비스를 통해 pod 로 네트워크 통신을 하는것이 아니라, ingress controller 가 ingress 를 보고 해당 서비스에 알아서 부하를 계산하여 최적화된 pod 를 선택한다.**</mark>
{% endhint %}

### 4. **Load Balancer와 DNS**

* **LoadBalancer**:
  * 외부에서 들어오는 트래픽을 받아 클러스터 내의 서비스로 분산한다.
  * `233.12.42.56`, `233.12.42.57` IP를 가진 LoadBalancer가 있으며, 이는 각각 `portal.com`, `portal2.com` 도메인을 처리한다.
* **DNS Server**:
  * `portal.com` 도메인은 공인 DNS 서버에 등록되어 있으며, 외부에서 이 도메인으로 요청이 들어오면 LoadBalancer로 트래픽이 전달한다.
  * 내부 DNS 서버도 존재하며, 내부 서비스 간의 이름 해석을 담당

### 5. **ClusterRole과 ClusterRoleBinding**

* **ClusterRole**:
  * Ingress Controller가 Kubernetes API에 접근해 리소스를 관리할 수 있는 권한을 정의
* **ClusterRoleBinding**:
  * 정의된 ClusterRole을 특정 ServiceAccount에 바인딩하여 권한을 부여
