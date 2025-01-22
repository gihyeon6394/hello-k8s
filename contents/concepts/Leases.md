# Leases

- Node heartbeats
- Leader election
- API server identity
- Workloads

리스 : 분산시스템에서 필요한 개념으로 해당 lease를 획득한 프로세스는 해당 Lease와 관련된 자원의 독점을 보장하도록 하는 도구 (락과 비슷하나, 리스는 독점 시간제한이 있음)

분산 시스템은 종종 리스가 필요함 (공유 리소스에 락을 설정하고 협업하기 위함)  
k8s는 `coordination.k8s.io` API group 에 Lease object를 나타내어, node heartbeat, component 레벨의 리더 선출과 같은 시스템 크리티컬한 작업에 사용함

---

## Node heartbeats

1. 노드마다 lease object를 가지고 있음
2. 노드의 hearbeat : 해당 노드의 kubelet이 lease object의 `spec.renewTime` 필드를 주기적으로 갱신하는 요청
3. control plane은 해당 필드의 time stamp를 활용해 node의 상태를 추적함

k8s 는 Lease API를 사용해 kublet node heart beat를 k8s API server로 전송함. 모든 노드에는 Lease object가 있음 (
namespace: `kube-node-lease`)  
모든 kublet heartbeat는 Leas object에 대한 update 요청임. (Lease object의 `spec.renewTime` 필드에 대한 갱신 요청) control plane은 해당 필드의
time stamp를 활용해 node의 상태를 추적함

## Leader election

k8s는 lease를 사용해 컴포넌트의 인스턴스가 동시에 한개만 실행되도록 함. `kube-controller-manager`, `kube-scheduler` 등의 control plane controller는 오직
한개의 인스턴스만 실행되게하고 나머지는 stand-by 상태로 유지함

## API server identity

````
kubectl -n kube-system get lease -l apiserver.kubernetes.io/identity=kube-apiserver
NAME                                        HOLDER                                                                           AGE
apiserver-07a5ea9b9b072c4a5f3d1c3702        apiserver-07a5ea9b9b072c4a5f3d1c3702_0c8914f7-0f35-440e-8676-7844977d3a05        5m33s
apiserver-7be9e061c59d368b3ddaf1376e        apiserver-7be9e061c59d368b3ddaf1376e_84f2a85d-37c1-4b14-b6b9-603e62e4896f        4m23s
apiserver-1dfef752bcb36637d2763d1868        apiserver-1dfef752bcb36637d2763d1868_c5ffa286-8a9a-45d4-91e7-61118ed58d2e        4m43s

````

각 `kube-apiserer`는 Lease API를 사용해 다른 시스템에 자신의 ID 게시. 클라이언트는 control plane을 운영하는 `kube-apiserver` 인스턴스 수를 파악할 수 있음.  
각 kube-apiserver가 소유한 lease object를 `kube-system` namespace의 `apiserver-<sha256-hase>`로 확인할 수
있음. (`apiserver.kubernetes.io/identity` label을 사용하는 것도 가능)

## Workloads

workload가 소유할 Lease를 정의 가능. 예를들어 리더인 custom controller가 동작하도록 하고, 다른 동료(레플리카)들은 동작하지 않도록 할 수 있음  
리스 명명시에는 연결된 프로덕트나 컴포넌트와 연관되도록 하는것이 좋음. e.g. `Foo` component의 리스 `example-foo`    
컴포넌트의 인스턴스를 여러개 배포할 때는 prefix를 정하고 접미사로 해시값을 넣어주는 것도 좋음 (충돌 방지)

