# 세션 어피니티와 트래픽 폴리쉬

<figure><img src="../../../.gitbook/assets/Screenshot 2024-08-29 at 5.40.54 PM.png" alt=""><figcaption></figcaption></figure>

### 1. **세션 어피니티(Session Affinity)**

* **SessionAffinity**: Kubernetes Service 설정에서 클라이언트의 요청이 특정한 Pod에 지속적으로 전달되도록 설정할 수 있다.
  * **None**: 기본 설정으로, 클라이언트의 요청이 여러 Pod로 분산된다.
  * **ClientIP**: 동일한 클라이언트 IP에서 오는 요청이 동일한 Pod로 전달된다.
* **sessionAffinityConfig**: ClientIP를 사용한 세션 어피니티에서 세션 유지 시간을 설정. 예를 들어, `timeoutSeconds: 600`은 10분 동안 동일한 클라이언트가 동일한 Pod에 연결되도록 한다.

{% hint style="danger" %}
현재 이 기능은 일부 Pod에 과부하가 걸려 성능 문제가 발생할 수 있으며, 새로운 Pod로의 부하 분산이 제대로 이루어지지 않아 확장의 이점을 충분히 활용하지 못할 수 있고 로드밸런생 효율이 떨어져 권장되지 않는 방법
{% endhint %}

### 2. **트래픽 폴리시(Traffic Policy)**

* **internalTrafficPolicy**: 클러스터 내에서 트래픽이 어떻게 분산될지 결정한다
  * **Cluster**: 클러스터 내 모든 노드에서 트래픽을 처리
  * **Local**: 트래픽을 요청을 받은 노드 내에서만 처리한다. 다른 노드로 트래픽이 전달되지 않는다
* **externalTrafficPolicy**: 외부에서 들어오는 트래픽이 노드에 도달했을 때의 처리를 정의한다.
  * **Cluster**: 트래픽이 클러스터 전체로 분산
  * **Local**: 트래픽이 트래픽을 수신한 노드에만 전달되며, 다른 노드로 분산되지 않는다

### 3. **LoadBalancer**

* 위 그림에서 Ingress conroller LoadBalancer는 DaemonSet 으로 설치되어 모든 포트로 트래픽을 분산하고 있다.
* Ingress Controller를 DaemonSet으로 설치하면, 클러스터 내 모든 노드에 Ingress Controller가 실행되므로, 각 노드에서 직접 트래픽을 처리할 수 있다
* 이로 인해 트래픽이 불필요하게 다른 노드로 전송되는 것을 방지할 수 있다
  * 예를들어 ingress controller 가 특정 노드에만 있고 다른 노드로 트래픽이 들어오면 ingress controller 를 찾으려고 node 끼리 불필요한 통신을 하게 된다.
* Ingress Controller가 DaemonSet으로 설치된 경우, 외부 트래픽이 특정 노드로 들어왔을 때 그 노드에서 직접 트래픽을 처리할 수 있다.&#x20;
  * 이는 **internalTrafficPolicy: Local** 설정과 결합될 때 더욱 효과적이다.&#x20;
  * 즉, 로컬 노드에서 트래픽을 처리함으로써 네트워크 대역폭의 사용을 줄이고, 성능을 향상시킨다

### 4. **세션 관리**

* **Redis를 사용하여 pod에서 세션을 공통으로 사용한다**&#x20;
* 세션 데이터를 메모리에 저장하여 성능을 향상시키고, 확장성을 지원한다

### 5. **DaemonSet**

* **DaemonSet**: 클러스터의 모든 노드에 특정 Pod를 배포하기 위해 사용된다. Nginx와 같은 경우 모든 노드에 설치하여 트래픽 분산을 원활하게 하기 위함이다
