# 8강 쿠버네티스 스토리지
| https://kubernetes.io/ko/docs/concepts/storage/volumes/
<br><br>


### 쿠버네티스 클러스터 구조
(개념 다시 짚고 넘어가기)<br>
<img src="https://user-images.githubusercontent.com/57446639/192849244-093c8b7d-bd19-4da1-91bd-d9bfd8278fd0.png" width="650"/>
<br><br>


쿠버네티스에서는 저장소를 '스토리지' 라고 하지 않고 '**볼륨(Volume)**' 이라는 개념으로 말한다.<br>
볼륨은 컨테이너가 외부 스토리지에 엑세스하고 공유하는 방법이다. 파드의 각 컨테이너에는 고유의 분리된 파일 시스템이 존재하고, 볼륨은 파드의 스펙에 의해 정의되는 파드의 컴포넌트이다.
<br><br>


### 볼륨의 종류
1. 임시 볼륨 (emptyDir)
    - 컨테이너 <> 컨테이너 간의 데이터 공유시 사용 
2. 로컬 볼륨 (hostPath, local)
    - 컨테이너 <> 노드 간의 데이터 공유시 사용 
    - 로컬 = 노드
3. 네트워크 볼륨 (iSCSI, NFS, cephFS...)
    - 파드 <> 파드 간의 데이터 공유시 사용 
    - 클러스터 외부에 있는 자원과 데이터를 공유하기 위해 쓰인다.
4. 네트워크 볼륨 (클라우드 종속적) (gcePersistentDist, awsEBS, azureFile...)
    - 외부 클라우드 서비스에서 제공해주는 네트워크 볼륨
<br><br>


#### 임시 볼륨 (emptyDir)
| https://kubernetes.io/docs/concepts/storage/volumes/#emptydir
파드가 노드에 할당될 때 처음 생성되며, 해당 노드에서 파드가 실행되는 동안에만 존재한다.<br>
= 컨테이너가 파괴되면(노드에서 파드가 제거되면) 임시 볼륨도 제거된다.
<br>

#### 로컬 볼륨 (hostPath, local)
| https://kubernetes.io/ko/docs/concepts/storage/volumes/#hostpath
노드의 파일 시스템에 있는 특정 파일 또는 디렉터리를 지정한다.<br>
= 노드가 달라지면 파일 공유가 되지 않기 때문에 다른 노드의 포드끼리는 데이터 공유를 할 수 없다.
= 보통 특정 노드를 관리할 목적으로 쓰인다. (모니터링)
<br>

#### 네트워크 볼륨
| https://kubernetes.io/ko/docs/concepts/storage/volumes/#gcepersistentdisk
파드를 제거해도 네트워크 볼륨은 내용이 지워지지 않고, 마운트가 해제될 뿐이다.
<br><br>


### PV, PVC
개발자(사용자)가 인프라의 세부 사항(스토리지 기술의 종류)을 알지 못해도 사용할 수 있도록 제공해주는 리소스. 사용자 입장에서는 Storage 가 추상화 되어 있다고 인지
- PV: Persistent Volume (관리자 영역)
- PVC: Persistent Volume Claim (사용자 영역)

PV 는 유지가능한 Disk 정도로 이해하면 된다.

<img src="https://user-images.githubusercontent.com/57446639/192859388-7c3c99cb-1796-4ceb-9470-41c32aa8df75.png" width="650"/>

인프라 관리자가 GCE Storage 를 설계하고 PVC 를 만들면, 개발자(사용자)는 사용하는 Storage 의 상세 스펙이 뭔지 몰라도
1. "PV 내용과 일치한다" 는 내용의 PVC 를 생성해서
2. 파드에 설정한 볼륨에 "PVC를 참조한다" 정도만 작성하면 클러스터의 스토리지를 사용할 수 있게 된다.

<img src="https://user-images.githubusercontent.com/57446639/192862997-976cbf6b-7550-40e0-bd54-b7aeeebb4551.png" width="650"/>
<br><br>

#### PV 동적 프로비저닝
PV 를 직접 만드는 대신 사용자가 원하는 PV 유형을 선택하도록 오브젝트를 정의할 수 있는 기능. 단, 가상 디스크를 만들어야 하기 때문에 gcp 와 같은 클라우드 환경에서만 가능하다는 조건이 있다.
