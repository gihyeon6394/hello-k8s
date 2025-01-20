# Communication between Nodes and the Control Plane

- Node to Control Plane
- Control Plane to Node

API server와 kubernetes cluster 간에 통신하는 방법  
cluster가 미신뢰 네트워크와 통신할 때 통신을 강화할수 있도록 함

---

## Node to Control Plane

default로 node (& pod)에서 control plane으로 의 연결은 secured connection을 사용함.

k8s는 hub-and-spoke API pattern을 사용함. 노드로부터의 모둔 API 요청은 API server에서 끝남. control plane의 다른 컴포넌트는 remoter service를 노출하지
않음  
API server만 remote connection에 대해 listen (443 port HTTPS protocol). 익명의 요청이나 service access token을 허용하려면 한개 이상의 인가를
활성화해두어야함

cluster는 Node에게 public root certificate를 제공해서 Node가 유효한 client credential을 가지고 API server에 접근할 수 있도록 함. (권장 : kubelet에
client certificate 제공)

API server에 연결하려는 pod는 service account를 사용하여 k8s가 자동으로 public root certificate를 주입하고, 유요한 bearer token을 제공. `kubernetes`
service는 `kube-proxy`를 통해 API server의 HTTPS endpoint로 리다이렉트하는 virtual IP가 설정되어있음

control plane 컴포넌트와 API server가 통신시에도 secure port를 사용함

## Control Plane to Node

#### control plane -> node 2가지 방법

- 방법 1. API server -> kublet process
- 방법 2. API server -> nodes, pods, services (API server의 proxy 기능 사용)

### API server to kubelet

다음 목적으로 사용함

- pod 로그 fetch
- `kubectl` 을 통해 실행중이 pod에 접근
- kublet에 port-forwarding 기능 제공

kubelet HTTPS endpoint에서 종료됨. 기본적으로 API server는 kublet이 서빙하는 certifiate을 신뢰하지 않음  
`--kubelet-certificate-authority` flag를 사용하여 API server가 kublet의 certificate를 신뢰하도록 설정할 수 있음  
위 설정이 불가능하다면, API server와 kublet 간에 SSH tunneling을 사용하는 방법이 있음

### API server to nodes, pods, and services

public, 미신뢰 네트워크에서 안전하지 않은 방법.

기본적으로 HTTP 연결을 사용하므로 인증되지 않으며, 암호화되어있지 않음. `https:` 접두사로 접근이 가능하지만, certificate 인증을 거치지 않음

### SSH tunnels (currently deprecated, konnectivity service 사용할 것)

API server는 각 노드마다 SSh tunnel을 열고, node, pod, service로 가는 모든 트래픽을 tunnel을 통해 전달. 트래픽이 노드가 실행중이 네트워크 외부로 노출되니 않도록 보장함

### Konnectivity service

SSH tunnel이 deprecated되면서 대체됨. TCP 레벨 프록시를 사용하여 통신. control plane에 konnectivity server를 두고 node에 konnectivity agent를
둠.  
konnectivity agent는 konnectivity server에 대한 커넥션을 초기화하고, 유지함. konnectivity service가 활성화 되면 node에 접근하는 모든 control plane의
트래픽은 해당 커넥션을 사용
