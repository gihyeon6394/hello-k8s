# Nodes

- Management
- Node status
- Node heartbeats
- Node controller
- Resource capacity tracking
- Node topology
- Swap memory management

k8s는 사용자의 workload를 노드안의 파드 안의 컨테이너에 배치함으로서 구동함. 노드는 논리 혹은 실제 머신일 수 있음  
각 node는 control plane에 의해 관리되고, pod를 실행하기 위한 서비스들을 유지함

일반적으로 클러스터에는 여러개의 노드가 있으며 리소스가 제한적일 떄는 하나의 노드만 존재하기도 함

노드의 컴포넌트에는 kubelet, container runtime, kube-proxy가 있음

---

## Management

API server에 노드를 붙이는 방법은 크게 2가지 있음

- 방법 1. 노드의 kubelet이 스스로 control plane에 등록
- 방법 2. 사용자가 control plane에 노드를 수동으로 등록

node object를 생성하고 나면 (혹은 Kubelet 이 스스로 등록하면), control plane은 새로운 node object가 유효한지 체크

```yaml
{
  "kind": "Node",
  "apiVersion": "v1",
  "metadata": {
    "name": "10.240.79.157",
    "labels": {
      "name": "my-first-k8s-node"
    }
  }
}
```

1. 위 메니페스트로 node object를 생성
2. k8s가 해당 node object의 `metadata.name`에 해당하는 kublet이 API server에 등록되었는지 확인
3. 노드가 정상 실행중인 경우 해당 노드는 파드를 실행할 자격이 있음
    - 비정상인 경우 정상이 될때까지 클러스터에서 제외됨
    - k8s는 지속적으로 Invalid node를 체크함

### Node name uniqueness

name은 노드를 식별할 수 있어야함. 동시에 node name은 유일해야함. k8s는 resource name이 같으면 같은 object로 인식함  
즉, 인스턴스의 name이 같으면 같은 state로 인식함 (network setting, root disk content)
따라서 name이 바뀌지 않은채 인스턴스가 수정되면 k8s는 수정을 인식하지 못함   
노드의 재배치, 주요 업데이트 시 기존 노드를 API server로부터 제거한 뒤 수정 후 다시 등록할 것

### Self-registration of Nodes

kublet의 `--register-node` 플래그가 true (default)이면 kublet은 스스로를 API server에 등록함 (대부분의 배포판에서 선호 패턴)

#### self-registration을 위한 옵션

- `--kubeconfig` : API server 인증을 위한 credential 파일 경로
- `--cloud-provider` : cloud provider에게 metadata를 요청할 때 사용
- `--register-node` : API server에 자동으로 등록할지 여부
- `--register-with-taints` : 노드에 taint를 추가할지 여부 (comma-separated list of key=value)
    - 옵션이 없을시 `false`로 설정
- `--node-ip` : 노드에 대한 optional comma-separated list of IP addresses
    - 오직 하나의 ip만을 각 address family에 대해 사용할 수 있음
    - 설정 값이 없으면 Node의 기본 IPv4 주소를 사용함
        - node의 기본 IPv4가 없으면 node의 기본 IPv6 주소를 사용함
- `--node-labels` : cluster에 node를 등록할 때 추가할 라벨
- `--node-status-upate-frequency` : kublet이 node의 상태를 API server에 업데이트 요청하는 주기

##### 주의 : 노드를 업데이트 시 노드를 API server에 재등록하자

- 예를 들어 node가 변경된 `--ndoe-labels`로 새롭게 시작했지만 기존 node의 이름과 똑같다면, 변경 사항이 k8s에 반영되지 않음
    - 노드 라벨은 노드가 API server에 등록될 때만 설정되기 때문

### Manual Node administration

kubectl로 수동으로 node object 생성/수정 가능 flag `--register-node=false` 설정 시 노드를 수동으로 등록해야함  
`--register-node` 없이도 노드를 수정 가능

````
# node $NODENAME을 unschedulable로 설정
kubectl cordon $NODENAME
````

unschedulable 노드는 새로운 파드를 스케쥴링하지 않음 (기존 노드는 계속 실행됨)

## Node status

#### node status에 포함된 정보

- Addresses
- Conditions
- Capacity and Allocatable
- Info

````
# node 상태 확인
kubectl describe node <insert-node-name-here>
````

## Node heartbeats

k8s node는 주기적으로 cluster에 heartbeat을 전송해 clsuter가 node별 상태를 파악, 관리할 수 있게 함

#### heartbeat 형태

- node 의 `.status` 를 수정
- `kube-node-leas` namespace에 `Lease` object 생성, 각 노드별로 lease object를 생성

## Node controller

node의 여러 부분을 관리하는 control plane component

- 노드에 CIDR 할당
- cloud provider의 사용가능 머신에 대한 정보와 노드 리스트를 최신화해서 관리
- 노드 health check : 기본적으로 5초마다 체크
    - `kube-controller-manager`컴포넌트의  `--node-monitor-period` flag로 체크 주기 설정 가능

노드가 unhealthy하면 cloud provider에게 해당 노드의 vm 상태를 확인하고, 불가하다면 노드를 제거하고 리스트에서 삭제

1. 노드가 `unreachable` 상태가 되면, Node의 `.status` 필드의 `Ready` condition을 `Unknown`으로 설정
    - 기본적으로 node controller는 5분 기다림
2. 여전히 노드가 `unreachable` 상태라면, 해당 노드 내 모든 파드에 대해 API-initiated eviction을 수행

### Rate limits on eviction

대부분 node controllersms 초당 `--node-eviction-rate` 마다 evict를 수행함 (e.g. `--node-eviction-rate=0.1`, 10초마다 최대 1개의 노드를
evict)    
가용 영역에서 node가 Unhealthy가 되면 eviction을 시작함, 해당 영역에서 `Ready`가 `Unknown`으로 설정된 노드의 비율을 먼저 파악

- unhealthy 비율이 `--unhealthy-zone-threshold` 를 넘겼다면, evict 속도가 줄어듬
- 클러스터가 작아 노드 수가 `--large-cluster-size-threshold` 이하인 경우 evict 멈춤
- 아니면 `--secondary-node-eviction-rate` 를 따라 evict 속도를 줄임

노드를 가용 영역으로 구분하여 배치하는 이유는, 가용역역 전체가 다운되면, work load를 다른 가용 영역으로 이동시키기 위함  
따라서 한 영역의 모든 노드가 unhealthy면 `--node-eviction-rate` 에 따라 노드를 evict해서 다른 영역으로 이동시킴

`NoExecute` taint가 있는 노드로부터 pod를 Evict 수행, 문제가 있는 노드에 해당 문제에 대한 taint를 추가함  
스케쥴러는 해당 tainted 노드에 pod를 스케쥴링하지 않음

## Resource capacity tracking

node object는 node의 resource capacity (e.g. memory, cpu)를 추적함. Self register Node는 registeration 시 capacity report를 함  
수동으로 등록한 node는 node 추가시 capacity를 명시해야함

k8s scheduler는 노드의 모든 파드가 충분한 리소스를 보장받게 함. scheduler는 노드의 컨테이너가 요청량의 합이 node의 capacity를 넘지 않도록 함    
노드의 컨테이너가 요청량의 합에는 kublet이 관리하는 모든 컨테이너를 포함, container runtime에서 직접 시작한 컨테이너 제외, kublet 제어 밖에서 시작한 프로세스 제외

## Node topology

`TopologyManager` feature gate를 켜면, kubelt은 resource 할당시 topology 정보를 사용함

## Swap memory management

kublet의 `NodeSwap` feature gate를 켜고, `--fail-swap-on=false` cli flag 혹은 `failSwapOn` config를 false로 설정하면 node swap 가능  
pod가 swap을 사용하려면 `swapBehavior`를 `NoSwap`으로 설정하면 안됨

아래처럼 node의 swap memeory를 설정 가능

````yaml
memorySwap:
  swapBehavior: LimitedSwap
````

- `NoSwap` : default, k8s workload는 swap을 사용하지 않음
- `LimitedSwap` : k8s workload는 swap에 제한되지만 Burstable QoS의 파드는 스왑 메모리 사용 가능

용어 정리

- **nodeTotalMemory** : node가 사용가능한 전체 물리 메모리
- **totalPodsSwapAvailable** : pod에 의해 사용가능한 Noded의 전체 swap 메모리
- **containerMemoryRequest** : 컨테이너의 메모리 요청량
- swap 제한 량 = (containerMemoryRequest / nodeTotalMemory) * totalPodsSwapAvailable
