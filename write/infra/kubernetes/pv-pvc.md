# PV와 PVC란

`PV`(PersistentVolume)와 `PVC`(PersistentVolumeClaim)는 스토리지를 관리하고 애플리케이션이 필요로 하는 디스크 공간을 프로비저닝하는 데 사용되는 중요한 개념이다



<figure><img src="../../../.gitbook/assets/Screenshot 2024-08-22 at 5.27.08 PM.png" alt=""><figcaption></figcaption></figure>

## PersistentVolume (PV)

* PersistentVolume (PV)는 클러스터 관리자가(주로 인프라 담당자) 생성한 스토리지의 실제 물리적 리소스를 나타낸다.
* PV는 클러스터에서 독립적으로 존재하며, 특정 사용자나 파드에 직접 연결되지는 않는다.
* 이 리소스는 외부 스토리지 시스템에 연결되거나 클러스터 내에서 동적으로 할당될 수 있다.

## PersistentVolumeClaim (PVC)

* PersistentVolumeClaim (PVC)는 사용자(파드)가 필요한 스토리지의 특정 요구사항을 정의하는 요청이다
* PVC는 사용자가 스토리지를 요청하는 방법을 나타내며, PV와 연결되어 실제 스토리지를 소비하게 됨&#x20;
* PVC는 사용자가 필요로 하는 스토리지의 크기와 접근 모드를 지정할 수 있다
* 주로 개발자가 작성하며 PVC가 제출되면, Kubernetes는 PVC의 요구사항과 일치하는 PV를 찾아서 바인딩한다

## PV와 PVC의 관계

* **바인딩 (Binding)**: PVC가 생성되면 Kubernetes는 해당 PVC의 요구사항(스토리지 크기, 접근 모드 등)에 맞는 PV를 검색하고 이를 바인딩한다. 한 번 바인딩된 PV는 해당 PVC와 연결되어 특정 파드가 스토리지에 접근할 수 있게 된다.
* **접근 모드 (Access Modes)**:
  * `ReadWriteOnce (RWO)`: 단일 노드에서 읽기/쓰기가 가능.
  * `ReadOnlyMany (ROX)`: 여러 노드에서 읽기만 가능.
  * `ReadWriteMany (RWX)`: 여러 노드에서 읽기/쓰기가 가능.
* **스토리지 정책 (Reclaim Policy)**: PV의 스토리지 반환 정책을 정의
  * `Retain`: PV의 데이터가 유지됨.
  * `Recycle`: PV의 데이터가 기본적인 삭제 후 재사용 가능.
  * `Delete`: PV와 그에 연결된 스토리지가 삭제됨.



{% hint style="info" %}
**Pod 에서 직접 적으로 Volumes에 연결하면 되는것 아닌가?!**

* k8s에서의 pod는 개발자가 관리하는 것 이므로 volumn같이 인프라 관련 요소들에 대한 관리 포인트를 분리하기 위함과 많은 pv 솔루션에 대해 유연하게 대처할 수 있다
* 인프라 담당자가 이것을 관리하는 컨셉으로 PV라는 오브젝트를 만드는 것이다
* 그리고 PVC는 pod 에서 필요한 자원을 요청하는 용도로 개발자가 만들게 된다
* 이렇게 인터페이스를 해주는 것이 존재함으로써 PV의 솔루션이 변경되더라도 pod까지 손대는 일이 없어지게 된다
{% endhint %}

<figure><img src="../../../.gitbook/assets/Screenshot 2024-08-23 at 9.26.21 AM.png" alt=""><figcaption></figcaption></figure>
