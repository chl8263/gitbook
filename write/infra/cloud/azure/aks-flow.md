---
description: >-
  Azure Kubernetes Service (AKS) 클러스터의 네트워크 아키텍처의 네트워크 흐름과 보안 장치를 통해 요청과 응답이 어떻게
  전달되는지를 설명한다
---

# AKS 네트워크 보안 flow

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

## 네트워크 구성 요소

* **Public Internet (공공 인터넷):** 외부 사용자가 클러스터에 접근하는 출발점
* **Web Application Firewall (WAF):** AKS 클러스터에 도달하기 전에 트래픽을 필터링하여 웹 애플리케이션을 보호하는 방화벽
* **Internal Load Balancer (AKS-managed):** 내부 트래픽을 분산시켜 AKS 서비스 간의 로드 밸런싱을 관리
* **Azure Kubernetes Service (AKS):** 애플리케이션을 실행하는 쿠버네티스 클러스터
* **Azure Firewall:** 네트워크의 보안을 강화하기 위한 Azure의 관리형 방화벽
* **VNet Peering:** 두 개의 가상 네트워크(VNet)를 연결하여 서로 간의 트래픽을 허용하는 메커니즘
* **Hub:** 중앙 보안 및 네트워크 관리가 이루어지는 장소로, Azure Firewall이 배치되어 있다
* **Spoke:** AKS 클러스터와 관련된 리소스가 배치

## 네트워크 흐름

1. **사용자 요청 (Request – User Initiated):**
   * 사용자는 공공 인터넷을 통해 AKS 클러스터에 요청을 보낸다
   * 이 요청은 먼저 Web Application Firewall (WAF)을 통해 들어간다
   * WAF는 트래픽을 검사한 후, 안전하다고 판단되면 트래픽을 내부 로드 밸런서로 전달한다
   * 내부 로드 밸런서는 요청을 적절한 AKS 노드로 분산시킨다
2. **사용자 응답 (Response – User Initiated):**
   * AKS 클러스터에서 응답이 생성되면, 내부 로드 밸런서를 통해 WAF로 다시 전달
   * WAF는 이 응답을 공공 인터넷으로 전달하여 사용자에게 되돌려 보낸다
3. **클러스터 시작 요청 (Request – Cluster Initiated):**
   * AKS 클러스터 내부에서 발생하는 요청(예: 외부 API 호출)은 Azure Firewall을 통해 외부로 나간다
     * 이 경우 Application gateway 로 나가는게 아니다
   * 이러한 요청은 VNet Peering을 통해 Hub 네트워크로 이동한 후 Azure Firewall을 거쳐 외부로 전달
4. **클러스터 응답 (Response – Cluster Initiated):**
   * 외부에서의 응답은 다시 Azure Firewall을 통해 받아들여지며, VNet Peering을 통해 Spoke의 AKS 클러스터로 전달된다

## 네트워크 보안

* **WAF (Web Application Firewall):** 외부에서 들어오는 HTTP(S) 트래픽을 검사하여 악의적인 요청을 차단힌다
* **Azure Firewall:** AKS 클러스터와 외부 간의 모든 트래픽을 필터링하고, 인바운드 및 아웃바운드 규칙을 적용하여 보안을 강화.
