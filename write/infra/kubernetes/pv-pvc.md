# PV와 PVC란

`PV`(PersistentVolume)와 `PVC`(PersistentVolumeClaim)는 스토리지를 관리하고 애플리케이션이 필요로 하는 디스크 공간을 프로비저닝하는 데 사용되는 중요한 개념이다



<figure><img src="../../../.gitbook/assets/Screenshot 2024-08-22 at 5.27.08 PM.png" alt=""><figcaption></figcaption></figure>

## PersistentVolume (PV)

* PersistentVolume (PV)는 클러스터 관리자가(주로 인프라 담당자) 생성한 스토리지의 실제 물리적 리소스를 나타낸다.
* PV는 클러스터에서 독립적으로 존재하며, 특정 사용자나 파드에 직접 연결되지는 않는다.
* 이 리소스는 외부 스토리지 시스템에 연결되거나 클러스터 내에서 동적으로 할당될 수 있다.

## PersistentVolumeClaim (PVC)

*
