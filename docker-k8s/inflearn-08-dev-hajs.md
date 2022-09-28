# 8강 쿠버네티스 스토리지
> https://kubernetes.io/ko/docs/concepts/storage/volumes/

<br><br>


## 쿠버네티스 클러스터 구조
(개념 다시 짚고 넘어가기)<br>
<img src="https://user-images.githubusercontent.com/57446639/192849244-093c8b7d-bd19-4da1-91bd-d9bfd8278fd0.png" width="650"/>
<br><br>


쿠버네티스에서는 저장소를 '스토리지' 라고 하지 않고 '**볼륨(Volume)**' 이라는 개념으로 말한다.<br>
볼륨은 컨테이너가 외부 스토리지에 엑세스하고 공유하는 방법이다. 파드의 각 컨테이너에는 고유의 분리된 파일 시스템이 존재하고, 볼륨은 파드의 스펙에 의해 정의되는 파드의 컴포넌트이다.
<br><br>


## 볼륨의 종류
1. 임시 볼륨 (emptyDir)
    - 컨테이너 <> 컨테이너 간의 데이터 공유시 사용 
2. 로컬 볼륨 (hostPath, local)
    - 컨테이너 <> 노드 간의 데이터 공유시 사용 
    - 로컬을 노드라고 생각하면 편하다.
3. 네트워크 볼륨 (iSCSI, NFS, cephFS...)
    - 파드 <> 파드 간의 데이터 공유시 사용 
    - 클러스터 외부에 있는 자원과 데이터를 공유하기 위해 쓰인다.
4. 네트워크 볼륨 (클라우드 종속적) (gcePersistentDist, awsEBS, azureFile...)
    - 외부 클라우드 서비스에서 제공해주는 네트워크 볼륨
<br><br>


### 임시 볼륨 (emptyDir)
> https://kubernetes.io/docs/concepts/storage/volumes/#emptydir

- 파드가 노드에 할당될 때 처음 생성되며, 해당 노드에서 파드가 실행되는 동안에만 존재한다.
- 컨테이너가 파괴되면(노드에서 파드가 제거되면) 임시 볼륨도 제거된다.
<br>

### 로컬 볼륨 (hostPath, local)
> https://kubernetes.io/ko/docs/concepts/storage/volumes/#hostpath

- 노드의 파일 시스템에 있는 특정 파일 또는 디렉터리를 지정한다.
- 노드가 달라지면 파일 공유가 되지 않기 때문에 다른 노드의 포드끼리는 데이터 공유를 할 수 없다.
- 보통 특정 노드를 관리할 목적으로 쓰인다. (모니터링)
<br>

### 네트워크 볼륨
> https://kubernetes.io/ko/docs/concepts/storage/volumes/#gcepersistentdisk

- 파드를 제거해도 네트워크 볼륨은 내용이 지워지지 않고, 마운트가 해제될 뿐이다.
<br><br><br>


## PV, PVC
개발자(사용자)가 인프라의 세부 사항(스토리지 기술의 종류)을 알지 못해도 사용할 수 있도록 제공해주는 리소스. 사용자 입장에서는 Storage 가 추상화 되어 있다고 인지 가능
- **PV**: Persistent Volume (관리자 영역)
- **PVC**: Persistent Volume Claim (사용자 영역)

PV 는 유지가능한 Disk 정도로 이해하면 된다.

<img src="https://user-images.githubusercontent.com/57446639/192859388-7c3c99cb-1796-4ceb-9470-41c32aa8df75.png" width="650"/>
<br><br>

인프라 관리자가 GCE Storage 를 설계하고 PVC 를 만들면, 개발자(사용자)는 사용하는 Storage 의 상세 스펙이 뭔지 몰라도
1. "PV 내용과 일치한다" 는 내용의 PVC 를 생성해서
2. 파드에 설정한 볼륨에 "PVC를 참조한다" 정도만 작성하면 클러스터의 스토리지를 사용할 수 있게 된다.

<img src="https://user-images.githubusercontent.com/57446639/192862997-976cbf6b-7550-40e0-bd54-b7aeeebb4551.png" width="650"/><br>
<br><br><br>


## PV 동적 프로비저닝
> https://kubernetes.io/ko/docs/concepts/storage/dynamic-provisioning/

- PV 를 직접 만드는 대신 사용자가 원하는 PV 유형을 선택하도록 오브젝트를 정의할 수 있는 기능.
- 가상 디스크를 만들어야 하기 때문에 gcp 와 같은 클라우드 환경에서만 가능하다는 제약 조건이 있다.
<br><br>

<img width="650" alt="image" src="https://user-images.githubusercontent.com/57446639/192863817-58cae4a0-a658-49c4-80a7-08ba04b7c3ec.png">
- parameters.type: 제공자에게 전달될 매개변수 (`pd-ssd` 는 GC 쪽에서 사용되는 용어, 클라우드마다 다름)
- provisioner: 프로비저닝에 사용할 플러그인 선택
- storageClassName: 스토리지 클래스 이름을 넣어주면 자동으로 PV 를 만들어서 붙여주는 역할을 함
<br><br><br>


## Statefulset
> https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

스테이트풀셋은 애플리케이션의 상태를 저장하고 관리하는 데 사용되는 쿠버네티스 객체이다.<br>
파드를 삭제하고 생성하면 새로운 가상환경이 시작되기 때문에 파드의 상태를 유지할 수 있는 방법이 없는데(*단순히 PV 를 하나 마운트해서 유지하는건 어렵기 때문에 논외*), 이때 활용할 수 있다.<br>
<br>

<img width="650" alt="image" src="https://user-images.githubusercontent.com/57446639/192867847-f1d51203-f619-4eb8-89dc-14717768b171.png"><br>
- 한 pod 에서 PVC 가 설정이 되고, PV 가 만들어지고, Storage 가 만들어지는 구조
- 가상환경(pod) 마다 Storage 가 있으므로 동일한 머신으로 동작하게 할 수 있다.
<br>

**특징**
- 스테이트풀셋은 각 파드의 기존 스펙(hostname, ip 등) 을 유지하여 생성한다.
- 이 전에 소개했던 deployment, replicaset 은 stateless 하므로 성격이 다르다.
- 파드 네트워크 ID 를 유지하기 위해서 **헤드레스(headless) 서비스** 가 필요하다.
<br>

**주의점**
- stateful 한 성격을 지녔기 때문에 스테이트풀셋과 관련된 볼륨은 삭제가 되지 않으므로 별도로 관리가 필요하다.
<br><br><br>


### Headless Service
서비스를 만들 때 `.spec.clusterIp: None` 이라고 지정해주면 된다.<br>
기존의 서비스와는 달리 kube-proxy 가 밸런싱이나 프록시 형태로 동작하지 않는다. (로드 밸런싱을 위한 목적이 아닌 도메인 주소를 공급하기 위함)
```yaml
apiVersion: v1
kind: Service
...
spec:
  clusterIp: None
  ...
```
<br><br>


## 그 외
> https://rook.io/

ceph 은 파일 스토리지를 가상화시키는 클러스터를 구성할 수 있는 소프트웨어이며,<br>
rook-ceph 는 rook project 에서 ceph 를 쉽게 사용할 수 있도록 만들어준 오픈소스이다.
