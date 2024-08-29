# Ingress TCL 적용및 관리

<figure><img src="../../../.gitbook/assets/Screenshot 2024-08-29 at 10.21.30 AM.png" alt=""><figcaption></figcaption></figure>

#### 1. **기본 네트워크 구성**

* **Master Node**:
  * Master Node에는 `coreDNS`와 `ingress-nginx-controller`가 배포되어 있다. `coreDNS`는 클러스터 내 DNS 역할을 담당하며, `ingress-nginx-controller`는 외부로부터의 요청을 수신하고 적절한 서비스로 라우팅하는 역할을 한다.
* **Worker Node**:
  * 두 개의 Worker Node가 있으며, 각 Node에는 여러 Pod가 배포되어 있다.
  * `nginx-ingress-controller`는 Worker Node에서 `LoadBalancer` 서비스로 노출되어 있으며, 80과 443 포트에서 트래픽을 처리한다.
  * 각 Worker Node의 31080, 31443 포트는 외부로 노출되어 있으며, 이 포트들을 통해 외부 요청이 들어오게 된다.

#### 2. **TLS 설정**

* **TLS 설정 적용 위치**
  * `Ingress` 리소스에서 TLS 설정이 적용될 수 있다. 위 그림에서 보여주듯이, `portal.com` 도메인에 대해 TLS가 적용되어 있으며, SSL 리다이렉션이 설정되어 있다.
  * Nginx SSL 리다이렉션 설정은 http 트래픽을 https로 자동 변환한다.
  * Ingress에 따라 각기 다른 TLS 설정을 적용할 수 있으며, 이를 통해 특정 서비스에만 TLS를 적용하거나, 필요한 경우 다른 인증서를 사용할 수 있다.
  * TLS 는 LoadBalancer, ingress controller, ingress 별로 각각 적용할 수 있지만, 중복이 될 경우 성능에 영향을 줄 수 있으므로 확인이 필요하다.
* **TLS 설정과 Secret**
  * TLS를 적용하려면 인증서와 개인 키가 필요하다. 도식도에서는 `Secret` 리소스를 통해 인증서(`tls.crt`)와 키(`tls.key`)를 Kubernetes 클러스터에 저장하고 이를 Ingress가 참조한다.

#### 3. **TLS 적용의 장단점 분석**

* **장점**:
  1. **보안 강화**:
     * TLS를 통해 통신 내용을 암호화함으로써 데이터 전송 시 보안을 강화할 수 있다.
     * 중간 기기(예: 라우터, 게이트웨이, 로드밸런서)에서 트래픽을 감청하는 것을 방지할 수 있다.
  2. **신뢰성 확보**:
     * 신뢰할 수 있는 CA(Certificate Authority)에서 발급한 인증서를 사용하면 사용자가 웹사이트의 신뢰성을 확신할 수 있다.
  3. **SEO 및 사용자 경험 개선**:
     * 검색 엔진은 HTTPS 사이트를 선호하며, 브라우저는 HTTPS 사이트에 대해 긍정적인 표시를 제공
* **단점**:
  1. **성능 저하**:
     * TLS를 적용하면 암호화 및 복호화 과정에서 CPU 리소스를 사용하기 때문에 성능이 다소 저하될 수 있다. 특히, 트래픽이 많은 경우 성능에 영향을 미칠 수 있습니다.
  2. **복잡성 증가**:
     * TLS 인증서를 관리하고 갱신하는 과정이 복잡할 수 있다. 잘못된 설정으로 인해 서비스가 중단될 위험도 있다.
  3. **추가적인 설정 필요**:
     * TLS 설정과 SSL 리다이렉션을 제대로 적용하려면 추가적인 설정이 필요. 예를 들어, 리다이렉션 포트를 변경하는 등의 설정이 필요하다.

#### 4. **Nginx 관련 설정**

* **ConfigMap**:
  * `ConfigMap`을 통해 `nginx-ingress-controller`의 설정을 관리할 수 있다. 도식도에서는 로드 밸런싱 알고리즘을 `round-robin`에서 `ewma`로 변경하여 트래픽 분산 방식을 실시간으로 조정할 수 있다.
* **로그 포맷 및 타임존 변경**:
  * Nginx의 로그 포맷과 타임존도 도식도에서 보여주듯이 쉽게 변경 가능

#### 5. **트래픽 흐름과 URL 경로 변경**

* **Rewrite Target**:
  * `Ingress`에서 `rewrite-target`을 사용해 특정 경로(`/core/`, `/cust/`)를 다른 경로로 재지정할 수 있다. 이 기능은 URL 구조를 단순화하거나 외부로 노출되는 URL을 재구성하는 데 유용하다.
* **포트 변경 및 리다이렉션**:
  * `nginx.ingress.kubernetes.io/ssl-redirect: "true"` 설정을 통해 HTTP 요청을 HTTPS로 리다이렉션할 수 있으며, 포트 번호도 변경 가능.
