# 8. 인덱스

* CPU, 메모리 같은 전기적 장치의 성능은 발전하지만 디스크 같은 기계식 장치의 성능은 제한적으로 발전함
* 많은 양의 데이터는 결국 디스크에 저장되기 때문에 DB 튜닝은 디스크 I/O 를 얼마나 줄이느냐가 관식

## 8.1 디스크 읽기 방식 <a href="#8.1" id="8.1"></a>

### 8.1.1 하드 디스크 드라이브(HDD)와 솔리드 스테이트 드라이브(SSD) <a href="#8.1.1-hdd-ssd" id="8.1.1-hdd-ssd"></a>

* SSD 는 HDD 에서 데이터 저장용 플래터(원판)를 제거하고 그 대신 플래시 메모리를 장착
* 때문에 디스크 원판을 회전시킬 필요가 없기 때문에 데이터를 빨리 읽고 쓸 수있음
* 플래시 메모리는 전원이 공급되지 않아도 데이터가 삭제되지 않는다
* 순차 I/O 는 HDD 와 SDD 가 큰 차이가 없지만 랜덤 I/O 에서는 큰 차이가 있음
* 랜덤 I/O 를 통해 데이터를 읽고 쓰는 작업이 대부분이므로 SSD 는 DBMS 용 스토리지에 최적

### 8.1.2 랜덤 I/O 와 순차 I/O <a href="#8.1.2-i-o-i-o" id="8.1.2-i-o-i-o"></a>

* 디스크의 성능은 디스크 헤더의 위치 이동 없이 얼마나 많은 데이터를 한 번에 기록하느냐에 따라 결정됨
* 여러번 읽고 쓰는 작업인 랜덤 I/O 작업이 작업 부하가 월씬 더 큼
* 때문에 MySQL 서버는 그룹 커밋이나 바이너리 로그 버퍼 또는 InnoDB 로그 버퍼 등의 기능이 내장
* 일잔적으로 쿼리를 튜닝하는 것은 랜덤 I/O 자체를 줄여주는 것이 목적 -> 쿼리를 처리하는 데 꼭 필요한 데이터만 읽도록 쿼리를 개선하는 것을 의미
* 인덱스 레인지 스캔은 데이터를 읽기 위해 랜덤 I/O 를 사용하고 풀 테이블 스캔은 순차 I/O 를 사용함
* 그래서 큰 테이블의 레코드 대부분을 읽는 작업에서는 인덱스를 사용하지 않고 풀 테이블 스캔을 사용하도록 유도(데이터 웨어하우스나 통계 작업에서 사용)

## 8.2 인덱스란? <a href="#8.2" id="8.2"></a>

* 인덱스는 데이터 파일에 저장된 레코드의 주소에 비유됨
* DB 테이블의 모든 데이터를 검색해서 원하는 결과를 빠르게 가져오기 위해 칼럼의 값과 저장된 주소를 (key-value) 형태로 삼아 인덱스를 만들어 두는 것
* Index 는 항상 `정렬된` 상태를 유지
* Index 는 항상 정렬된 상태를 유지하기 때문에 데이터가 저장될 때마다 항상 값을 정렬해야 하므로 저장하는 과정이 복잡하고 느리지만, 이미 정렬되어 있기 때문에 원하는 값을 읽을 때는 아주 빨리 읽을 수있다.
* Index 가 많은 테이블은 INSERT, UPDATE, DELETE 문장의 처리는 느려지는 반면, SELETE 문장은 매우 빨리 처리됨
* Index 의 생성은 데이터 저장의 속도를 어디까지 희생하면서 읽기 속도를 얼마나 더 빠르게 만들어야 하느냐에 따라 결

## 8.3 B-Tree 인덱스

* B(Balanced)-tree 인덱스는 컬럼의 원래 값을변형 시키지 않고 인덱스 구조체 내에서는 항상 정렬된 상태로 유지한다.
* 가장 범용적인 인덱스 알고리즘

### 8.3.1 구조 및 특성

* B-Tree 인덱스는 트리구조의 최상위에 하나의 Root node 가 존재하고 그 하위에 자식 노드가 붙어 있는 형태다.&#x20;
* 트리 구조의 가장 하위에 있는 노드를 Leaf node 라 한다.
* 트리구조에서 루트 노드도 아니고 리프 노드도 아닌 중간의 노드를 Branch node 라 한다.
* 인덱스 Leaf node 는 항상 실제 데이터 레코드를 찾아가기 위한 주솟값을 가지고 있다.

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

* 위 그림처럼 인덱스의 키 값은 모두 정렬되어 있지만, 레코드는 정렬되어 있지 않고 임의의 순서로 저장되 있다.
* Insert 순서대로 저장되는것 처럼 생각하지만 Delete 된 레코드의 빈 공간을 재사용 하기 때문에 항상 Insert 순서대로 레코드가 저장되는 설계는 아니다.

#### MyISAM engine 의 index 설계

<figure><img src="../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

* MyISAM 테이블은 세컨더리 인덱스가 물리적인 주소를 바로 가진다.



#### InnoDB engine 의 index 설계

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

* &#x20;InnoDB 테이블은 프라이머리 키를 주소처럼 사용하기 때문에 논리적인 주소를 가진다고 볼 수있다.
* InnoDB 테이블에서 인덱스를 통해 레코드를 읽을 때는 인덱스에 저장돼 있는 프라이머리 키 값을 이용해 프라이머리 키 인덱스를 한 번 더 검색한 후, 프라이머리 키 인덱스의 리프 페이지에 저장돼 있는 레코드를 읽는다.
* InnoDB 스토리지 엔진에서는 모든 세컨더리 인덱스 검색에서 데이터 레코드를 읽기 위해서는 반드시 프라이머리 키를 저장하고 있는 B-Tree를 다시 한번 검색해야 한다.
* 성능이 떨어질것 같지만 두 엔진의 다른 설계는 장단점이 있다.

### 8.3.2 B-Tree 인덱스 키 추가 및 삭제

#### 8.3.2.1 인덱스 키 추가


