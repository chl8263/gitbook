---
description: >-
  Azure Kubernetes Service (AKS) 클러스터와 Azure Application Gateway를 활용하여 애플리케이션을
  배포할 때의 네트워크 흐름을 설명한다
---

# AKS 네트워크 flow

&#x20;

<figure><img src="../../../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

## 네트워크 흐름 및 구성 요소

1. **사용자(User)와 Application Gateway 간의 통신**
   * 사용자는 인터넷을 통해 애플리케이션에 접근한다
   * 이 요청은 먼저 **Application Gateway**로 전달된다
   * Application Gateway는 웹 애플리케이션 방화벽(WAF) 기능을 포함할 수 있으며, HTTPS 요청을 처리한다
   * SSL/TLS 트래픽을 암호화하여 안전한 연결을 유지합니다. 이 과정에서 인증서가 사용되며, 이 인증서는 Azure Key Vault에서 관리된다
   * 사용자가 발급받은 인증서 파일(.pfx 또는 .pem 형식)을 Azure Key Vault 로 수동으로 업로드 할 수 있고
   *   **Key Vault를 통해 자동 발급:** Key Vault는 Azure에서 직접 인증서를 발급받을 수 있는 기능을 제공한다. Azure Key Vault는 Digicert, GlobalSign, Let's Encrypt 등의 인증 기관과 통합되어 있어서 이런 인증서 관련 작업을 단순화 시켜준다


2.  **Application Gateway와 AKS 클러스터 간의 통신:**

    * Application Gateway는 요청을 받아 내부 네트워크를 통해 AKS 클러스터로 전달한다
    * 이 과정에서도 트래픽은 암호화된 상태를 유지하며, 인그레스 컨트롤러가 클러스터 내부에서 트래픽을 관리한다


3.  **Ingress Controller와 Workload 간의 통신:**

    * **Ingress Controller**는 AKS 클러스터 내에서 실행되며, 외부 요청을 적절한 서비스나 파드로 라우팅하는 역할을 수행한다
    * 클러스터 내부의 워크로드(애플리케이션 컨테이너)로 트래픽을 전달, 이때도 보안이 유지된다


4. **Azure Key Vault와의 연계:**
   * 인증서 관리가 중요한 핵심인데, **Azure Key Vault**는 SSL/TLS 인증서를 안전하게 저장하고 관리하는 역할을 한다.
   * Application Gateway는 Key Vault에서 필요한 인증서를 가져와 HTTPS 연결을 설정하는 데 사용하고 다양한 편의성을 제공한다.
   * `위 이미지에서 delta.contoso.com` 및 `*.aks-ingress.contoso.com` 도메인에 대한 인증서가 Key Vault에 저장되어 있는 것을 볼 수 있다
