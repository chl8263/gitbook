# Azure 가상 사설망

Azure 가상 사설망(Virtual Private Network, VPN)은 Azure 클라우드 환경과 온-프레미스 네트워크 또는 다른 클라우드 네트워크 간에 안전한 연결을 제공한다. 이를 통해 분산된 네트워크 환경에서도 보안이 보장된 상태로 데이터를 주고받을 수 있다. Azure VPN에는 여러 가지 형태가 있으며, 각각의 사용 사례에 따라 적합한 옵션을 선택할 수 있다.

#### Azure VPN의 주요 구성 요소 및 유형

1. **Azure VPN Gateway**:
   * Azure VPN Gateway는 Azure 가상 네트워크(VNet)를 다른 VNet이나 온-프레미스 네트워크에 연결하기 위한 네트워크 게이트웨이 서비스이다.
   * 두 가지 주요 VPN 연결 모드를 제공한다:
     * **Site-to-Site (S2S) VPN**: 온-프레미스 네트워크와 Azure 가상 네트워크를 IPsec/IKE 프로토콜을 사용하여 연결한다. 주로 기업 네트워크와 Azure 간의 상호 연결을 위해 사용된다.
     * **Point-to-Site (P2S) VPN**: 개별 클라이언트(예: 랩톱, 모바일 디바이스)와 Azure 가상 네트워크를 SSL 기반의 VPN 프로토콜을 사용하여 연결한다. 주로 원격 작업자들이 Azure 리소스에 안전하게 접근할 수 있도록 하기 위해 사용된다.
2. **VPN Gateway SKU**:
   * Azure VPN Gateway에는 성능과 기능에 따라 다양한 SKU(Standard, HighPerformance 등)가 있다. 필요한 대역폭, 연결 수, 암호화 방식 등에 따라 적절한 SKU를 선택할 수 있다.
3. **Virtual Network Gateway**:
   * Virtual Network Gateway는 VPN Gateway를 생성하고 구성하는 데 사용된다. 이 가상 장치는 여러 네트워크 연결을 지원하며, 이를 통해 온-프레미스 네트워크와 Azure VNet 간의 트래픽을 라우팅한다.
4. **Connection**:
   * Connection은 두 네트워크 간의 실제 VPN 터널을 나타낸다. Site-to-Site 연결의 경우, 온-프레미스 VPN 장치와 Azure VPN Gateway 간의 터널을 설정한다. Point-to-Site 연결의 경우, 클라이언트 디바이스와 Azure VPN Gateway 간의 터널을 설정한다.

#### Azure VPN 구성의 주요 단계

1. **가상 네트워크(VNet) 생성**:
   * Azure 포털, Azure CLI, PowerShell 등을 사용하여 Azure 가상 네트워크를 생성한다.
   * 서브넷을 구성하여 네트워크 세그먼트를 정의한다.
2. **VPN Gateway 생성**:
   * Azure 가상 네트워크에 VPN Gateway를 생성한다.
   * 필요한 VPN Gateway SKU를 선택하고, 게이트웨이 서브넷을 지정한다.
3. **온-프레미스 VPN 장치 구성**:
   * 온-프레미스 네트워크와 Azure 가상 네트워크 간의 Site-to-Site 연결을 설정하려면, 온-프레미스 VPN 장치를 IPsec/IKE 프로토콜을 사용하여 구성해야 한다.
   * Azure에서 제공하는 구성 템플릿을 사용할 수 있다.
4. **VPN 연결 설정**:
   * Azure 포털 또는 CLI를 사용하여 VPN Gateway와 온-프레미스 VPN 장치 간의 연결을 설정한다.
   * Point-to-Site 연결의 경우, 클라이언트 디바이스에 VPN 클라이언트를 설치하고, 적절한 인증서를 구성한다.

#### Azure VPN의 장점

* **보안**: IPsec/IKE 및 SSL 프로토콜을 사용하여 데이터 암호화를 통해 안전한 통신을 보장한다.
* **확장성**: 다양한 SKU를 통해 네트워크 요구 사항에 맞는 적절한 성능과 기능을 선택할 수 있다.
* **유연성**: Site-to-Site, Point-to-Site, ExpressRoute(프라이빗 연결 옵션) 등 다양한 연결 옵션을 제공하여 다양한 네트워크 환경을 지원한다.
* **관리 용이성**: Azure 포털, CLI, PowerShell을 통해 쉽게 구성 및 관리할 수 있다.

Azure VPN을 통해 클라우드와 온-프레미스 간의 네트워크 연결을 안전하고 효율적으로 관리할 수 있으며, 이를 통해 하이브리드 클라우드 환경을 구축할 수 있다.
